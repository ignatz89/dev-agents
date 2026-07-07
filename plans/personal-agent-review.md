# Review: personal-agent (Architect + Security + Reviewer)

**Modus:** reiner Analyse-Pass — **keine** Umsetzung. Alle Vorschläge gesammelt.
**Projekt:** `d:\repos\personal-agent` — Telegram-Bot (Text/Voice/Bild/PDF → Claude),
kumulatives Gedächtnis, `/code` → GitHub-Issue → claude-code-action → PR. Läuft als
systemd-Service auf dem Hetzner-Server.
**Kern:** `bot.py` (~640 Zeilen, Monolith), `config.ini`, `system_prompt.txt`,
`agent-storage.json`, GitHub-Workflow.

Priorität: **[S1]** (offene Auth) und **[S3]** (private Chats im Repo) zuerst.

---

## 🏗️ Architect — Struktur & Design

**A1 · Monolith `bot.py`.** Alle Belange in einer Datei (Config, State, Gedächtnis,
Claude-Calls, Telegram-Handler, GitHub, Audio, PDF). Für einen wachsenden Bot unübersichtlich.
*2-3 Ansätze:*
1. **So lassen** — für einen Ein-Personen-Bot vertretbar, klar mit Abschnitten gegliedert.
2. **In Module splitten** — `state.py`, `memory.py`, `claude_client.py`, `github.py`, `audio.py`, `handlers.py`.
3. **An `skills/`-Muster** der Schwester-Projekte (job-search/therapy-agent) angleichen → Konsistenz.
*Empfehlung:* (2)/(3) sobald weitere Features dazukommen; sonst (1) ok. Kein Selbstzweck.

**A2 · Topic-Feature ist ein No-Op.** Alle 5 Topics mappen auf denselben `SHARED_CHARACTER`
(Zeilen 112-118); dazu ein großer **auskommentierter** Block mit den echten Topic-Prompts.
Die `/topic`-Buttons ändern also nur ein Label, nicht das Verhalten.
*Entscheidung nötig:* Feature **richtig implementieren** (differenzierte Prompts reaktivieren)
**oder entfernen** (YAGNI) — der aktuelle Halbzustand ist irreführend.

**A3 · Skalierung des State.** `transcript` **und** `history` wachsen unbegrenzt in *einer*
`state_{chat_id}.json`, die bei **jeder** Nachricht komplett neu geschrieben wird → O(n)-Write
pro Turn, Datei wächst ewig. Für einen dauerhaft laufenden persönlichen Bot relevant.
*Ansätze:* (a) `transcript` als **append-only JSONL**; (b) `history` **kappen** (an Claude geht
ohnehin nur `[-CONTEXT_MESSAGES:]`); (c) SQLite. *Empfehlung:* (b)+(a) — simpel, großer Effekt.

**A4 · Modell-IDs inline verstreut** (`claude-sonnet-4-6`, `claude-haiku-4-5-...` mehrfach).
Zentral als Konstanten — erleichtert Wechsel/Migration.

---

## 🛡️ Security — Threat Model

**Assets:** private Gespräche + kumulatives Gedächtnis, API-Keys (Anthropic/Groq/AudD),
**GitHub-Token mit `repo`-Scope** (Code-Ausführung via `/code` in allen eigenen Repos).
**Attack Surface:** offener/kompromittierter Bot, prompt-injizierte `/code`-Ideen, ggf. öffentliches Repo.

**🔴 S1 · Fail-open-Authentifizierung.** `allowed_chat_id` hat Default `"0"`, und
`is_allowed()` gibt bei `ALLOWED_CHAT == 0` **für jeden** `True` zurück (bot.py:270-272).
Ist die ID nicht gesetzt, kann **jeder Fremde** den Bot voll nutzen — inkl. `/code`
(erzeugt Issues → automatisierte Code-Änderungen), Web-Suche und API-Kosten auf deine Rechnung.
*Fix:* **fail-closed** — bei `0` verweigern (und/oder `allowed_chat_id` als Pflichtfeld beim Start prüfen).
*Prüfbefehl:* von einem fremden Telegram-Account schreiben → **keine** Antwort.

**🟡 S2 · `/code` erlaubt jedes Owner-Repo.** `_github_repos()` listet **alle** eigenen Repos,
der Nutzer wählt beliebig → Issue+Automatisierung in *irgendeinem* Repo. Das konfigurierte
`GITHUB_REPO` wird ignoriert. Trust-Boundary zu breit (v. a. zusammen mit S1).
*Fix (Defense in Depth):* auf das konfigurierte Ziel-Repo beschränken statt „alle".

**🟡 S3 · Private Gesprächsinhalte im Repo.** `agent-storage.json` ist **getrackt** und enthält
echte Chat-Historie (`agent-marc-history` mit persönlichen Inhalten). Wenn das Repo public ist →
schwerwiegende Offenlegung; auch bei private unnötig im Versionsverlauf.
*Fix:* aus Git nehmen (`git rm --cached agent-storage.json`) + gitignoren; Repo-Sichtbarkeit prüfen;
falls je public → als offengelegt behandeln, History bereinigen.
*Gut:* `config.ini` und `data/` sind bereits gitignored ✅ (Keys/aktueller State nicht im Repo).

**🟡 S4 · GitHub-Token `repo`-Scope (alle Repos).** Least Privilege: ein **fine-grained PAT**,
beschränkt auf das eine Ziel-Repo, reduziert den Blast-Radius erheblich (greift S1/S2 ab).

**🟢 S5 · Rohe Exceptions an den Chat** (`⚠️ Fehler: {e}`, GitHub-Fehlerbody). Für Owner-only ok;
bei offenem Bot (S1) ein Info-Leak. Generische Nutzer-Meldung, Details ins Log.

**🟢 S6 · `urlopen` ohne Timeout** in `_github_repos()` / `_create_github_issue()` → kann hängen.
Timeout ergänzen. (`drop_pending_updates=True` beim Start ist gut.)

---

## 🔍 Reviewer — Korrektheit / YAGNI / Effizienz

**🟡 R1 · Möglicher Crash in `on_document`** (bot.py:424): `doc.file_name.endswith(...)` —
`file_name` kann bei Telegram `None` sein → `AttributeError`. Guard: `(doc.file_name or "")`.

**🟡 R2 · Effizienz/Doppelspeicherung.** `history` und `transcript` speichern **dieselben** Daten
(DRY-Verstoß); `history` wird nie gekappt, nur beim Senden an Claude gesliced. Zusammen mit dem
Voll-Neuschreiben der Datei (A3) wächst der Aufwand pro Nachricht linear.
*Fix:* `history` aus dem `transcript`-Tail ableiten **oder** `history` hart kappen.

**🟢 R3 · Toter Code / No-op-Feature.** Auskommentierter `TOPIC_PROMPTS`-Block + wirkungslose
Topics (= A2). Entfernen oder reaktivieren.

**🟢 R4 · DRY:** Modell-ID-Strings mehrfach dupliziert (= A4).

**🟢 R5 · `_extract_pdf` bei fehlendem `pypdf`** gibt einen Platzhalter-String zurück, der dann
als „Dateiinhalt" an Claude geschickt wird. Besser: klarer Nutzer-Hinweis; `pypdf` in
`requirements.txt` sicherstellen (prüfen).

**🟢 R6 · Optional (Modernisierung):** Web-Search-Tool `web_search_20250305` →
`web_search_20260209` (dynamisches Filtern) — wird von `claude-sonnet-4-6` unterstützt.

---

## Priorisierte Gesamtempfehlung

| # | Punkt | Warum zuerst |
|---|---|---|
| 1 | **S1** fail-closed Auth | Offener Bot = fremder Zugriff auf `/code`, Kosten, Daten |
| 2 | **S3** agent-storage.json aus Git + Sichtbarkeit prüfen | Private Chats potenziell exponiert |
| 3 | **S2** `/code` auf Ziel-Repo beschränken · **S4** fine-grained PAT | Blast-Radius der Automatisierung |
| 4 | **R1** None-Crash · **R2/A3** State kappen/entkoppeln | Korrektheit + Skalierung |
| 5 | **A2/R3** Topic-Feature entscheiden · **A4/R4** Modell-Konstanten | Aufräumen (YAGNI/DRY) |

*Nächster Schritt (auf Wunsch):* Implementer setzt ausgewählte Punkte im Sparring-Loop um —
die 🔴/🟡-Security-Punkte zuerst.
