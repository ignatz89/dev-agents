# Kommunikation zwischen den Agenten

Die Agenten reden **nicht frei**, sondern tauschen sich über getippte Nachrichten in
einer gemeinsamen Protokoll-Datei aus. Der **Code im Git** ist das eigentliche
Artefakt; das Protokoll trägt Entscheidungen, Findings und Antworten.

---

## Geteilter Arbeitsraum

- Pro Projekt ein Ordner: `protocols/<projekt>/`
- Pro Durchlauf eine (oder mehrere) Datei: `protocols/<projekt>/thread-NN.md`
- **Dateien werden nie gelöscht** — sie sind die Historie, wie die Agenten
  kommuniziert haben. Neuer Durchlauf → neue Datei (`NN` hochzählen).

Vorlage: [templates/thread.md](templates/thread.md)

---

## Nachrichten-Umschlag

Jede Übergabe ist ein Block mit festem Kopf:

```
## [<n>] <FROM> → <TO> · <TYPE> · <STATUS>
<!-- runde: <r> · zeit: <YYYY-MM-DD HH:MM> -->

<Inhalt>
```

- **TYPE:** `PLAN` · `IMPL` · `REVIEW` · `RESPONSE` · `SECURITY` · `SUMMARY`
- **STATUS:** `open` · `needs-changes` · `approved` · `blocked` · `done`
- `<n>` läuft fortlaufend über den ganzen Thread.

---

## Findings & Antworten (Sparring-Loop)

**Reviewer** (`TYPE: REVIEW`) listet Findings mit stabiler ID + Konfidenz:

```
🔴 R1 · loader.py:161 · [CONFIRMED] — groupby.mean() ohne numeric_only → Crash bei
        String-Spalte. Fix: numeric_only=True
🟡 R2 · feelfit.py:70 · [PLAUSIBEL] — kein Test für leeres Sheet
```

**Implementer** (`TYPE: RESPONSE`) antwortet **pro ID**:

```
R1 ✓ fixed — numeric_only=True gesetzt (loader.py:161)
R2 ✗ won't-fix — leeres Sheet ist in load_feelfit() bereits abgefangen
```

Symbole: `✓ fixed` · `✗ won't-fix (+ Grund)` · `↷ deferred`.
Vorlage: [templates/review_response.md](templates/review_response.md)

**Reviewer** schließt eine ID erst, wenn er den Fix im **echten Code verifiziert** hat
(`[CONFIRMED]`), nicht auf Zuruf.

---

## Abschluss

Der letzte Block ist immer eine **SUMMARY** (vom Reviewer bzw. Conductor):
- Endzustand — konvergiert? Wie viele Runden?
- offene `won't-fix` / Dissens → Entscheidung durch dich
- bei Security: offene 🔴, die den Push blockieren

Die Terminierungs-Regeln stehen in [MODES.md](MODES.md).

---

## Orchestrierung — wer dirigiert?

- **Jetzt (manuell):** Du wählst den Modus, lädst die Persona (`Lies agents/<rolle>.md`),
  und das `thread`-File ist das geteilte Gedächtnis. Innerhalb eines Modus läuft der
  Loop autonom durch — du greifst inhaltlich nicht ein, sondern liest am Ende die SUMMARY.
- **Später (automatisiert):** ein schlanker **Conductor** oder Superpowers
  `subagent-driven-development` — frische Subagenten pro Rolle, der Conductor reicht
  Findings weiter und prüft die Terminierung. Das Nachrichtenformat bleibt identisch.
