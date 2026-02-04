# SSL-Zertifikat erneuern für Tactical RMM (Docker) mit manueller DNS-Challenge

> **Anwendungsfall:** Wildcard-Zertifikat für Tactical RMM Docker Setup mit externem Reverse Proxy (z.B. Nginx Proxy Manager) bei einem DNS-Provider ohne API-Zugang.

## Übersicht

| Komponente | Details |
|------------|---------|
| RMM-VM | Debian/Ubuntu mit Docker, Verzeichnis `~/trmm` |
| Reverse Proxy | Nginx Proxy Manager (separate VM/Container) |
| Domain | `example.com` (bei DNS-Provider ohne API) |
| Subdomains | `rmm.example.com`, `api.example.com`, `mesh.example.com` |

## Voraussetzungen

- SSH-Zugang zur RMM-VM
- Zugang zum DNS-Provider (für TXT-Records)
- Zugang zur NPM Web-UI

---

## Teil 1: Wildcard-Zertifikat mit Certbot erstellen

### 1.1 Certbot prüfen/installieren

```bash
# Prüfen ob certbot installiert ist:
certbot --version

# Falls nicht installiert:
sudo apt update
sudo apt install certbot -y
```

### 1.2 Wildcard-Zertifikat anfordern

```bash
sudo certbot certonly --manual \
  --preferred-challenges dns-01 \
  --manual-public-ip-logging-ok \
  -d "*.example.com" \
  -m admin@example.com \
  --agree-tos
```

> 💡 Ersetze `example.com` mit deiner Domain und `admin@example.com` mit deiner E-Mail.

Certbot zeigt einen Challenge-Wert an:

```
Please deploy a DNS TXT record under the name:
_acme-challenge.example.com
with the following value:
9ihDbjYfTExAYeDs4DBUeuTo18KBzwvTEjUnSwd32-c
```

> ⚠️ **NOCH NICHT ENTER DRÜCKEN!**

### 1.3 TXT-Record beim DNS-Provider anlegen

1. DNS-Provider Kundenbereich öffnen
2. **Domain-Verwaltung** → **DNS-Einstellungen**
3. Neuen TXT-Record erstellen:

| Feld | Wert |
|------|------|
| Subdomain/Präfix | `_acme-challenge` |
| Typ | TXT |
| Wert | Challenge-Wert von Certbot |

### 1.4 DNS-Propagation prüfen

```bash
# In einem neuen Terminal-Fenster:
dig TXT _acme-challenge.example.com
```

Warte bis der richtige Wert erscheint (ca. **5-30 Minuten** je nach Provider).

Alternative Prüfung über Google DNS:
```bash
dig TXT _acme-challenge.example.com @8.8.8.8
```

Erst dann in Certbot **Enter drücken**.

### 1.5 Zertifikate prüfen

```bash
sudo ls -la /etc/letsencrypt/live/example.com/
```

Erwartete Dateien:

| Datei | Beschreibung |
|-------|--------------|
| `cert.pem` | Dein Domain-Zertifikat |
| `privkey.pem` | Private Key |
| `chain.pem` | Intermediate Certificate |
| `fullchain.pem` | cert.pem + chain.pem kombiniert |

---

## Teil 2: Zertifikate in Tactical RMM eintragen

### 2.1 Berechtigungen fixen (falls nötig)

```bash
# Falls Verzeichnis root gehört:
sudo chown -R $USER:$USER ~/trmm

# Prüfen:
ls -la ~/trmm/
```

### 2.2 Alte Zertifikat-Einträge entfernen

```bash
cd ~/trmm

# Backup machen
cp .env .env.backup.$(date +%Y%m%d)

# Alte CERT-Zeilen entfernen
sed -i '/CERT_PUB_KEY/d' .env
sed -i '/CERT_PRIV_KEY/d' .env
```

### 2.3 Neue Zertifikate als Base64 eintragen

```bash
cd ~/trmm

# Public Key (fullchain.pem) als Base64:
echo "CERT_PUB_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/example.com/fullchain.pem)" >> .env

# Private Key (privkey.pem) als Base64:
echo "CERT_PRIV_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/example.com/privkey.pem)" >> .env
```

> 💡 Ersetze `example.com` mit deiner Domain.

### 2.4 Prüfen ob alles stimmt

```bash
# Sollte genau EINE Zeile für jeden Key zeigen:
grep "CERT_" .env
```

### 2.5 Container neu starten

```bash
cd ~/trmm
sudo docker compose down
sudo docker compose up -d
```

### 2.6 Zertifikat testen

```bash
echo | openssl s_client -servername api.example.com -connect api.example.com:443 2>/dev/null | openssl x509 -noout -dates
```

Sollte das neue Ablaufdatum zeigen (ca. 90 Tage in der Zukunft).

---

## Teil 3: Zertifikat in NPM aktualisieren

### 3.1 Zertifikat-Dateien anzeigen

Jeden Befehl **einzeln** ausführen und Ausgabe separat kopieren:

```bash
# 1. Certificate (für NPM-Feld "Certificate"):
sudo cat /etc/letsencrypt/live/example.com/cert.pem
```
→ Ausgabe kopieren (alles von `-----BEGIN CERTIFICATE-----` bis `-----END CERTIFICATE-----`)

```bash
# 2. Certificate Key (für NPM-Feld "Certificate Key"):
sudo cat /etc/letsencrypt/live/example.com/privkey.pem
```
→ Ausgabe kopieren (alles von `-----BEGIN PRIVATE KEY-----` bis `-----END PRIVATE KEY-----`)

```bash
# 3. Intermediate Certificate (für NPM-Feld "Intermediate Certificate"):
sudo cat /etc/letsencrypt/live/example.com/chain.pem
```
→ Ausgabe kopieren (alles von `-----BEGIN CERTIFICATE-----` bis `-----END CERTIFICATE-----`)

### 3.2 In NPM importieren

1. NPM Web-UI öffnen
2. **SSL Certificates** → **Add SSL Certificate** → **Custom**
3. Felder ausfüllen:

| NPM-Feld | Datei-Inhalt |
|----------|--------------|
| Certificate | `cert.pem` |
| Certificate Key | `privkey.pem` |
| Intermediate Certificate | `chain.pem` |

4. Speichern

> 💡 Falls NPM nur 2 Felder hat (ohne Intermediate), nutze `fullchain.pem` statt `cert.pem`.

### 3.3 Proxy Hosts aktualisieren

Für alle drei Proxy Hosts:
- `rmm.example.com`
- `api.example.com`
- `mesh.example.com`

Jeweils:
1. Bearbeiten
2. **SSL-Tab** → neues Custom-Zertifikat auswählen
3. Speichern

> 💡 Das alte Custom-Zertifikat kann danach gelöscht werden.

---

## Schnellreferenz - Alle Befehle

```bash
# ============================================
# TEIL 1: Zertifikat anfordern
# ============================================
sudo certbot certonly --manual --preferred-challenges dns-01 --manual-public-ip-logging-ok -d "*.example.com" -m admin@example.com --agree-tos

# DNS prüfen (in neuem Terminal):
dig TXT _acme-challenge.example.com

# Zertifikate prüfen:
sudo ls -la /etc/letsencrypt/live/example.com/

# ============================================
# TEIL 2: Cert in TRMM eintragen
# ============================================
sudo chown -R $USER:$USER ~/trmm
cd ~/trmm
cp .env .env.backup.$(date +%Y%m%d)
sed -i '/CERT_PUB_KEY/d' .env
sed -i '/CERT_PRIV_KEY/d' .env

echo "CERT_PUB_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/example.com/fullchain.pem)" >> .env
echo "CERT_PRIV_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/example.com/privkey.pem)" >> .env

grep "CERT_" .env

sudo docker compose down
sudo docker compose up -d

# ============================================
# TEIL 3: Zertifikate für NPM (einzeln ausführen!)
# ============================================
# 1. Certificate:
sudo cat /etc/letsencrypt/live/example.com/cert.pem

# 2. Certificate Key:
sudo cat /etc/letsencrypt/live/example.com/privkey.pem

# 3. Intermediate Certificate:
sudo cat /etc/letsencrypt/live/example.com/chain.pem
```

---

## Wichtige Hinweise

### Let's Encrypt Zertifikate - 90 Tage Gültigkeit

- Erinnerung setzen für ca. **30 Tage vor Ablauf**
- Bei DNS-Providern ohne API muss die Erneuerung jedes Mal manuell erfolgen
- TXT-Record `_acme-challenge` kann nach erfolgreicher Zertifikatserstellung gelöscht werden

### Alternative: DNS zu Cloudflare migrieren

Für automatische Wildcard-Zertifikat-Erneuerung:
1. Kostenlosen Cloudflare Account erstellen
2. Domain hinzufügen (Cloudflare kopiert bestehende Records)
3. Nameserver beim Registrar auf Cloudflare ändern
4. In NPM: Cloudflare DNS Challenge nutzen → vollautomatische Erneuerung

Der bisherige Registrar bleibt erhalten, nur die Nameserver zeigen auf Cloudflare.

---

## Troubleshooting

### Zertifikat wird nicht erkannt nach Neustart

```bash
sudo docker compose logs tactical-nginx
```

### Permission denied bei docker compose

```bash
# Entweder sudo verwenden oder User zur docker-Gruppe hinzufügen:
sudo usermod -aG docker $USER
# Danach neu einloggen!
```

### DNS-Challenge schlägt fehl

- Prüfe ob der TXT-Record korrekt angelegt wurde
- Warte länger (manche Provider brauchen bis zu 30 Minuten)
- Prüfe mit Google DNS: `dig TXT _acme-challenge.example.com @8.8.8.8`

### Zertifikat in NPM wird nicht akzeptiert

- Stelle sicher, dass du den **kompletten** Inhalt kopiert hast (inkl. `-----BEGIN...` und `-----END...`)
- Prüfe ob Certificate und Private Key zusammenpassen
- Intermediate Certificate nicht vergessen (falls NPM 3 Felder hat)
