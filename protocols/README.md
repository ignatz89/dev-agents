# Protokolle

Hier liegt die **Kommunikations-Historie** der dev-agents — ein Ordner pro Projekt,
eine oder mehrere Thread-Dateien pro Durchlauf.

## Struktur

```
protocols/
  <projekt>/
    thread-01.md
    thread-02.md      # weiterer Durchlauf am selben Projekt
```

## Regeln

- **Nie löschen.** Diese Dateien zeigen, wie die Agenten miteinander kommuniziert
  haben — bewusst als Nachschlage-Historie gedacht.
- Neuer Durchlauf → **neue** Datei (`thread-NN` hochzählen), nicht überschreiben.
- Format & Ablauf: [../COMMUNICATION.md](../COMMUNICATION.md) und [../MODES.md](../MODES.md).
- Vorlage für einen neuen Thread: [../templates/thread.md](../templates/thread.md).

Ein ausgefülltes Beispiel: [_example-health-feelfit/thread-01.md](_example-health-feelfit/thread-01.md).
