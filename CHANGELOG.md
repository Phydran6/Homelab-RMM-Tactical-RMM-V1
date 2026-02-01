# Changelog

Alle wichtigen Änderungen an diesem Projekt werden in dieser Datei dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/de/1.0.0/),
und dieses Projekt folgt [Semantic Versioning](https://semver.org/lang/de/).

## [Unreleased]

### Geplant
- Automatische Zertifikat-Erneuerung via Cron
- Integration mit zusätzlichen Monitoring-Tools
- Backup-Scripts für TRMM-Datenbank
- Support für Rocky Linux / AlmaLinux

## [1.0.0] - 2026-02-01

### Added
- Initial Release
- `update-check.sh` - Script zur Update-Prüfung mit Kategorisierung
- `server-update.sh` - Script zur automatischen Installation unkritischer Updates
- Vollständige Homelab-Infrastruktur Dokumentation
- Tactical RMM Docker Deployment Anleitung
- SSL-Zertifikat Management via Certbot
- Pi-hole Konfigurations-Guidelines
- Troubleshooting-Dokumentation
- Quick Reference Guide

### Features
- Automatische Kategorisierung von Updates (KRITISCH/MITTEL/NIEDRIG)
- Sichere automatische Installation unkritischer Pakete
- Manuelle Kontrolle über kritische System-Updates
- Docker Compose v2 Installation Guide
- Wildcard SSL-Zertifikate Setup
- MeshCentral Remote Access Integration
- Nginx Proxy Manager Konfiguration

### Documentation
- Vollständige Markdown-Dokumentation
- PDF-Version der Dokumentation
- README mit Quick Start Guide
- Contribution Guidelines
- MIT License

## Versionshistorie

### Version Nummern Erklärung

- **MAJOR** (1.x.x): Breaking Changes, große neue Features
- **MINOR** (x.1.x): Neue Features, abwärtskompatibel
- **PATCH** (x.x.1): Bugfixes, kleine Verbesserungen

[Unreleased]: https://github.com/yourusername/homelab-infrastructure/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/yourusername/homelab-infrastructure/releases/tag/v1.0.0
