# dev-agents

Spezialisierte Entwicklungs-Agenten für Claude Code.

Jeder Agent hat einen klar definierten Charakter und Verantwortungsbereich.
Die System-Prompts werden über die Zeit verbessert.

---

## Workflow

```
1. Idee pitchen (templates/pitch.md ausfüllen)
        ↓
2. Agents auswählen
        ↓
3. Architect → Plan erstellen (plans/ speichern)
        ↓
4. Implementer → Code schreiben
        ↓
5. Reviewer + Security → Prüfen
        ↓
6. Push zu GitHub
```

---

## Agenten

| Agent | Datei | Aufgabe |
|---|---|---|
| **Architect** | `agents/architect.md` | Plant, stellt Rückfragen, trifft Tech-Entscheidungen. Schreibt NIE Code. |
| **Implementer** | `agents/implementer.md` | Setzt Plan exakt um. Kein Over-Engineering. |
| **Reviewer** | `agents/reviewer.md` | Kritische Code-Prüfung. Output: 🔴/🟡/🟢 |
| **Security** | `agents/security.md` | Threat Modeling, Sicherheitsanalyse. |

---

## Wie man einen Agent aktiviert

In Claude Code einfach sagen:

> "Lies `agents/architect.md` als deinen Charakter für diese Aufgabe."

Oder bei einem Pitch:

> "Idee: [Beschreibung]. Agents: Architect + Implementer. Lies die entsprechenden Prompts."

---

## Verzeichnisse

```
agents/      — System-Prompts der Agenten
templates/   — Vorlagen für Pitches und Übergaben
plans/       — Gespeicherte Projektpläne
```

---

## Übergabe zwischen Agenten

Strukturierte Übergaben verhindern Informationsverlust:

- **Architect → Implementer**: `templates/handoff_arch_to_impl.md`
- **Implementer → Reviewer**: `templates/handoff_impl_to_reviewer.md`
