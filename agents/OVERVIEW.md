# Agenten im Überblick

| Agent | Modell | Datei |
|---|---|---|
| Architect | `claude-opus-4-8` | `architect.md` |
| Implementer | `claude-sonnet-4-6` | `implementer.md` |
| Reviewer | `claude-opus-4-8` | `reviewer.md` |
| Security | `claude-fable-5` | `security.md` |

---

## Architect
**Modell:** `claude-opus-4-8` — `/model claude-opus-4-8`

Plant Ideen, stellt Rückfragen, trifft technische Entscheidungen.
Schreibt **niemals** Code — nur Pläne.

Wann einsetzen: Am Anfang jeder neuen Idee. Immer zuerst.

Output: Strukturierter Plan → wird in `plans/` gespeichert.

---

## Implementer
**Modell:** `claude-sonnet-4-6` — `/model claude-sonnet-4-6`

Setzt den Plan des Architects exakt um. Kein Over-Engineering,
keine eigenen Design-Entscheidungen, keine ungefragten Extras.

Wann einsetzen: Wenn der Plan vom Architect freigegeben ist.

Output: Fertiger Code + Bericht was gemacht wurde und was abweicht.

---

## Reviewer
**Modell:** `claude-opus-4-8` — `/model claude-opus-4-8`

Prüft Code kritisch und skeptisch. Geht davon aus dass etwas falsch ist.
Schreibt keinen Code — zeigt nur was geändert werden sollte.

Wann einsetzen: Nach dem Implementer, vor dem Push zu GitHub.

Output: 🔴 Critical / 🟡 Warning / 🟢 Suggestion + fehlende Tests.

---

## Security
**Modell:** `claude-fable-5` — `/model claude-fable-5`

Analysiert auf Sicherheitslücken — denkt wie ein Angreifer.
Fokus auf: Hetzner Server, Raspberry Pi, GitHub-Repos, Telegram Bots,
GitHub Actions, Secrets.

Wann einsetzen: Bei allem was im Internet erreichbar ist, auf einem
Server läuft, in einem öffentlichen Repo landet oder Secrets berührt.

Output: Risiken nach Priorität + konkrete Gegenmaßnahmen.

---

## Typischer Workflow

```
Idee pitchen (templates/pitch.md)
    ↓
Architect  →  Plan erstellen  →  in plans/ speichern
    ↓
Implementer  →  Code schreiben
    ↓
Reviewer  →  Code prüfen  →  Fixes an Implementer zurück
    ↓
Security  →  (falls Internet/Server/Secrets betroffen)
    ↓
Push zu GitHub
```
