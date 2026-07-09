# Protokoll: spiele — thread-01

<!-- Modus: Sparring (Implementer ⇄ Reviewer) -->
<!-- Ziel: neues Repo, Mehrspieler-Web-Spiele (Malen&Raten + Mensch ärgere dich nicht), auf Hetzner gehostet -->
<!-- Dateien: d:\repos\spiele\* -->
<!-- Start: 2026-07-10 -->

---

## [1] Human → Implementer · IMPL · open
<!-- runde: 0 -->

Neues Repo, Ordner pro Spiel. Malen & Raten + Mensch ärgere dich nicht. Hosting: Hetzner.

---

## [2] Implementer → Reviewer · IMPL · needs-changes
<!-- runde: 1 -->

- **server.py** (aiohttp): generischer Relay — Räume, Broadcast an andere, gezielt via `to`.
  Spiel-neutral; Logik im Host-Client. Serviert Lobby + `games/<spiel>/index.html`.
- **games/maler**: Runden, Zufallswort, Live-Striche (normierte Koords), Chat/Raten, Punkte, Timer, Zeichner-Rotation. Host-autoritativ; Wort nur an den Zeichner (`to`).
- **games/aergern**: Mensch ärgere dich nicht, host-autoritativ (6 zum Rauskommen, schlagen, Haus, Überwurf-Schutz, Extra-Wurf bei 6, Sieg = alle 4 im Ziel), runder Canvas-Plan.
- Lobby (static/): Name + Raum-Code → Spiel.

---

## [3] Reviewer → Implementer · REVIEW · needs-changes
<!-- runde: 1 -->

🔴 R1 · aergern render() · [CONFIRMED] — kaputte Ring-Strichfarbe (`"#3a4considered".slice()`)
   → ungültige Farbe. Fix: `"#2a3040"`.
🟡 R2 · maler · [CONFIRMED] — Reste aus 1. Wurf: No-op-`send`, toter `_oldHandle`, doppeltes
   `time`-Handling via extra addEventListener. Fix: `time` in handle()-Switch, Reste raus.
🟢 R3 · Laufzeit-Gameplay lokal nicht testbar (kein Browser) → Playtest durch die Nutzer.

---

## [4] Implementer → Reviewer · RESPONSE · needs-changes
<!-- runde: 2 -->

R1 ✓ fixed — Ring-Strichfarbe korrigiert.
R2 ✓ fixed — maler aufgeräumt (time im Switch, Reste entfernt).
R3 ✓ akzeptiert — statt Gameplay-Test: Server-Verifikation + Deploy-Nachweis.

---

## [5] Reviewer → Human · SUMMARY · approved
<!-- runde: 2 -->

Statisch konvergiert. Verifiziert: `py_compile server.py` OK; Deploy auf Hetzner (systemd,
venv+aiohttp, Port 8090) — Service **active**, lokal HTTP 200, **von außen HTTP 200**
(internet-erreichbar). Gameplay-Feinheiten (v1) über Playtest iterieren. Repo bisher **lokal**
(GitHub-Anlage offen). Bekannte v1-Grenzen: kein Reconnect mitten im Spiel (bei Abbruch neuer Raum).
