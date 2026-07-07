---
datum: 2026-07-06
system: job-search
status: offen
kritische_punkte: 0
---

# Security Review: job-search

## Assets
- Anthropic API-Key · Telegram Bot-Token + chat_id · persönliches Bewerber-Profil
  (Gehalt, Dealbreaker, Werdegang) · Jobsuche-Historie

## Attack Surface
- Öffentliches GitHub-Repo (ignatz89/job-search) · Telegram-Bot · Selenium-Scraper

## Risiken

### 🔴 Kritisch
(keine)

### 🟡 Mittel (bald beheben)
- **Persönliche Daten im (öffentlichen) Repo** — getrackt sind `profiles/marc.ini`
  (Wunschgehalt, Dealbreaker, ausgeschlossene Rollen), `profiles/marc.txt`
  (Werdegang) und `data/seen_jobs_*.json` / `data/sent_jobs_*.json` (komplette
  Jobsuche-Historie). `.gitignore` schließt `*.json` aus, holt aber `data/*.json`
  bewusst wieder rein (`!data/*.json`).
  *Angriffsszenario:* Wenn das Repo public ist, ist das persönliche Bewerberprofil
  + Suchverhalten für jeden einsehbar (auch in der History).
  *Gegenmaßnahme (Schichten):* 1) Repo auf **private** stellen (schnellster Schutz);
  2) `data/*.json` und `profiles/*` aus Git nehmen (gitignoren) — nur lokal halten;
  3) falls schon public gewesen: History bereinigen (git filter-repo) + als
  kompromittiert behandeln.
  *Prüfbefehl:* `git ls-files | grep -E 'profiles/|data/.*json'` → sollte leer sein
  (nach Umsetzung). Repo-Sichtbarkeit im GitHub-Dashboard prüfen.

### 🟢 Gering
- **Telegram-Bot Eingaben** (`telegram_bot.py`, `getUpdates`) — sicherstellen, dass
  nur die eigene `chat_id` Befehle auslösen kann (Absender-ID validieren, keine
  offene Bot-Nutzung). *Prüfen:* Wird die Sender-`chat_id` gegen den Config-Wert geprüft?
- **Selenium** startet Chrome mit `--no-sandbox` — nur lokal relevant; auf einem
  Server nicht als root laufen lassen.

## Gut gemacht
- `config.ini` (API-Key, Bot-Token) ist **gitignored** → Secrets nicht im Repo. ✅
- `ANTHROPIC_API_KEY` wird aus der Umgebung gelesen, nicht hardcoded. ✅
