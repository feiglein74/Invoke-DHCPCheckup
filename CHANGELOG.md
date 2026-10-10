# Changelog

Alle bedeutenden Änderungen an diesem Projekt werden in dieser Datei dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/de/1.0.0/),
und dieses Projekt folgt [Semantic Versioning](https://semver.org/lang/de/).

## [Unreleased]

### Geplant
- Export-Funktion für Audit-Ergebnisse
- Verbose und Quiet Modi
- Logging-Funktionalität

## [1.0.0] - 2025-11-15

### Hinzugefügt
- Initiale Veröffentlichung des Invoke-DHCPCheckup Tools
- Funktion `GetActiveActiveDhcpServers` zur Erkennung aktiver DHCP-Server
- Funktion `GetStrongUsers` zur Identifizierung privilegierter AD-Gruppen
- Funktion `Check-DnsCredentialSettings` zur Überprüfung der DNS-Anmeldeinformationen
- Funktion `Check-DhcpNameProtectionSettings` zur Audit von Namensschutz-Einstellungen
- Funktion `Check-DnsUpdateProxyMembership` zur Überprüfung der DNSUpdateProxy-Gruppe
- Funktion `Find-VulnerableDnsRecords` zum Scannen anfälliger DNS-Einträge
- Hauptfunktion `Invoke-DHCPCheckup` als zentrale Orchestrierung
- Unterstützung für IPv4 und IPv6 DHCP-Scopes
- ASCII-Art Banner für professionelle Ausgabe
- Umfassende Fehlerbehandlung bei nicht erreichbaren DHCP-Servern
- Parameter-Validierung für domainName und dnsServerName
- README.md mit deutscher Dokumentation
- TODO.md zur Projektverwaltung
- CHANGELOG.md zur Versionsverfolgung
- CLAUDE.md für AI-Assistenz
- .gitignore für Git-Repository

### Sicherheit
- Read-Only Implementierung - keine Systemänderungen
- Identifizierung von privilegierten Credentials in DHCP-Konfiguration
- Warnung bei fehlenden Namensschutz-Einstellungen
- Erkennung von DNS-Einträgen mit schwachen ACLs
- Überprüfung auf "Authenticated Users" Schreibrechte

---

## Versionshinweise

### Semantic Versioning Erklärung

Dieses Projekt verwendet Semantic Versioning (MAJOR.MINOR.PATCH):

- **MAJOR** (1.x.x): Inkompatible API-Änderungen
- **MINOR** (x.1.x): Neue Funktionalität (abwärtskompatibel)
- **PATCH** (x.x.1): Bugfixes (abwärtskompatibel)

### Kategorien

- **Hinzugefügt** - Neue Features
- **Geändert** - Änderungen an bestehender Funktionalität
- **Veraltet** - Bald zu entfernende Features
- **Entfernt** - Entfernte Features
- **Behoben** - Bugfixes
- **Sicherheit** - Sicherheitsrelevante Änderungen

---

[Unreleased]: https://github.com/user/repo/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/user/repo/releases/tag/v1.0.0
