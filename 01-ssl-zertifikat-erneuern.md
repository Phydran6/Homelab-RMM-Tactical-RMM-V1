# SSL-Zertifikat erneuern für Tactical RMM (Docker) mit Strato DNS

> **Letzte Aktualisierung:** Februar 2026  
> **Zertifikat gültig bis:** 05. Mai 2026  
> **Nächste Erneuerung:** ca. 05. April 2026

## Übersicht

| Komponente | Details |
|------------|---------|
| RMM-VM | Debian mit Docker, User `rmm` (sudo), Verzeichnis `~/trmm` |
| NPM-VM | Separate VM mit Nginx Proxy Manager |
| Domain | phytech.de bei Strato (kein DNS-API!) |
| Subdomains | rmm.phytech.de, api.phytech.de, mesh.phytech.de |

## Voraussetzungen

- SSH-Zugang zur RMM-VM
- Zugang zum Strato Kundenbereich (DNS-Verwaltung)
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
  -d "*.phytech.de" \
  -m it@phytech.de \
  --agree-tos
```

Certbot zeigt einen Challenge-Wert an:

```
Please deploy a DNS TXT record under the name:
_acme-challenge.phytech.de
with the following value:
9ihDbjYfTExAYeDs4DBUeuTo18KBzwvTEjUnSwd32-c
```

> ⚠️ **NOCH NICHT ENTER DRÜCKEN!**

### 1.3 TXT-Record bei Strato anlegen

1. [Strato Kundenbereich](https://www.strato.de/apps/CustomerService) öffnen
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
dig TXT _acme-challenge.phytech.de
```

Warte bis der richtige Wert erscheint (bei Strato ca. **5-15 Minuten**).

Alternative Prüfung über Google DNS:
```bash
dig TXT _acme-challenge.phytech.de @8.8.8.8
```

Erst dann in Certbot **Enter drücken**.

### 1.5 Zertifikate prüfen

```bash
sudo ls -la /etc/letsencrypt/live/phytech.de/
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

### 2.1 Berechtigungen fixen (einmalig)

```bash
# Das Verzeichnis gehört evtl. root, daher einmalig fixen:
sudo chown -R rmm:rmm ~/trmm

# Prüfen ob es geklappt hat:
ls -la ~/trmm/
```

Alle Dateien sollten jetzt `rmm rmm` zeigen.

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
echo "CERT_PUB_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/phytech.de/fullchain.pem)" >> .env

# Private Key (privkey.pem) als Base64:
echo "CERT_PRIV_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/phytech.de/privkey.pem)" >> .env
```

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
echo | openssl s_client -servername api.phytech.de -connect api.phytech.de:443 2>/dev/null | openssl x509 -noout -dates
```

Sollte das neue Ablaufdatum zeigen (ca. 90 Tage in der Zukunft).

---

## Teil 3: Zertifikat in NPM aktualisieren

### 3.1 Zertifikat-Dateien anzeigen

Auf der **RMM-VM** - jeden Befehl **einzeln** ausführen und Ausgabe separat kopieren:

```bash
# 1. Certificate (für NPM-Feld "Certificate"):
sudo cat /etc/letsencrypt/live/phytech.de/cert.pem
```
→ Ausgabe kopieren (alles von `-----BEGIN CERTIFICATE-----` bis `-----END CERTIFICATE-----`)

```bash
# 2. Certificate Key (für NPM-Feld "Certificate Key"):
sudo cat /etc/letsencrypt/live/phytech.de/privkey.pem
```
→ Ausgabe kopieren (alles von `-----BEGIN PRIVATE KEY-----` bis `-----END PRIVATE KEY-----`)

```bash
# 3. Intermediate Certificate (für NPM-Feld "Intermediate Certificate"):
sudo cat /etc/letsencrypt/live/phytech.de/chain.pem
```
→ Ausgabe kopieren (alles von `-----BEGIN CERTIFICATE-----` bis `-----END CERTIFICATE-----`)

### 3.2 In NPM importieren

1. NPM Web-UI öffnen (http://npm-ip:81)
2. **SSL Certificates** → **Add SSL Certificate** → **Custom**
3. Felder ausfüllen:

| NPM-Feld | Datei-Inhalt |
|----------|--------------|
| Certificate | `cert.pem` |
| Certificate Key | `privkey.pem` |
| Intermediate Certificate | `chain.pem` |

4. Speichern

### 3.3 Proxy Hosts aktualisieren

Für alle drei Proxy Hosts:
- `rmm.phytech.de`
- `api.phytech.de`
- `mesh.phytech.de`

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
sudo certbot certonly --manual --preferred-challenges dns-01 --manual-public-ip-logging-ok -d "*.phytech.de" -m it@phytech.de --agree-tos

# DNS prüfen (in neuem Terminal):
dig TXT _acme-challenge.phytech.de

# Zertifikate prüfen:
sudo ls -la /etc/letsencrypt/live/phytech.de/

# ============================================
# TEIL 2: Cert in TRMM eintragen
# ============================================
sudo chown -R rmm:rmm ~/trmm
cd ~/trmm
cp .env .env.backup.$(date +%Y%m%d)
sed -i '/CERT_PUB_KEY/d' .env
sed -i '/CERT_PRIV_KEY/d' .env

echo "CERT_PUB_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/phytech.de/fullchain.pem)" >> .env
echo "CERT_PRIV_KEY=$(sudo base64 -w 0 /etc/letsencrypt/live/phytech.de/privkey.pem)" >> .env

grep "CERT_" .env

sudo docker compose down
sudo docker compose up -d

# ============================================
# TEIL 3: Zertifikate für NPM (einzeln ausführen!)
# ============================================
# 1. Certificate:
sudo cat /etc/letsencrypt/live/phytech.de/cert.pem

# 2. Certificate Key:
sudo cat /etc/letsencrypt/live/phytech.de/privkey.pem

# 3. Intermediate Certificate:
sudo cat /etc/letsencrypt/live/phytech.de/chain.pem
```

---

## Wichtige Hinweise

### Let's Encrypt Zertifikate - 90 Tage Gültigkeit

| Info | Datum |
|------|-------|
| Aktuelles Zertifikat läuft ab | **05. Mai 2026** |
| Erinnerung setzen für | **ca. 05. April 2026** |

Da Strato keine DNS-API hat, muss die Erneuerung jedes Mal manuell erfolgen.

> 💡 Der TXT-Record `_acme-challenge` bei Strato kann nach erfolgreicher Zertifikatserstellung gelöscht werden.

### Alternative: DNS zu Cloudflare migrieren

Für automatische Wildcard-Zertifikat-Erneuerung:
1. Kostenlosen Cloudflare Account erstellen
2. Domain `phytech.de` hinzufügen (Cloudflare kopiert bestehende Records)
3. Nameserver bei Strato auf Cloudflare ändern
4. In NPM: Cloudflare DNS Challenge nutzen → vollautomatische Erneuerung

Strato bleibt dabei Registrar, nur die Nameserver zeigen auf Cloudflare.

---

## Troubleshooting

### Zertifikat wird nicht erkannt nach Neustart

```bash
sudo docker compose logs tactical-nginx
```

### Permission denied bei docker compose

```bash
# Entweder sudo verwenden oder User zur docker-Gruppe hinzufügen:
sudo usermod -aG docker rmm
# Danach neu einloggen!
```

### DNS-Challenge schlägt fehl

- Prüfe ob der TXT-Record korrekt angelegt wurde
- Warte länger (manchmal dauert Strato bis zu 30 Minuten)
- Prüfe mit Google DNS: `dig TXT _acme-challenge.phytech.de @8.8.8.8`

### Zertifikat in NPM wird nicht akzeptiert

- Stelle sicher, dass du den **kompletten** Inhalt kopiert hast (inkl. `-----BEGIN...` und `-----END...`)
- Prüfe ob Certificate und Private Key zusammenpassen
- Intermediate Certificate nicht vergessen
