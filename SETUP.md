# GitHub Setup Guide

Diese Anleitung zeigt dir, wie du dieses Repository auf GitHub veröffentlichst.

## 🚀 Repository auf GitHub erstellen

### Schritt 1: GitHub Repository erstellen

1. Gehe zu [GitHub](https://github.com) und melde dich an
2. Klicke auf das **+** Icon oben rechts → **New repository**
3. Repository-Einstellungen:
   - **Repository name:** `homelab-infrastructure`
   - **Description:** `Automatisierte System-Verwaltung und Monitoring für Homelab-Umgebungen`
   - **Visibility:** Public (oder Private)
   - **❌ NICHT** "Initialize this repository with a README" ankreuzen
4. Klicke auf **Create repository**

### Schritt 2: Lokales Repository initialisieren

```bash
cd homelab-infrastructure

# Git initialisieren
git init

# Alle Dateien hinzufügen
git add .

# Initial Commit
git commit -m "Initial commit: Homelab Infrastructure v1.0.0"

# Main Branch als Standard setzen
git branch -M main

# Remote hinzufügen (ersetze 'yourusername' mit deinem GitHub Username)
git remote add origin https://github.com/yourusername/homelab-infrastructure.git

# Zum GitHub Repository pushen
git push -u origin main
```

### Schritt 3: README anpassen

Ersetze in den folgenden Dateien `yourusername` mit deinem GitHub Username:

```bash
# README.md
sed -i 's/yourusername/DEIN_USERNAME/g' README.md

# GitHub Workflows (falls vorhanden)
find .github -type f -name "*.yml" -exec sed -i 's/yourusername/DEIN_USERNAME/g' {} \;

# Git commit und push
git add README.md .github/
git commit -m "docs: update GitHub username in links"
git push
```

## 🏷️ Release erstellen (Optional)

### Tag erstellen und pushen

```bash
# Tag für v1.0.0 erstellen
git tag -a v1.0.0 -m "Release v1.0.0 - Initial Release"

# Tag pushen
git push origin v1.0.0
```

### Release auf GitHub erstellen

1. Gehe zu deinem Repository auf GitHub
2. Klicke auf **Releases** → **Create a new release**
3. Wähle den Tag `v1.0.0`
4. Release Title: `v1.0.0 - Initial Release`
5. Beschreibung:
```markdown
## 🎉 Initial Release

Erste öffentliche Version der Homelab Infrastructure.

### Features
- ✅ Update-Management Scripts mit automatischer Kategorisierung
- 🐳 Tactical RMM Docker Deployment Guide
- 🔐 SSL-Zertifikat Management
- 📖 Vollständige Dokumentation

### Installation
```bash
git clone https://github.com/yourusername/homelab-infrastructure.git
cd homelab-infrastructure
sudo ./install.sh
```

Siehe [README.md](README.md) für Details.
```

6. Klicke auf **Publish release**

## 📋 Repository Settings (Empfohlen)

### Branch Protection Rules

1. Gehe zu **Settings** → **Branches**
2. Klicke auf **Add rule**
3. Branch name pattern: `main`
4. Aktiviere:
   - ✅ Require pull request reviews before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require conversation resolution before merging

### GitHub Actions aktivieren

1. Gehe zu **Settings** → **Actions** → **General**
2. Unter **Actions permissions** wähle:
   - ✅ Allow all actions and reusable workflows
3. Klicke auf **Save**

### Topics hinzufügen

1. Gehe zu deinem Repository
2. Klicke auf **⚙️** neben "About"
3. Füge Topics hinzu:
   - `homelab`
   - `automation`
   - `debian`
   - `docker`
   - `tactical-rmm`
   - `system-administration`
   - `update-management`
   - `bash-scripts`

### GitHub Pages (Optional - für Dokumentation)

1. Gehe zu **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` → `/docs`
4. Klicke auf **Save**

## 🔧 Nach dem Setup

### Erste Schritte nach Veröffentlichung

1. **Teste die Workflows:**
   ```bash
   # Kleine Änderung machen
   echo "Test" >> README.md
   git add README.md
   git commit -m "test: trigger GitHub Actions"
   git push
   ```
   
2. **Prüfe GitHub Actions:**
   - Gehe zu **Actions** Tab
   - Schaue ob ShellCheck läuft

3. **Erstelle erste Issue:**
   - Teste die Issue Templates
   - Gehe zu **Issues** → **New issue**

4. **README Badges updaten:**
   - Nach erstem erfolgreichen Workflow werden Badges aktiv

## 📢 Projekt bewerben

### Nach Veröffentlichung kannst du:

1. **Social Media:**
   - Teile auf Twitter/X, Reddit (r/homelab, r/selfhosted)
   - LinkedIn Post

2. **Communities:**
   - [Awesome Selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)
   - Homelab Discord/Forums

3. **Blog Post:**
   - Schreibe einen Blog-Artikel über dein Setup

## 🆘 Troubleshooting

### Git Authentication Fehler

Wenn `git push` fehlschlägt:

**Option 1: Personal Access Token (empfohlen)**
```bash
# Token auf GitHub erstellen: Settings → Developer settings → Personal access tokens
# Dann verwenden statt Passwort
git remote set-url origin https://TOKEN@github.com/yourusername/homelab-infrastructure.git
```

**Option 2: SSH**
```bash
# SSH Key generieren
ssh-keygen -t ed25519 -C "your_email@example.com"

# Public Key zu GitHub hinzufügen: Settings → SSH and GPG keys

# Remote URL ändern
git remote set-url origin git@github.com:yourusername/homelab-infrastructure.git
```

### Workflow schlägt fehl

1. Prüfe die Logs im **Actions** Tab
2. Häufige Fehler:
   - ShellCheck findet Syntax-Fehler → Scripts korrigieren
   - Permissions → Workflow-Datei prüfen

## ✅ Checkliste

- [ ] GitHub Repository erstellt
- [ ] Initial Commit gepusht
- [ ] Username in allen Dateien ersetzt
- [ ] Release v1.0.0 erstellt
- [ ] Topics hinzugefügt
- [ ] Branch Protection aktiviert
- [ ] GitHub Actions getestet
- [ ] README Badges funktionieren
- [ ] Issue Templates getestet

---

**Viel Erfolg mit deinem Homelab Infrastructure Repository! 🚀**

Bei Fragen: it@phytech.de
