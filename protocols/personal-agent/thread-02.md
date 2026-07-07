# Protokoll: personal-agent — thread-02

<!-- Modus: Sparring (Implementer ⇄ Reviewer) -->
<!-- Ziel: nur S2 aus plans/personal-agent-review.md — /code aufs konfigurierte Repo beschränken -->
<!-- Dateien: bot.py -->
<!-- Start: 2026-07-08 -->

---

## [1] Human → Implementer · IMPL · open
<!-- runde: 0 -->

S2 umsetzen: `/code` nicht mehr in beliebige Owner-Repos, sondern nur ins konfigurierte Ziel-Repo.

---

## [2] Implementer → Reviewer · IMPL · needs-changes
<!-- runde: 1 -->

`/code` erstellt das Issue jetzt **direkt** im konfigurierten `GITHUB_REPO`. Entfernt:
der Repo-Picker (Inline-Buttons über alle Owner-Repos), die Funktion `_github_repos()`
(listete alle Repos), der Callback `on_code_repo_callback` + dessen Handler, sowie das
`pending_code`-Zwischenspeichern. Neue `GITHUB_REPO`-Leer-Prüfung ergänzt.
Deckt sich mit der README (`/code` → `ignatz89/meins`, ein Ziel-Repo).

---

## [3] Reviewer → Implementer · REVIEW · approved
<!-- runde: 1 -->

🟢 Verifiziert: `py_compile` OK; **keine** verwaisten Referenzen mehr auf `_github_repos`,
`on_code_repo_callback`, `pending_code`, `code_repo:` (grep leer); AST-Check: `cmd_code`
nutzt `GITHUB_REPO` und ruft `_create_github_issue(GITHUB_REPO, …)`, kein `reply_markup`/
Picker mehr. `_create_github_issue` bleibt genutzt. Kein 🔴/🟡.

Nebeneffekt (gewollt, YAGNI): der Multi-Repo-Picker fällt weg — weniger Code, klarere
Trust-Boundary.

---

## [4] Reviewer → Human · SUMMARY · approved
<!-- runde: 1 -->

Konvergiert nach 1 Runde. S2 umgesetzt und verifiziert: `/code` ist auf das konfigurierte
`GITHUB_REPO` beschränkt; die automatisierte Code-Ausführung kann nicht mehr in ein
beliebiges eigenes Repo gelenkt werden. Offen bleiben S3/S4/S7, R1, State-Handling.
