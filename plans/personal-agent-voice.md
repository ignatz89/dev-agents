# Plan: personal-agent — Sprach-Ausgabe (Voice-Antworten) & Telefonie

**Agent:** Architect (nur Plan, kein Code).
**Ziel:** (1) Der Bot antwortet mit **Sprachnachrichten** (nicht nur Text). (2) Ausbaustufe:
**echtes Telefonieren** (Live-Gespräch).

## Ausgangslage (aktueller Stand in bot.py)
- **Sprache REIN funktioniert bereits**: Voice → Groq-Whisper (STT) → Claude → **Text**.
- Es fehlt nur die **Rück-Richtung**: Text → Sprache (TTS) → als Telegram-Voice zurück.
- Zwei harte Rahmenbedingungen:
  - **Anthropic** liefert keine Audio-Ausgabe → TTS braucht einen externen Anbieter.
  - **Telegram-Bot-API** kann **keine** Live-Anrufe → Telefonie = eigene Telefonie-/Voice-Schicht.

---

## Ansätze & Trade-offs

### Stufe 1 — Voice-Antworten in Telegram (klein, hoher Nutzen)
Nach `get_reply()` den Text durch **TTS** schicken und per `reply_voice()` als OGG/Opus-
Sprachnachricht senden. Bleibt komplett im bestehenden Bot → **Gedächtnis & Charakter bleiben**.

**TTS-Anbieter (Deutsch-Qualität zählt — es ist ein persönlicher Begleiter):**

| Anbieter | Deutsch | Latenz | Kosten | Neuer Key? |
|---|---|---|---|---|
| **ElevenLabs** | sehr natürlich/emotional | niedrig (Streaming) | ab ~5 $/Mon + pro Zeichen | ja |
| **Groq PlayAI TTS** | ok (Deutsch verifizieren) | sehr niedrig | günstig | **nein** — Groq-Key ist schon da |
| **OpenAI TTS** (`gpt-4o-mini-tts`) | gut | niedrig | sehr günstig | ja |
| Azure/Google Neural TTS | sehr gut | niedrig | günstig-mittel | ja + mehr Setup |

**Weitere Design-Entscheidungen Stufe 1:**
- **Wann Voice?** Empfehlung: **spiegeln** — schickst du Sprache, antwortet er per Sprache;
  tippst du, antwortet er per Text. Plus `/voice`-Toggle für „immer Sprache". (Passt zu „mir Voice-Mails schicken".)
- **Format:** Telegram-Voice = OGG/Opus. Die meisten TTS liefern MP3/WAV → **ffmpeg** auf dem
  Server konvertiert (MP3 → OGG/Opus). (Alternativ `reply_audio` mit MP3 = weniger Setup, aber
  keine echte „Sprachnachricht-Blase".)
- **Länge/Kosten:** lange Antworten = teure/lange Audios → für Voice-Modus Antwortlänge kappen
  oder ab X Zeichen zusätzlich Text schicken.
- **Robustheit:** TTS-Fehler → Fallback auf Textantwort.

**Aufwand:** klein — 1 neues Skill-Modul + wenige Zeilen in `on_voice`/`on_text` + ffmpeg.

### Stufe 2 — Echtzeit-Telefonie (eigenes, größeres System)
Da Telegram-Bots nicht telefonieren, braucht es eine **Telefonie-/Voice-Agent-Schicht**. Drei Wege:

- **2a · Twilio-Nummer + eigene Realtime-Pipeline (Claude als „Brain").**
  Anruf → Twilio Media Streams (WebSocket) → Server: STT (Deepgram/Groq realtime) → **Claude
  (mit Gedächtnis)** → TTS (ElevenLabs Streaming) → zurück.
  *+* volle Claude-Persönlichkeit & Gedächtnis. *−* hoher Eigenbau (VAD, Barge-in, Latenz-Tuning),
  Server muss öffentlich erreichbar sein (Cloudflare Tunnel), ~1-2 s Latenz/Turn.

- **2b · Voice-Agent-Plattform (Vapi / Pipecat / LiveKit) + Twilio.** *(empfohlen für Stufe 2)*
  Die Plattform orchestriert STT+LLM+TTS realtime und bindet die Telefonnummer an; **Claude** als
  LLM, ElevenLabs-Stimme, Deepgram-STT einsteckbar. Gedächtnis via Custom-LLM-Endpoint (unser
  Bot-Server liefert Claude+Memory).
  *+* viel weniger Eigenbau, schnell zum funktionierenden Anruf, Claude-Brain bleibt. *−* Plattform-Kosten.

- **2c · Natives Realtime-Speech-Modell (OpenAI Realtime / Gemini Live).**
  *+* niedrigste Latenz, natürlichstes Gespräch. *−* der „Brain" ist dann **nicht Claude** —
  Persönlichkeit/Gedächtnis des Agents gingen verloren bzw. müssten mühsam reininjiziert werden.

---

## Technologie-Empfehlung
1. **Stufe 1 zuerst** — kleiner, in sich geschlossener Schritt, behält Gedächtnis/Charakter,
   liefert sofort „Voice-Mails vom Agent". Anbieter: **ElevenLabs** für die beste Companion-Stimme,
   **Groq PlayAI** als pragmatische Reuse-Key-Variante (Deutsch vorher kurz gegenhören).
2. **Stufe 2 später** als eigenes Projekt, wenn Stufe 1 sitzt — voraussichtlich **2b
   (Vapi/Pipecat + Twilio + Claude + ElevenLabs)**, weil es den Claude-Brain behält und den
   Realtime-Eigenbau minimiert. 2c nur, wenn dir minimale Latenz wichtiger ist als die Claude-Identität.

## Für den Implementer (Stufe 1, konkret — später)
- Neues Modul `skills/tts.py`: `synthesize(text) -> ogg_opus_bytes` (Anbieter-Call + ffmpeg-Konvertierung), mit Fehler→None.
- `config.ini` `[tts]`: `provider`, `api_key` (bzw. Groq-Key wiederverwenden), `voice_id`, `enabled`.
- In `on_voice`: nach `reply = get_reply(...)` → wenn Voice-Modus aktiv: `audio = tts.synthesize(reply)`; `await update.message.reply_voice(audio)` (Fallback `reply_text` bei None). `/voice`-Command als Toggle im State.
- Server: `apt-get install -y ffmpeg`. Ggf. Anbieter-SDK in `requirements.txt` (oder schlicht `urllib`).

## Akzeptanzkriterien (Stufe 1)
- Sprachnachricht rein → Bot antwortet als **Telegram-Voice** (OGG/Opus), deutsche Stimme.
- Kurze Antworten < ~4 s Ende-zu-Ende; TTS-Fehler → automatischer Text-Fallback.
- `/voice` schaltet „immer Sprache" an/aus; getippte Nachrichten bleiben standardmäßig Text.
- Gedächtnis/Transcript unverändert wie bei Textantworten.

## Nicht im Scope
- Implementierung von Stufe 2; Stimmen-Klonen; Mehrbenutzer; Telefon-Nummer-Beschaffung.

## Offene Fragen (bitte entscheiden)
1. **TTS-Anbieter/Budget** für Stufe 1: ElevenLabs (beste Qualität, kostenpflichtig) / Groq
   (Key vorhanden, Deutsch prüfen) / OpenAI (günstig, neuer Key)?
2. **Voice-Logik**: „spiegeln + `/voice`-Toggle" ok, oder lieber **immer** Sprache?
3. **Telefonie (Stufe 2)**: echte Telefonnummer gewünscht (Twilio, Land/Kosten)? Ist dir
   **Claude-als-Brain** wichtiger als minimale Latenz? → bestimmt 2b vs. 2c.
