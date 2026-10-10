# Changelog

Alle bedeutenden Änderungen an diesem Projekt werden in dieser Datei dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/de/1.0.0/),
und dieses Projekt folgt [Semantic Versioning](https://semver.org/lang/de/).

## [Unreleased]

Basis ist jetzt [ForumSchlampe/Invoke-DHCPCheckup](https://github.com/ForumSchlampe/Invoke-DHCPCheckup) (Stand 2025-01-29) statt des inaktiven Akamai-Originals.

### Hinzugefügt
- Funktion `Check-DhcpDynamicNameUpdateSettings`: prüft DNS-Dynamic-Update-Einstellungen auf Server- und Scope-Ebene (IPv4/IPv6) (diecknet, Akamai-PR #16)

### Geändert
- `GetStrongUsers` erkennt privilegierte Gruppen über SIDs statt englischer Namen, funktioniert damit auch in deutschsprachigen AD-Umgebungen (ForumSchlampe, Akamai-PR #13)
- "Authenticated Users" wird über die SID `S-1-5-11` aufgelöst statt über den englischen Namen
- `Get-ADGroupMember` wird rekursiv abgefragt, verschachtelte Mitgliedschaften werden erkannt

### Behoben
- Falsche SID für Domain Admins (`-516` statt `-512`) (michaelbast)
- Von DHCP-Credentials besessene DNS-Einträge wurden unter PowerShell 7 nicht erkannt (`.Split('\\')` trennte dort am Doppel-Backslash statt am Backslash)
- Bei nur einem aktiven DHCP-Server wurden die Credential-Paare falsch zerlegt; Credentials mit Leerzeichen brachen den String-Split
- Nicht erreichbare DHCP-Server werden per `try/catch` mit `-ErrorAction Stop` erkannt statt über das globale `$error`-Array
- `Check-DnsUpdateProxyMembership` und `Find-VulnerableDnsRecords` bekommen `domainName`/`dnsServerName` als Parameter statt über den Scope der Hauptfunktion

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

[Unreleased]: https://github.com/feiglein74/Invoke-DHCPCheckup/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/feiglein74/Invoke-DHCPCheckup/releases/tag/v1.0.0
