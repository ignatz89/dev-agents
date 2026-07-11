# Protokoll: spiele — thread-03

<!-- Modus: Implementer + Reviewer (Sparring) -->
<!-- Ziel: Mensch ärgere dich nicht — Einhorn-Nacht-Brett (rund/klassisch), Persistenz -->
<!-- Dateien: games/aergern/index.html, server.py -->
<!-- Start: 2026-07-11 -->

---

## [1] Human → Implementer · open
<!-- runde: 0 -->

Brett: runde Grundform, aber klassischer Look. Design: **Einhorn**, dunkel/Nacht/Lila,
glitzernd (dunkle Version, nicht hell-weiß). Spielstand soll erhalten bleiben, auch wenn
man die Session verlässt (2–4 Spieler, online, Würfel, alle Regeln, rundenbasiert).

---

## [2] Implementer → Reviewer · IMPL · needs-review
<!-- runde: 1 -->

- **Brett (aergern):** dunkles Lila/Nacht-Theme, glitzernder Sternenhimmel (Twinkle-Loop 30 fps),
  leuchtendes Einhorn (Silhouette + Spiralhorn + Sichelmond) als Ziel in der Mitte; klassisch-runder
  40er-Ring, farbige Startfelder/Zielwege, Höfe als Panels, Figuren als leuchtende Edelsteine.
- **Persistenz (server.py):** stabile `pid` (localStorage) als Mitglieds-ID → Reconnect behält
  Platz/Farbe/Zug; letzter State je Raum in Speicher **+ Platte** (`data/rooms/`, überlebt Neustart,
  TTL 7 Tage), Nachlieferung beim (Wieder-)Beitritt. Client: Host-Migration (`setHost` aus `peers`,
  Zug-Cache heilt aus State).

---

## [3] Reviewer → Implementer · REVIEW · needs-changes
<!-- runde: 1 -->

- 🔴 **Host-Panel aktualisiert nicht mehr:** `bcast()` rief kein Render fürs eigene Seitenpanel;
  da der Server dem Sender kein Echo schickt, sah der Host nach eigener Aktion Würfel/Zug/Button
  veraltet. → `bcast()` ruft jetzt `syncDOM()`.
- 🟢 Balance-/Sanity-Check des Client-JS ok; Spiel-Logik (legalFor/hostRoll/hostMove) unverändert.

---

## [4] Implementer → Human · SUMMARY · done
<!-- runde: 2 -->

Umgesetzt + deployed (Hetzner, tar→/opt/spiele, `nobody`). **Verifiziert:** Persistenz-Smoke-Test
grün (State → trennen → Wiederbeitritt liefert exakt denselben Stand von Platte), Dienst active,
Lobby + aergern extern HTTP 200. Commit `d1fa1ea` (games-Repo). Security-Notiz zur Platten-Schreib-
Fläche in `plans/security/2026-07-10_spiele.md` ergänzt.
