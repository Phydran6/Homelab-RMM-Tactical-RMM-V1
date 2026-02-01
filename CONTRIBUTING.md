# Contributing to Homelab Infrastructure

Vielen Dank für dein Interesse an diesem Projekt! 🎉

## 🤝 Wie kann ich beitragen?

### Reporting Bugs

Wenn du einen Bug findest:

1. **Prüfe**, ob der Bug bereits als Issue existiert
2. **Erstelle ein neues Issue** mit folgenden Informationen:
   - Beschreibung des Problems
   - Steps to Reproduce
   - Erwartetes Verhalten
   - Aktuelles Verhalten
   - System-Informationen (OS, Version, etc.)
   - Logs (falls relevant)

### Feature Requests

Feature-Ideen sind willkommen:

1. **Erstelle ein Issue** mit dem Label `enhancement`
2. **Beschreibe**:
   - Was soll das Feature tun?
   - Warum ist es nützlich?
   - Wie könnte es implementiert werden?

### Pull Requests

#### Vorbereitung

1. **Fork** das Repository
2. **Clone** deinen Fork:
   ```bash
   git clone https://github.com/dein-username/homelab-infrastructure.git
   cd homelab-infrastructure
   ```
3. **Erstelle einen Branch**:
   ```bash
   git checkout -b feature/mein-feature
   # oder
   git checkout -b fix/mein-bugfix
   ```

#### Code-Standards

**Bash Scripts:**
- Verwende `#!/bin/bash` als Shebang
- Füge Header-Kommentare hinzu (Autor, Version, Beschreibung)
- Verwende aussagekräftige Variablennamen
- Kommentiere komplexe Logik
- Teste mit ShellCheck: `shellcheck script.sh`

**Dokumentation:**
- Markdown für alle `.md` Dateien
- Deutsche Sprache (Projekt-Standard)
- Klare, präzise Formulierungen
- Code-Beispiele wo sinnvoll

**Commits:**
- Aussagekräftige Commit-Messages
- Format: `type: subject`
- Typen: `feat`, `fix`, `docs`, `style`, `refactor`, `test`
- Beispiel: `feat: add automatic backup script`

#### Testing

Vor dem PR:

1. **Teste deine Änderungen** auf Debian 12
2. **Führe ShellCheck aus**:
   ```bash
   shellcheck scripts/*.sh
   ```
3. **Prüfe die Dokumentation** auf Tippfehler und Vollständigkeit

#### Pull Request einreichen

1. **Push** deinen Branch:
   ```bash
   git push origin feature/mein-feature
   ```
2. **Erstelle einen PR** auf GitHub
3. **Beschreibe** deine Änderungen:
   - Was wurde geändert?
   - Warum wurde es geändert?
   - Wie wurde es getestet?

## 📋 Checkliste für PRs

- [ ] Code folgt den Projekt-Standards
- [ ] ShellCheck gibt keine Warnungen
- [ ] Dokumentation ist aktualisiert
- [ ] Auf Debian 12 getestet
- [ ] Commit-Messages sind aussagekräftig
- [ ] Keine sensiblen Daten (Passwörter, IPs) im Code

## 🎯 Bereiche für Contributions

Wir suchen besonders nach Hilfe in:

- **Testing** auf anderen Debian/Ubuntu Versionen
- **Dokumentation** - Verbesserungen, Übersetzungen
- **Features** - Neue Update-Kategorien, Monitoring-Integrationen
- **Bugfixes** - Edge Cases, Kompatibilitätsprobleme
- **Scripts** - Zusätzliche Automatisierungs-Scripts

## ❓ Fragen?

Bei Fragen:
- Erstelle ein Issue mit dem Label `question`
- Kontaktiere den Maintainer: it@phytech.de

## 📜 Code of Conduct

- Sei respektvoll und konstruktiv
- Hilf anderen Contributoren
- Fokus auf technische Diskussionen
- Keine Diskriminierung oder Belästigung

Vielen Dank für deine Unterstützung! 🚀
