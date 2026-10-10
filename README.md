# Invoke-DHCPCheckup

Ein PowerShell-Sicherheitsaudit-Tool zur Überprüfung von Microsoft DHCP-Servern in Active Directory-Umgebungen.

## Version

**Aktuelle Version:** 1.0.0

## Beschreibung

Invoke-DHCPCheckup führt eine umfassende Sicherheitsüberprüfung aller DHCP-Server in einer Active Directory-Domäne durch und identifiziert häufige Fehlkonfigurationen, die zu DNS-Spoofing-Schwachstellen führen können.

Das Tool erkennt folgende Sicherheitsrisiken:
- Unsichere DHCP DNS-Anmeldeinformationen
- Fehlende oder deaktivierte DHCP-Namensschutz-Einstellungen
- Problematische DNSUpdateProxy-Gruppenmitgliedschaften
- DNS-Einträge mit schwachen Berechtigungen

## Autor

- **Ori David** (@oridavid123)
- **Organisation:** Akamai SIG
- **Blog-Artikel:** https://akamai.com/blog/security-research/spoofing-dns-by-abusing-dhcp

## Voraussetzungen

### System-Anforderungen
- Windows-System mit PowerShell 5.1 oder höher
- Mitglied einer Active Directory-Domäne

### Erforderliche PowerShell-Module
Das Skript benötigt folgende Module (werden automatisch importiert):
- `DHCPServer`
- `ActiveDirectory`
- `DnsServer`

### Berechtigungen
Der ausführende Benutzer benötigt:
- Leserechte auf DHCP-Servern (`Get-DhcpServerInDC`, `Get-DhcpServerSetting`, etc.)
- Leserechte in Active Directory (`Get-ADGroupMember`)
- Leserechte auf DNS-Server-Ressourcen (`Get-DnsServerResourceRecord`)

## Installation

1. Lade das Skript herunter:
```powershell
# Navigiere zum gewünschten Verzeichnis
cd C:\Scripts\DHCPCheckup
```

2. Überprüfe die Execution Policy:
```powershell
Get-ExecutionPolicy
# Falls erforderlich, anpassen:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## Verwendung

### Grundlegende Verwendung

```powershell
# Skript importieren
. .\Invoke-DHCPCheckup.ps1

# Audit durchführen
Invoke-DHCPCheckup -domainName "beispiel.local" -dnsServerName "dc01.beispiel.local"
```

### Parameter

| Parameter | Typ | Erforderlich | Beschreibung |
|-----------|-----|--------------|--------------|
| `-domainName` | String | Ja | Name der Active Directory-Domäne, die gescannt werden soll |
| `-dnsServerName` | String | Ja | Name des DNS-Servers, der die AD-integrierte DNS-Zone hostet |

### Beispiel

```powershell
Invoke-DHCPCheckup -domainName "contoso.local" -dnsServerName "dc2022.contoso.local"
```

## Was wird überprüft?

### 1. Aktive DHCP-Server
- Ermittelt alle in der Domäne registrierten DHCP-Server
- Testet die Erreichbarkeit jedes Servers
- Listet alle aktiven Server auf

### 2. DNS-Anmeldeinformationen
- Prüft, welche Credentials die DHCP-Server zum Erstellen von DNS-Einträgen verwenden
- Warnt, wenn privilegierte Konten (Domain Admins, Enterprise Admins, etc.) verwendet werden
- Identifiziert Computerkonten, die als Fallback verwendet werden

### 3. DHCP-Namensschutz
- Überprüft Namensschutz-Einstellungen auf Server-Ebene (IPv4 und IPv6)
- Überprüft Namensschutz-Einstellungen für jeden einzelnen Scope
- Meldet deaktivierte Namensschutz-Konfigurationen

### 4. DNSUpdateProxy-Gruppe
- Listet alle Mitglieder der DNSUpdateProxy-Gruppe auf
- Identifiziert DHCP-Server in dieser Gruppe (Sicherheitsrisiko)
- Meldet Nicht-DHCP-Mitglieder, die entfernt werden sollten

### 5. Anfällige DNS-Einträge
- Scannt alle DNS-Einträge in der Zone
- Findet Einträge, die von DHCP-Servern erstellt wurden (anfällig für Client-Spoofing)
- Identifiziert Einträge mit Schreibrechten für "Authenticated Users"

## Ausgabe

Das Tool gibt detaillierte Informationen aus:
- ✅ **Gefundene aktive DHCP-Server**
- ⚠️  **Sicherheitswarnungen** bei problematischen Konfigurationen
- 🔍 **Liste anfälliger DNS-Einträge**

## Sicherheitshinweise

- Dieses Tool ist ausschließlich für **autorisierte Sicherheitstests** gedacht
- Das Skript führt **keine Änderungen** am System durch (read-only)
- Verwende es nur in Umgebungen, für die du die entsprechende Berechtigung hast
- Die Ergebnisse sollten zur Behebung von Sicherheitslücken verwendet werden

## Fehlerbehebung

### Fehler: "No DHCP servers found in the domain"
- Überprüfe, ob DHCP-Server in der Domäne autorisiert sind
- Stelle sicher, dass du über ausreichende Berechtigungen verfügst

### Fehler: "DHCP server X is not responding"
- Überprüfe die Netzwerkverbindung zum Server
- Stelle sicher, dass der DHCP-Dienst auf dem Server läuft
- Überprüfe Firewall-Regeln

### Module können nicht geladen werden
```powershell
# RSAT-Tools installieren (Windows 10/11)
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
Add-WindowsCapability -Online -Name Rsat.Dhcp.Tools~~~~0.0.1.0
Add-WindowsCapability -Online -Name Rsat.Dns.Tools~~~~0.0.1.0
```

## Lizenz

Dieses Projekt ist ein Fork von [akamai/Invoke-DHCPCheckup](https://github.com/akamai/Invoke-DHCPCheckup) (über [ForumSchlampe/Invoke-DHCPCheckup](https://github.com/ForumSchlampe/Invoke-DHCPCheckup)) und steht unter der Apache License 2.0.

```
Copyright 2023 Akamai Technologies Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Support

Bei Fragen oder Problemen:
- Öffne ein Issue in [feiglein74/Invoke-DHCPCheckup](https://github.com/feiglein74/Invoke-DHCPCheckup/issues)

## Changelog

Siehe [CHANGELOG.md](CHANGELOG.md) für eine vollständige Versionshistorie.
