# Modi — welches Agenten-Setup für welche Aufgabe

Du wählst den Modus **einmal am Anfang** und nennst Ziel + betroffene Dateien.
Danach läuft der gewählte Ablauf **autonom** — du wirst **nicht** zwischen den
Iterationen gefragt. Erst am Ende bekommst du eine **SUMMARY** mit dem Ergebnis und
allen Punkten, die wirklich eine Entscheidung von dir brauchen.

> Nachrichtenformat & Kommunikation: [COMMUNICATION.md](COMMUNICATION.md)
> Rollen der Agenten: [agents/OVERVIEW.md](agents/OVERVIEW.md)

---

## Modus 1 — Sparring   (Implementer ⇄ Reviewer)

**Kette:** Implementer → Reviewer → ⟲ Loop → SUMMARY
**Wofür:** klar umrissene Änderung/Feature, kein neuer Entwurf nötig.

1. Implementer setzt das Ziel um.
2. Reviewer prüft, vergibt Findings (R1, R2 …) mit Severity + Konfidenz.
3. Implementer antwortet **pro ID** (`✓ fixed` / `✗ won't-fix` + Grund) und passt Code an.
4. Reviewer re-reviewt nur offene IDs + Deltas — **liest den echten Code**.
5. Wiederholung **ohne Rückfrage an dich**.

**Terminierung (autonom):**
- Konvergiert, sobald **kein 🔴 und kein vom Reviewer bestandenes 🟡** offen ist.
- 🔴 hat der Reviewer die Hoheit — muss behoben werden.
- **Harte Grenze: 4 Runden.** Danach Stopp + SUMMARY mit offenen Punkten.
- `won't-fix`-Dissens blockiert **nicht** mitten im Loop — wird gesammelt und in der
  SUMMARY zur Entscheidung an dich gelegt.

---

## Modus 2 — Full Build   (Idee → fertig)

**Kette:** Architect → Implementer → Reviewer ⟲ → Security → SUMMARY
**Wofür:** neue, riskante oder internetseitige Idee von Grund auf.

1. Architect klärt (Rückfragen nur **einmal am Anfang**, falls Ziel unklar) und
   schreibt den Plan.
2. Implementer setzt den Plan um.
3. Sparring-Loop wie Modus 1 (bis konvergiert / Rundenlimit).
4. Security — **nur** wenn Server / Internet / Secrets / öffentliches Repo betroffen —
   mit Verifikations-Phase (Prüfbefehl + Status je Maßnahme).
5. SUMMARY.

**Terminierung:** Loop wie Modus 1. Security-Befunde mit offenem 🔴 gehen in die
SUMMARY (blockieren den Push, unterbrechen aber nicht den Loop).

---

## Modus 3 — Refactor-Pass   (bestehendes Projekt verbessern)

**Kette:** Architect (Gesamtblick) → Implementer → Reviewer ⟲ → SUMMARY
**Wofür:** Projekt ist „fertig", soll aber ganzheitlich überarbeitet werden.

1. Architect betrachtet das **ganze** Projekt, benennt Schwachstellen/Schulden und
   schreibt einen Verbesserungs-Plan (kein Code).
2. Implementer setzt den Plan um.
3. Sparring-Loop mit dem Reviewer bis konvergiert / Rundenlimit.
4. SUMMARY.

---

## Autonomie-Regel (gilt für alle Modi)

- **Am Anfang** entscheidest du: Modus, Ziel, betroffene Dateien.
- **Während der Iteration**: keine Rückfragen an dich. Die Agenten entscheiden nach
  den Terminierungs-Regeln oben.
- **Am Ende**: genau eine SUMMARY. Nur *harte Blocker* landen dort zur Entscheidung —
  widersprüchliche Anforderung, fehlender Zugang, oder ein offener 🔴 nach Rundenlimit.
- Jeder Schritt wird ins Protokoll geschrieben und **nie gelöscht** → du kannst später
  nachvollziehen, wie die Agenten kommuniziert haben ([protocols/](protocols/)).

---

## Modus starten (Beispiele)

> „**Modus: Sparring.** Ziel: FeelFit-Loader robust gegen leere Sheets machen.
> Dateien: `analyzers/feelfit.py`. Protokoll: `protocols/health-feelfit/`."

> „**Modus: Full Build.** Idee: NAS lokal-only für Android-Backup. Security aktiv."

> „**Modus: Refactor-Pass.** Projekt: `meins/health` ganzheitlich aufräumen."
