# Protokoll: personal-agent — thread-04

<!-- Modus: Sparring (Implementer ⇄ Reviewer) -->
<!-- Ziel: Stufe 1 Voice-Antworten (Piper, selbst gehostet) + /voice /text Toggle -->
<!-- Dateien: tts.py (neu), bot.py, README.md -->
<!-- Start: 2026-07-10 -->

> Grundlage: plans/personal-agent-voice.md. Anbieter-Entscheid: Piper (Groq kann kein
> Deutsch — vor dem Bau verifiziert).

---

## [1] Human → Implementer · IMPL · open
<!-- runde: 0 -->

Stufe 1 mit Piper. Toggle: /voice → Sprache bis /text.

---

## [2] Implementer → Reviewer · IMPL · needs-changes
<!-- runde: 1 -->

- Neues Modul **`tts.py`** (nicht `skills/` — personal-agent ist Ein-Datei-Projekt):
  `available()` + `synthesize(text)->ogg/opus|None` via Piper→WAV→ffmpeg→OGG/Opus.
  Anbieter/Modell aus `config.ini [tts]`, austauschbar. Fehler → None (Aufrufer fällt auf Text).
- `bot.py`: `_deliver(update, state, text)` — sendet Voice, wenn `voice_mode` aktiv UND TTS
  verfügbar, sonst Text (Synthese in `asyncio.to_thread`, blockt den Event-Loop nicht).
  Kommandos `/voice` (an) und `/text` (aus), persistenter `voice_mode` im State.
  Alle 5 Antwort-Pfade (Text/Bild/Doc/Voice-Song/Voice-Transkript) über `_deliver`.
- README: `[tts]`-Config + Server-Setup (ffmpeg, piper-tts, deutsche Thorsten-Stimme).

Für den Reviewer: Laufzeit (Piper synthetisiert, ffmpeg-Opus, Telegram akzeptiert Voice)
ist lokal nicht testbar → Smoke-Test auf dem Server vor dem Restart.

---

## [3] Reviewer → Implementer · REVIEW · approved
<!-- runde: 1 -->

🟢 Statisch: `py_compile` bot.py + tts.py OK; Grep — alle 5 Sends nutzen `_deliver`, kein
`reply_text(reply)` mehr; `reply_voice` nur in `_deliver`. Fail-safe: ohne Modell →
`available()` False → Text (Bot bleibt funktionsfähig). Kein 🔴/🟡.
🟡 Offen: Laufzeit-Smoke auf dem Server (Piper-Invocation/Flags, ffmpeg libopus, Telegram-Voice).

---

## [4] Reviewer → Human · SUMMARY · approved
<!-- runde: 1 -->

Statisch konvergiert. Stufe 1 gebaut, sicher (Text-Fallback). Laufzeit-Beleg + Server-Setup
(ffmpeg + piper-tts + Thorsten-Stimme + `[tts]`-Config) beim Ausrollen. Danach: `/voice` testen.
