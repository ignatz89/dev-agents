# Protokoll: personal-agent — thread-03

<!-- Modus: Sparring (Implementer ⇄ Reviewer) -->
<!-- Ziel: alle Claude-Modelle eine Stufe hoch, jeweils aktuellste Version -->
<!-- Dateien: bot.py -->
<!-- Start: 2026-07-08 -->

---

## [1] Human → Implementer · IMPL · open
<!-- runde: 0 -->

Alle Modelle eins hoch: Sonnet statt Haiku, Opus statt Sonnet — jeweils neueste Version.

---

## [2] Implementer → Reviewer · IMPL · needs-changes
<!-- runde: 1 -->

- Chat (Text/Bild): `claude-sonnet-4-6` → **`claude-opus-4-8`** (2 Calls).
- Gedächtnis + `/code`-Titel: `claude-haiku-4-5-20251001` → **`claude-sonnet-5`** (2 Calls).
- Migrations-Detail: Sonnet 5 hat adaptives Thinking bei weggelassenem Parameter default AN.
  Bei den zwei Utility-Calls mit `max_tokens` 1200/60 würde Thinking das Budget auffressen →
  explizit `thinking={"type":"disabled"}` gesetzt (schnelles Haiku-artiges Verhalten erhalten).
- Opus 4.8 läuft bei weggelassenem Thinking ohne Thinking → Chat-Calls sauberer String-Tausch.
- Keine `temperature`/Prefills/`budget_tokens` im Code → keine weiteren Breaking Changes.

---

## [3] Reviewer → Implementer · REVIEW · needs-changes
<!-- runde: 1 -->

🟢 Statisch verifiziert: `py_compile` OK; Grep — Chat = `claude-opus-4-8` (2×, ohne Thinking),
Utility = `claude-sonnet-5` (2×, je mit `thinking:disabled`); keine `claude-sonnet-4-6`/`claude-haiku`
mehr; Groq-STT unberührt.
🟡 Offen: **Laufzeit-Verifikation** (API akzeptiert die Modelle + `thinking:disabled` mit dem
Account/SDK) kann lokal nicht laufen (kein Key/SDK) → als Smoke-Test beim Deploy auf dem Server.

---

## [4] Implementer → Reviewer · RESPONSE · needs-changes
<!-- runde: 2 -->

Smoke-Test beim Deploy vorgesehen: je ein Mini-Call gegen `claude-opus-4-8` und
`claude-sonnet-5` (thinking off) über die venv+config des Servers, bevor der Bot neu startet.

---

## [5] Reviewer → Human · SUMMARY · approved
<!-- runde: 2 -->

Statisch konvergiert. Modelle getauscht + Thinking-Migration korrekt. Laufzeit-Beleg folgt
beim Ausrollen (Smoke-Test beider Modelle). Kosten steigen bewusst (Opus/Sonnet teurer als
Sonnet/Haiku) — vom User so gewollt. Optional/offen: A4/R4 (Modell-IDs zentralisieren) und
Web-Tool `web_search_20250305` → `web_search_20260209` — nicht Teil dieses Auftrags.
