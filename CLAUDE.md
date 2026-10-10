# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Übersicht

Dies ist ein PowerShell-Sicherheitsaudit-Tool namens **Invoke-DHCPCheckup**, das Sicherheitsüberprüfungen auf Microsoft DHCP-Servern in Active Directory-Umgebungen durchführt. Es identifiziert häufige Fehlkonfigurationen, die zu DNS-Spoofing-Schwachstellen führen können.

**Autor:** Ori David (@oridavid123) von Akamai SIG
**Referenz:** https://akamai.com/blog/security-research/spoofing-dns-by-abusing-dhcp

## Skript ausführen

Der Haupteinstiegspunkt ist die Funktion `Invoke-DHCPCheckup` in `Invoke-DHCPCheckup.ps1`:

```powershell
# Skript importieren und ausführen
. .\Invoke-DHCPCheckup.ps1
Invoke-DHCPCheckup -domainName "beispiel.local" -dnsServerName "dc.beispiel.local"
```

**Erforderliche Parameter:**
- `-domainName`: Der Active Directory-Domänenname zum Scannen
- `-dnsServerName`: Der DNS-Server, der die AD-integrierte DNS-Zone hostet

**Voraussetzungen:**
- Muss auf einem Windows-System mit PowerShell ausgeführt werden
- Benötigt folgende PowerShell-Module (werden beim Skriptstart importiert):
  - `DHCPServer`
  - `ActiveDirectory`
  - `DnsServer`
- Benutzer muss entsprechende Berechtigungen haben, um DHCP-Server und Active Directory abzufragen

## Architektur

Das Skript ist als einzelne PowerShell-Datei mit sechs Hauptfunktionen strukturiert:

### Sicherheitsüberprüfungs-Funktionen

1. **GetActiveActiveDhcpServers** (Zeilen 5-41)
   - Ermittelt alle DHCP-Server in der Domäne mittels `Get-DhcpServerInDC`
   - Testet jeden Server auf Erreichbarkeit
   - Gibt Liste der aktiven Server-DNS-Namen zurück

2. **GetStrongUsers** (Zeilen 43-57)
   - Identifiziert Mitglieder privilegierter AD-Gruppen: Domain Controllers, Domain Admins, Enterprise Admins, DnsAdmins, Administrators
   - Wird verwendet, um zu erkennen, ob DHCP-Credentials erhöhte Berechtigungen haben

3. **Check-DnsCredentialSettings** (Zeilen 59-109)
   - Untersucht die DNS-Credential-Konfiguration auf jedem DHCP-Server
   - Prüft, ob Credentials aus privilegierten Gruppen stammen (Sicherheitsrisiko)
   - Gibt Array von Credentials und deren zugehörigen Servern zurück

4. **Check-DhcpNameProtectionSettings** (Zeilen 111-172)
   - Prüft DHCP-Namensschutz-Einstellungen auf Server- und Scope-Ebene
   - Überprüft sowohl IPv4- als auch IPv6-Scopes
   - Namensschutz verhindert DNS-Eintrag-Spoofing

5. **Check-DnsUpdateProxyMembership** (Zeilen 174-194)
   - Überprüft DNSUpdateProxy-Gruppenmitgliedschaft
   - Identifiziert DHCP-Server in dieser Gruppe (anfällige Konfiguration)
   - Kennzeichnet Nicht-DHCP-Mitglieder, die entfernt werden sollten

6. **Find-VulnerableDnsRecords** (Zeilen 196-265)
   - Scannt DNS-Zonen-Einträge auf schwache ACLs
   - Identifiziert Einträge, die von DHCP-Servern erstellt wurden (anfällig für Client-Spoofing)
   - Findet Einträge, die von "Authenticated Users" beschreibbar sind

### Hauptfunktion

**Invoke-DHCPCheckup** (Zeilen 267-371) orchestriert die Sicherheitsüberprüfung durch:
1. Finden aktiver DHCP-Server
2. Prüfen der DNS-Credential-Konfigurationen
3. Audit der Namensschutz-Einstellungen
4. Überprüfung der DNSUpdateProxy-Mitgliedschaft
5. Scannen nach anfälligen DNS-Einträgen

## Sicherheitskontext

Dieses Tool ist für **autorisierte Sicherheitstests und defensive Sicherheit** in Unternehmensumgebungen konzipiert. Es identifiziert DHCP/DNS-Fehlkonfigurationen, die Angreifer ausnutzen könnten, um:
- DNS-Einträge zu fälschen
- Netzwerkverkehr umzuleiten
- Man-in-the-Middle-Angriffe durchzuführen

Das Skript ist read-only und führt keine Änderungen an der Umgebung durch.

## Code-Qualität & Bekannte Probleme

### Kritische Code-Issues (Stand: 2025-11-15)

**1. Globale Variablen / Scope-Bleeding**
- **Betroffene Funktionen:** `Check-DnsUpdateProxyMembership`, `Find-VulnerableDnsRecords`
- **Problem:** Nutzen `$domainName` und `$dnsServerName` ohne Parameter-Übergabe
- **Auswirkung:** Funktionen sind nicht standalone-fähig, nur aus Hauptfunktion aufrufbar
- **Lösung:** Parameter explizit übergeben

```powershell
# Aktuell (fehlerhaft):
function Check-DnsUpdateProxyMembership {
    $updateProxymembers = Get-ADGroupMember "DNSUpdateProxy"
    # $domainName wird aus Parent-Scope verwendet
}

# Korrekt:
function Check-DnsUpdateProxyMembership {
    param([string]$domainName)
    $updateProxymembers = Get-ADGroupMember "DNSUpdateProxy"
}
```

**2. Unsichere Fehlerbehandlung**
- **Betroffene Funktion:** `GetActiveActiveDhcpServers` (Zeilen 20-30)
- **Problem:** Manipulation des globalen `$error`-Arrays, leerer Catch-Block
- **Auswirkung:** Fehler können unerkannt bleiben oder falsch interpretiert werden

```powershell
# Aktuell (fehlerhaft):
$error.Clear()
try { $res = Get-DhcpServerSetting ... } catch { }
if (!$error[0]) { ... }

# Korrekt:
try {
    $res = Get-DhcpServerSetting -ComputerName $server.DnsName -ErrorAction Stop
    $success = $true
} catch {
    Write-Verbose "Server $($server.DnsName) nicht erreichbar: $_"
    $success = $false
}
if ($success) { ... }
```

**3. Array-Splitting ohne Validierung**
- **Betroffene Funktion:** `Find-VulnerableDnsRecords` (Zeilen 232-233)
- **Problem:** String-Split kann bei unerwarteten Formaten fehlschlagen
- **Auswirkung:** Runtime-Fehler bei Index-Out-Of-Bounds

```powershell
# Aktuell (fehlerhaft):
$DhcpServerCredential = $cred.split(' ')[0]
$DhcpServerName = $cred.split(' ')[1]

# Korrekt:
if ($cred -is [array] -and $cred.Count -ge 2) {
    $DhcpServerCredential = $cred[0]
    $DhcpServerName = $cred[1]
} else {
    Write-Warning "Ungültiges Credential-Format: $cred"
    continue
}
```

### Performance-Optimierungen

**4. GetStrongUsers - Ineffiziente AD-Abfragen**
- **Problem:** `Get-ADGroupMember` in Schleife, rekursive Gruppen werden mehrfach abgefragt
- **Auswirkung:** Bei großen AD-Umgebungen (>10.000 User) sehr langsam (30+ Sekunden)

```powershell
# Aktuell (langsam):
foreach ($group in $strongGroups) {
    $strongGroupsMembers += Get-ADGroupMember $group | select name
}

# Optimiert:
$strongGroupsMembers = $strongGroups | ForEach-Object {
    Get-ADGroupMember $_ -Recursive -ErrorAction SilentlyContinue
} | Select-Object -Unique -ExpandProperty Name
```

**5. Find-VulnerableDnsRecords - Nested Loop O(n*m)**
- **Problem:** Für jeden DNS-Eintrag wird die komplette DHCP-Credentials-Liste durchlaufen
- **Auswirkung:** Bei 1000 DNS-Einträgen und 10 DHCP-Servern = 10.000 Iterationen

```powershell
# Aktuell (langsam):
foreach ($record in $DnsRecords) {
    foreach ($cred in $DhcpCredentials) {
        if ($recordAcl.Owner -eq $cred) { ... }
    }
}

# Optimiert:
$credHash = @{}
foreach ($cred in $DhcpCredentials) {
    $credHash[$cred[0]] = $cred[1]
}
foreach ($record in $DnsRecords) {
    if ($credHash.ContainsKey($recordAcl.Owner)) { ... }
}
```

### Best Practices

**6. Funktionsnamen-Konventionen**
- PowerShell verwendet Verb-Noun-Pattern mit approved Verbs
- Aktuelle Namen: `GetActiveActiveDhcpServers`, `GetStrongUsers` (nicht standard-konform)
- Empfohlen: `Get-ActiveDhcpServer`, `Get-PrivilegedGroupMember`

**7. Pipeline-fähige Ausgabe**
- Aktuell nur `Write-Host` (nicht weiterverarbeitbar)
- Besser: PSCustomObject zurückgeben für Export/Filterung

```powershell
# Statt nur Write-Host:
return [PSCustomObject]@{
    ServerName = $server
    Issue = "NameProtectionDisabled"
    Severity = "High"
    Scope = $scopeID.Name
}
```

### Code-Verbesserungs-Roadmap

**Priorität 1 (Kritisch):**
1. Parameter-Übergabe statt globale Variablen
2. Fehlerbehandlung mit sauberem Try-Catch
3. Array-Zugriff absichern

**Priorität 2 (Performance):**
4. AD-Gruppen-Abfragen optimieren
5. DNS-Scanning mit HashTable

**Priorität 3 (Wartbarkeit):**
6. Funktionsnamen standardisieren
7. Strukturierte Ausgabe implementieren
8. Code-Duplikation reduzieren (IPv4/IPv6)

Detaillierte Umsetzung siehe TODO.md unter "Code-Review-Ergebnisse".

## Git-Konventionen

**WICHTIG:** Alle Git-Commits und Kommentare müssen auf Deutsch verfasst werden und sollten die informelle "du"-Form verwenden (kein "Sie").

### Commit-Message-Format

```
<Typ>: <Kurzbeschreibung>

<Optionaler längerer Beschreibungstext>
```

**Commit-Typen:**
- `feat`: Neue Funktion hinzugefügt
- `fix`: Bugfix
- `docs`: Dokumentationsänderungen
- `refactor`: Code-Refactoring ohne Funktionsänderung
- `test`: Tests hinzugefügt oder geändert
- `chore`: Wartungsarbeiten, Build-Prozess

**Beispiele:**
```
feat: Export-Funktion für CSV-Reports hinzugefügt

fix: Fehlerbehandlung bei nicht erreichbaren DHCP-Servern verbessert

docs: README mit zusätzlichen Beispielen erweitert
```
