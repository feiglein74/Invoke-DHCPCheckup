# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Übersicht

Dies ist ein PowerShell-Sicherheitsaudit-Tool namens **Invoke-DHCPCheckup**, das Sicherheitsüberprüfungen auf Microsoft DHCP-Servern in Active Directory-Umgebungen durchführt. Es identifiziert häufige Fehlkonfigurationen, die zu DNS-Spoofing-Schwachstellen führen können.

**Original-Autor:** Ori David (@oridavid123) von Akamai SIG
**Referenz:** https://akamai.com/blog/security-research/spoofing-dns-by-abusing-dhcp

## Herkunft & Remotes

Dieses Repo ist ein Fork. Das Akamai-Original wird seit Dezember 2023 nicht mehr gepflegt (offene PRs ohne Reaktion), deshalb entwickeln wir hier unabhängig weiter.

| Remote | Repository | Rolle |
|---|---|---|
| `origin` | `feiglein74/Invoke-DHCPCheckup` | Unser Fork, hierhin wird gepusht |
| `upstream` | `akamai/Invoke-DHCPCheckup` | Original, inaktiv |
| `forumschlampe` | `ForumSchlampe/Invoke-DHCPCheckup` | Basis unseres `main` (SID-basierte Gruppenerkennung, Dynamic-Update-Check, SID-Fix von michaelbast) |

- `main` baut auf `forumschlampe/main` (Stand 2025-01-29) auf.
- PRs an Akamai lohnen sich voraussichtlich nicht. Falls doch: Branch von `upstream/main` abzweigen, nur Code-Änderungen, englisch, ohne deutsche Doku und CLAUDE.md.
- Der alte lokale Stand vor dem Umbau liegt im lokalen Branch `backup/initial-setup`.

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

Das Skript ist als einzelne PowerShell-Datei mit sieben Prüf-/Hilfsfunktionen und einer Hauptfunktion strukturiert:

### Sicherheitsüberprüfungs-Funktionen

1. **GetActiveActiveDhcpServers** (Zeilen 5-33)
   - Ermittelt alle DHCP-Server in der Domäne mittels `Get-DhcpServerInDC`
   - Testet jeden Server auf Erreichbarkeit
   - Gibt Liste der aktiven Server-DNS-Namen zurück

2. **GetStrongUsers** (Zeilen 35-56)
   - Identifiziert Mitglieder privilegierter AD-Gruppen (rekursiv) über **SIDs**, damit sprachunabhängig:
     Domain Controllers (`<DomainSID>-516`), Domain Admins (`-512`), Enterprise Admins (`-519`), Administrators (`S-1-5-32-544`)
   - DnsAdmins wird per Name gesucht (keine feste SID, Name wird nicht lokalisiert)
   - Gibt das `name`-Attribut (CN) der Mitglieder zurück

3. **Check-DnsCredentialSettings** (Zeilen 58-108)
   - Untersucht die DNS-Credential-Konfiguration auf jedem DHCP-Server
   - Prüft, ob Credentials aus privilegierten Gruppen stammen (Sicherheitsrisiko)
   - Gibt PSCustomObjects mit `Credential` und `Server` zurück

4. **Check-DhcpNameProtectionSettings** (Zeilen 110-171)
   - Prüft DHCP-Namensschutz-Einstellungen auf Server- und Scope-Ebene
   - Überprüft sowohl IPv4- als auch IPv6-Scopes
   - Namensschutz verhindert DNS-Eintrag-Spoofing

5. **Check-DhcpDynamicNameUpdateSettings** (Zeilen 173-234) — aus dem Fork von diecknet/ForumSchlampe
   - Prüft die DNS-Dynamic-Update-Einstellung auf Server- und Scope-Ebene (IPv4 und IPv6)
   - Meldet jeden Wert außer `Never`; da `OnClientRequest` der Windows-Standard ist, schlägt der Check praktisch immer an (eher Hinweis als Befund)

6. **Check-DnsUpdateProxyMembership** (Zeilen 236-260)
   - Überprüft DNSUpdateProxy-Gruppenmitgliedschaft (rekursiv)
   - Identifiziert DHCP-Server in dieser Gruppe (anfällige Konfiguration)
   - Kennzeichnet Nicht-DHCP-Mitglieder, die entfernt werden sollten

7. **Find-VulnerableDnsRecords** (Zeilen 262-334)
   - Scannt DNS-Zonen-Einträge auf schwache ACLs
   - Identifiziert Einträge, die von DHCP-Servern erstellt wurden (anfällig für Client-Spoofing)
   - Findet Einträge, die von "Authenticated Users" beschreibbar sind (Auflösung über SID `S-1-5-11`, damit sprachunabhängig)

### Hauptfunktion

**Invoke-DHCPCheckup** (Zeilen 336-445) orchestriert die Sicherheitsüberprüfung durch:
1. Finden aktiver DHCP-Server
2. Prüfen der DNS-Credential-Konfigurationen
3. Prüfen der Dynamic-Update-Einstellungen
4. Audit der Namensschutz-Einstellungen
5. Überprüfung der DNSUpdateProxy-Mitgliedschaft
6. Scannen nach anfälligen DNS-Einträgen

## Sicherheitskontext

Dieses Tool ist für **autorisierte Sicherheitstests und defensive Sicherheit** in Unternehmensumgebungen konzipiert. Es identifiziert DHCP/DNS-Fehlkonfigurationen, die Angreifer ausnutzen könnten, um:
- DNS-Einträge zu fälschen
- Netzwerkverkehr umzuleiten
- Man-in-the-Middle-Angriffe durchzuführen

Das Skript ist read-only und führt keine Änderungen an der Umgebung durch.

## Code-Qualität & Bekannte Probleme

### Kritische Code-Issues (Stand: 2026-10-10)

Erledigt am 2026-10-10: Parameter statt globaler Variablen, `try/catch` mit `-ErrorAction Stop` statt `$error`-Prüfung, Credential-Paare als PSCustomObjects statt String-Split, Owner-Split funktioniert jetzt auch unter PowerShell 7.

**1. Credential-Abgleich mit unterschiedlichen Namensformaten**
- **Betroffene Funktion:** `Check-DnsCredentialSettings` (Zeile 82)
- **Problem:** `Get-DhcpServerDnsCredential` liefert den Anmeldenamen (`sAMAccountName`), `GetStrongUsers` liefert aber das `name`-Attribut (CN). Bei Benutzerkonten weichen beide oft ab (z. B. CN „Max Mustermann“, sAMAccountName „mmustermann“)
- **Auswirkung:** Ein privilegiertes DNS-Credential wird nicht als solches gemeldet (vgl. Akamai-Issue #11). Computerkonten (Zeile 96) sind meist nicht betroffen, da CN und Name ohne `$` übereinstimmen
- **Lösung:** `GetStrongUsers` sollte `SamAccountName` (oder SIDs) zurückgeben und beide Vergleiche darauf umstellen; vor dem Fix in einer Testumgebung verifizieren

### Performance-Optimierungen

**5. GetStrongUsers - AD-Abfragen**
- `-Recursive` ist inzwischen umgesetzt; die Abfrage läuft aber noch per `+=` in einer Schleife
- Geringe Priorität, nur bei sehr großen Gruppen relevant

**6. Find-VulnerableDnsRecords - Nested Loop O(n*m)**
- **Problem:** Für jeden DNS-Eintrag wird die komplette DHCP-Credentials-Liste durchlaufen
- **Auswirkung:** Bei 1000 DNS-Einträgen und 10 DHCP-Servern = 10.000 Iterationen; der eigentliche Engpass ist aber `get-acl` pro Record

```powershell
# Optimiert:
$credHash = @{}
foreach ($cred in $DhcpCredentials) {
    $credHash[$cred.Credential] = $cred.Server
}
foreach ($record in $DnsRecords) {
    if ($credHash.ContainsKey($ownerName)) { ... }
}
```

### Best Practices

**7. Funktionsnamen-Konventionen**
- PowerShell verwendet Verb-Noun-Pattern mit approved Verbs
- Aktuelle Namen: `GetActiveActiveDhcpServers`, `GetStrongUsers`, `Check-*` (nicht standard-konform)
- Empfohlen: `Get-ActiveDhcpServer`, `Get-PrivilegedGroupMember`, `Test-*`

**8. Pipeline-fähige Ausgabe**
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

**9. Code-Duplikation**
- IPv4/IPv6-Logik ist in `Check-DhcpNameProtectionSettings` und `Check-DhcpDynamicNameUpdateSettings` jeweils dupliziert

### Code-Verbesserungs-Roadmap

**Priorität 1 (Kritisch):**
1. Credential-Abgleich normalisieren (sAMAccountName vs. CN)

**Priorität 2 (Performance):**
5. DNS-Scanning mit HashTable

**Priorität 3 (Wartbarkeit):**
6. Funktionsnamen standardisieren
7. Strukturierte Ausgabe implementieren
8. Code-Duplikation reduzieren (IPv4/IPv6)

Detaillierte Umsetzung siehe TODO.md unter "Code-Review-Ergebnisse".

## Git-Konventionen

**WICHTIG:** Alle Git-Commits und Kommentare müssen auf Deutsch verfasst werden und sollten die informelle "du"-Form verwenden (kein "Sie"). Ausnahme: PRs an fremde Repos (Akamai, ForumSchlampe) auf Englisch.

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
