# Protokoll: job-search — thread-01

<!-- Modus: Audit (Security + Reviewer, Fokus Effizienz/Runtime) -->
<!-- Ziel: schneller/guenstiger OHNE Trefferquote zu reduzieren -->
<!-- Dateien: job_agent.py, scrapers/base.py, scrapers/job_details.py, skills/claude_scorer.py -->
<!-- Start: 2026-07-06 -->

> Nur-Analyse-Lauf (keine Code-Aenderung). Findings mit stabilen IDs fuer einen
> spaeteren Sparring-Loop.

---

## [1] Human → Security+Reviewer · REVIEW · open
<!-- runde: 0 -->

Security + Reviewer ueber job-search laufen lassen: Effizienz/Runtime verbessern
ohne Trefferquote zu senken.

---

## [2] Reviewer → Human · REVIEW · needs-changes
<!-- runde: 1 -->

Effizienz/Runtime — priorisiert. "Sicher" = kein Einfluss auf die Trefferquote.

🟢 GUT: `filter_unseen` laeuft VOR enrich+score → bereits gesehene Jobs werden
   nicht neu geladen/bewertet. Prompt-Caching (System-Prompt ephemeral) aktiv.

🟡 R1 · scrapers/base.py + run_scrapers · [CONFIRMED] — **groesster Runtime-Kostenpunkt.**
   Jeder `_scrape_one`-Aufruf erzeugt via `make_driver()` einen NEUEN Chrome. Bei
   ~27 Queries × 4 Selenium-Plattformen ≈ 108 Browser-Starts (je ~2-4s + Navigation).
   Fix (sicher): einen Driver pro Plattform ueber alle Queries wiederverwenden
   (Driver einmal oeffnen, Queries innen loopen). ~4 statt ~108 Browser. Selbe
   Seiten → Trefferquote unveraendert.

🟡 R2 · scrapers/job_details.py `enrich_jobs` · [CONFIRMED] — sequentieller
   urllib-Fetch + `time.sleep(0.3-0.7)` pro Job. Bei 50 neuen Jobs ~25s reines
   Sleep + Latenz. Fix (sicher): mit ThreadPoolExecutor parallelisieren (6-8 Worker).
   Zusatz: Stepstone-Detailseiten liefern 403 → enrich gibt immer {} zurueck =
   verschwendeter Fetch+Sleep. Fuer source="stepstone" enrich ueberspringen (war
   ohnehin erfolglos → keine Trefferquoten-Auswirkung).

🟡 R3 · skills/claude_scorer.py `score_jobs` · [CONFIRMED] — Batches (je 30) laufen
   SEQUENTIELL. Fix (sicher): Batch 1 zuerst (waermt System-Prompt-Cache), dann
   Batch 2..N parallel (lesen den Cache; gleichzeitige Requests koennen den
   in-flight-Write des jeweils anderen nicht lesen — darum erst einen warmlaufen
   lassen). Trefferquote unveraendert.

🟢 R4 · skills/claude_scorer.py · [CONFIRMED] — Modell `claude-opus-4-7` ist eine
   Generation alt. Drop-in-Update auf `claude-opus-4-8` (gleiche API-Oberflaeche,
   gleicher Preis, schneller/besser). Sicher.

Validieren-vor-Uebernahme (potenzieller Trefferquoten-Einfluss, NICHT blind):
🟡 R5 — Scoring-Modell auf `claude-sonnet-5` ($3/$15, Intro $2/$10) statt Opus
   ($5/$25): ~40% guenstiger + schneller, laut Doku fuer High-Volume-Klassifikation
   empfohlen. ABER Scores koennen sich verschieben → gegen Opus-Baseline A/B testen,
   bevor uebernommen. Alternativ 2-stufig: Haiku-Vorfilter → Opus nur auf Grenzfaelle.
🟢 R6 — `thinking: adaptive` + `output_config.effort` senken (z.B. medium) spart
   Tokens/Latenz; ebenfalls gegen Trefferquote pruefen.

Fazit: R1-R4 sind sichere Gewinne (Runtime + Kosten runter, Trefferquote gleich).
R5/R6 nur mit A/B-Test.

---

## [3] Security → Human · SECURITY · needs-changes
<!-- runde: 1 -->

Vollstaendiges Review → `plans/security/2026-07-06_job-search.md`.
Kurz: 🔴 keine. 🟡 persoenliche Daten (profiles/*, data/*.json) getrackt — wenn
Repo public, exponiert. Gut: `config.ini` (Secrets) ist gitignored, API-Key aus Env.

---

## [4] Reviewer → Human · SUMMARY · approved
<!-- runde: 1 -->

Keine 🔴. Sichere Effizienz-Gewinne (R1-R4) koennen ohne Trefferquoten-Risiko
umgesetzt werden — erwarteter Effekt: Runtime von vielen Minuten (Browser-Churn)
deutlich runter, Scoring schneller, Modell aktueller. R5/R6 als optionale
Kosten-Hebel mit A/B-Test. Security-Handlungsbedarf: Repo-Sichtbarkeit + persoenliche
Daten aus Git.
