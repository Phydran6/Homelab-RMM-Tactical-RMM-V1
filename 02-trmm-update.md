# Tactical RMM Update (Docker) - v1.3.1 → v1.4.0

> **Letzte Aktualisierung:** Februar 2026  
> **Quellversion:** v1.3.1  
> **Zielversion:** v1.4.0  
> **Getestet:** ✅ Erfolgreich

## Übersicht

| Komponente | Details |
|------------|---------|
| RMM-VM | Debian mit Docker, User `rmm` (sudo), Verzeichnis `~/trmm` |
| Aktuelle Version | v1.3.1 |
| Zielversion | v1.4.0 |
| Agent-Version (neu) | 2.10.0 |

## Voraussetzungen

- [x] SSH-Zugang zur RMM-VM
- [x] Proxmox Snapshot erstellt (Rollback-Möglichkeit)
- [x] SSL-Zertifikat aktuell (siehe separate Anleitung)

---

## Wichtige Änderungen in v1.4.0

- **3-4× schnelleres Dashboard-Loading**
- Neuer Windows Registry Editor
- Docker-Verbesserungen für Session/CSRF Domain Handling
- Agent-Version 2.10.0 (kann Antivirus-Fehlalarme auslösen!)

---

## Schritt 1: Automatische Agent-Updates deaktivieren

> ⚠️ **WICHTIG:** Vor dem Update unbedingt durchführen!

Die neue Agent-Version 2.10.0 kann bei einigen Antivirus-Programmen Fehlalarme auslösen. Daher erst manuell testen.

1. Im Browser `rmm.phytech.de` öffnen
2. **Settings** → **Global Settings**
3. **"Enable agent automatic self update"** → **Deaktivieren**
4. Speichern

---

## Schritt 2: Update durchführen

> 💡 **Hinweis:** Da `VERSION=latest` in der `.env` gesetzt ist, reicht es die neuen Images zu pullen. Die `docker-compose.yml` muss nicht geändert werden.

```bash
cd ~/trmm

# Neue Images holen (VERSION=latest zieht automatisch neueste Version)
sudo docker compose pull

# Container neu starten mit neuen Images
sudo docker compose down
sudo docker compose up -d

# Status prüfen - alle Container sollten "Up" oder "running" zeigen
sudo docker compose ps
```

---

## Schritt 3: Alte Images aufräumen

Nach dem Update bleiben alte Images auf der Festplatte. Diese können gelöscht werden:

```bash
# Zeigt ungenutzte Images an
sudo docker images

# Alle ungenutzten Images löschen
sudo docker image prune -a

# Bestätigen mit 'y'
```

Alternativ nur bestimmte Images löschen:

```bash
# Bestimmtes Image löschen (Image-ID aus 'docker images' kopieren)
sudo docker rmi <IMAGE_ID>

# Mehrere auf einmal
sudo docker rmi <IMAGE_ID_1> <IMAGE_ID_2> <IMAGE_ID_3>
```

Komplettes Docker-Cleanup (Images, Container, Volumes, Networks):

```bash
# VORSICHT: Löscht alles Ungenutzte!
sudo docker system prune -a
```

---

## Schritt 4: Update verifizieren

### 4.1 Web-UI prüfen

1. Im Browser `rmm.phytech.de` öffnen
2. Oben links sollte **v1.4.0** stehen

### 4.2 Container-Status prüfen

```bash
sudo docker compose ps
```

Alle Container sollten `Up` oder `running` zeigen.

---

## Schritt 5: Agents testen (optional)

> 💡 **Hinweis:** Falls die Agents bereits auf Version 2.10.0 sind, ist dieser Schritt nicht nötig. Prüfe in der Web-UI unter **Agents** die Agent-Version.

Falls Agents noch auf alter Version:

1. In der Web-UI: **Agents** → 2-3 Test-Agents auswählen
2. **Update Agents** klicken
3. Ein paar Minuten warten
4. Prüfen ob Agent-Version **2.10.0** zeigt
5. Prüfen ob Antivirus Alarm schlägt

### Bei Erfolg:

**Settings** → **Global Settings** → **"Enable agent automatic self update"** → **Aktivieren**

---

## Schnellreferenz - Alle Befehle

```bash
# ============================================
# TRMM Update v1.3.1 → v1.4.0
# ============================================

# 1. In Web-UI: Automatische Agent-Updates DEAKTIVIEREN!

# 2. Update durchführen
cd ~/trmm
sudo docker compose pull
sudo docker compose down
sudo docker compose up -d

# 3. Status prüfen
sudo docker compose ps

# 4. Alte Images aufräumen
sudo docker image prune -a

# 5. In Web-UI: Version prüfen (sollte v1.4.0 zeigen)
# 6. In Web-UI: Agent-Versionen prüfen (evtl. schon aktuell)
# 7. In Web-UI: Automatische Agent-Updates wieder AKTIVIEREN
```

---

## Troubleshooting

### Update zeigt alte Version

```bash
# Images wurden nicht neu gezogen. Nochmal pullen:
sudo docker compose pull
sudo docker compose down
sudo docker compose up -d
```

### Container starten nicht

```bash
# Logs prüfen
sudo docker compose logs

# Einzelnen Container prüfen (z.B. backend)
sudo docker compose logs tactical-backend
```

### Agent-Update schlägt fehl

1. Antivirus temporär deaktivieren
2. Agent manuell updaten
3. Antivirus-Ausnahme für Tactical Agent hinzufügen
4. Antivirus wieder aktivieren

### Rollback auf v1.3.1

Falls alles schiefgeht → Proxmox Snapshot zurückspielen.

---

## Nach dem Update

- [x] Version v1.4.0 in Web-UI bestätigt
- [x] Alle Container laufen
- [x] Agent-Versionen geprüft (waren bereits aktuell)
- [x] Automatische Agent-Updates wieder aktiviert
- [ ] Proxmox Snapshot kann gelöscht werden (oder behalten für Sicherheit)
- [x] Alte Docker Images aufgeräumt

---

## Weiterführende Links

- [Tactical RMM Dokumentation](https://docs.tacticalrmm.com/)
- [Docker Update Anleitung (offiziell)](https://docs.tacticalrmm.com/update_docker/)
- [Release Notes v1.4.0](https://github.com/amidaware/tacticalrmm/releases/tag/v1.4.0)
- [GitHub Issues](https://github.com/amidaware/tacticalrmm/issues)
