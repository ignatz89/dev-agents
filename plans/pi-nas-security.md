# Plan: pi-nas — Security-Refactor (Architect, Fokus Sicherheit)

**Repo:** `d:\repos\pi-nas` · **Modus:** Refactor-Pass (Security-Fokus)
**Ziel:** Das über den Cloudflare Tunnel öffentlich erreichbare NAS in Schichten härten,
bevor es produktiv läuft.

## Assets (schützenswert)
- Persönliche Fotos/Videos, Dateien, Kontakte, Kalender (Nextcloud-Daten auf `/mnt/nas`)
- Nextcloud-Admin-Zugang · Datenbank · Cloudflare-Tunnel-Token · `.env`-Secrets

## Attack Surface
- Öffentliche HTTPS-URL über den Cloudflare Tunnel (Login-Seite exponiert)
- Nextcloud im LAN unverschlüsselt (Port 8080) · Docker-Container · SSH auf dem Pi
- Supply Chain (Docker-Images) · Datenverlust (nur eine Kopie der Daten)

## Tasks für den Implementer (priorisiert)

### 🔴 Kritisch
1. **Nextcloud nicht offen im LAN** — Port auf `127.0.0.1:8080` binden. Zugriff nur
   lokal / via Tunnel. *Verify:* `ss -tlnp | grep 8080` → `127.0.0.1`.
2. **Reverse-Proxy korrekt & sicher** — `OVERWRITEPROTOCOL=https`, `OVERWRITECLIURL`,
   und `TRUSTED_PROXIES` **auf das Docker-Netz begrenzt** (nicht `0.0.0.0/0`, sonst
   X-Forwarded-For-Spoofing → Brute-Force-Schutz umgehbar). *Verify:* `occ config:system:get trusted_proxies`.
3. **Cloudflare Zero Trust Access** als Gate vor Nextcloud (Dashboard-Schritt → SECURITY.md).

### 🟡 Mittel
4. **Redis mit Passwort** (`--requirepass` + `REDIS_HOST_PASSWORD`) — Defense in Depth.
5. **Container-Härtung** — `security_opt: no-new-privileges` auf allen Services.
6. **Images pinnen** — `mariadb:10.11`, `redis:7-alpine`; `cloudflared:latest` bewusst
   aktuell (Security-Connector).
7. **Backup** — zweite Kopie der NAS-Daten (`scripts/backup.sh` + cron).
8. **Host-Härtung** dokumentieren — SSH key-only, ufw, unattended-upgrades, fail2ban.

### 🟢 Gering
9. `PHP_UPLOAD_LIMIT`/`PHP_MEMORY_LIMIT` hoch (Foto-/Video-Backup) — Funktion, kein Risiko.
10. `.env`-Rechte `chmod 600`.

## Nicht im Scope
- Live-Deployment/Tests auf dem Pi (macht der User). Dashboard-Schritte (Zero Trust, 2FA)
  sind dokumentiert, aber nicht automatisierbar.

## Definition of Done
Compose gehärtet & valide; SECURITY.md mit Prüfbefehlen; Backup-Skript vorhanden;
Reviewer-Loop ohne offene 🔴/🟡. Host-/Dashboard-Schritte als Checkliste an den User.
