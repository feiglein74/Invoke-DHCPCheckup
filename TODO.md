# TODO - Invoke-DHCPCheckup

## Aktuell in Arbeit

_Keine offenen Tasks_

## Code-Review-Ergebnisse (2025-11-15, aktualisiert 2026-10-10 nach Umbau auf ForumSchlampes Fork)

### 🔴 Priorität 1 - Kritische Probleme (sofort beheben)

- [x] **Globale Variablen beheben** (2026-10-10)
- [x] **Fehlerbehandlung in `GetActiveActiveDhcpServers` verbessern** (2026-10-10)
- [x] **Credential-Paare als PSCustomObjects statt String-Split** (2026-10-10, behebt auch den Fall mit nur einem DHCP-Server)
- [x] **Owner-Split unter PowerShell 7 repariert** (2026-10-10, `.Split('\\')` trennte dort am Doppel-Backslash)

- [ ] **Credential-Abgleich normalisieren** (neu)
  - `Check-DnsCredentialSettings` (Zeile 82): vergleicht `sAMAccountName` mit dem CN aus `GetStrongUsers`
  - Problem: Privilegierte Benutzer-Credentials werden nicht erkannt, wenn CN ≠ sAMAccountName (vgl. Akamai-Issue #11)
  - Lösung: `GetStrongUsers` gibt `SamAccountName` zurück; in Testumgebung verifizieren

### 🟡 Priorität 2 - Performance-Optimierungen

- [x] **GetStrongUsers: `-Recursive`** (durch ForumSchlampes Fork erledigt)
- [ ] **GetStrongUsers: `+=` in Schleife durch Pipeline ersetzen** (Zeilen 47-50, geringe Priorität)

- [ ] **DNS-Record-Scanning optimieren** (Zeilen 280-318)
  - DHCP-Credentials in HashTable cachen für O(1) Lookup
  - Nested Loop O(n*m) vermeiden
  - Hinweis: Hauptkosten verursacht `get-acl` pro Record

### 🟢 Priorität 3 - Best Practices & Wartbarkeit

- [ ] **Funktionsnamen standardisieren**
  - `GetActiveActiveDhcpServers` → `Get-ActiveDhcpServer` (PowerShell Verb-Noun Convention)
  - `GetStrongUsers` → `Get-PrivilegedGroupMember`
  - Einheitliche Namenskonvention für alle Funktionen

- [ ] **Code-Duplikation reduzieren**
  - IPv4/IPv6-Prüfungen in `Check-DhcpNameProtectionSettings` (Zeilen 110-171) und `Check-DhcpDynamicNameUpdateSettings` (Zeilen 173-234) in Hilfsfunktionen auslagern


- [ ] **Strukturierte Ausgabe implementieren**
  - PSCustomObject statt nur Write-Host zurückgeben
  - Pipeline-fähige Funktionen erstellen
  - Ermöglicht: Export, Filterung, Weiterverarbeitung

- [ ] **Parameter-Validierung erweitern**
  - `[ValidateNotNullOrEmpty()]` für Pflichtparameter
  - Domain-/DNS-Server-Existenz validieren
  - Netzwerk-Erreichbarkeit prüfen vor Scan-Start

### 📝 Priorität 4 - Dokumentation

- [ ] **Inline-Kommentare ergänzen**
  - Komplexe Logik in `Find-VulnerableDnsRecords` dokumentieren
  - ACL-Prüfungen erklären (Zeilen 284-317)
  - Sicherheitsrisiken in Kommentaren beschreiben

- [ ] **Comment-Based Help verbessern**
  - Für alle Helper-Funktionen `.SYNOPSIS` hinzufügen
  - Beispiele für jede Funktion ergänzen

## Geplante Verbesserungen

### Version 1.1.0
- [ ] Export-Funktion für Audit-Ergebnisse (CSV, JSON, HTML-Report)
- [ ] Parameter `-Verbose` für detaillierte Ausgaben hinzufügen
- [ ] Parameter `-Quiet` für minimale Ausgaben hinzufügen
- [ ] Logging-Funktionalität mit Timestamp in Log-Datei

### Version 1.2.0
- [ ] Remediation-Vorschläge für gefundene Probleme
- [ ] Automatische Fix-Option für unkritische Probleme (mit Bestätigung)
- [ ] Überprüfung von DHCP-Failover-Konfigurationen
- [ ] Prüfung der DHCP-Option 81 (Client FQDN) Einstellungen

### Version 2.0.0
- [ ] Multi-Domain-Support (Forest-weite Scans)
- [ ] REST-API für Integration in Monitoring-Systeme
- [ ] PowerShell-Modul erstellen und im PowerShell Gallery veröffentlichen
- [ ] GUI-Version mit Windows Forms oder WPF

## Dokumentation

- [ ] Video-Tutorial erstellen
- [ ] Best-Practice-Guide für DHCP-Sicherheit schreiben
- [ ] FAQ-Sektion erweitern
- [ ] Beispiel-Ausgaben zu README hinzufügen

## Testing

- [ ] Pester-Tests schreiben für alle Funktionen
- [ ] Test-Umgebung mit verschiedenen DHCP-Konfigurationen aufbauen
- [ ] Performance-Tests bei großen Umgebungen (>100 DHCP-Server)
- [ ] Kompatibilitätstests mit verschiedenen Windows Server-Versionen

## Code-Qualität

- [ ] Code-Kommentare auf Deutsch übersetzen
- [ ] PSScriptAnalyzer-Warnungen beheben
- [ ] Fehlerbehandlung verbessern (Try-Catch-Blöcke)
- [ ] Parameter-Validierung erweitern

## Sicherheit

- [ ] Credential-Management mit SecureString verbessern
- [ ] Audit-Trail für alle durchgeführten Checks
- [ ] Read-Only-Modus explizit verifizieren und dokumentieren
- [ ] Signierung des PowerShell-Skripts

## Bekannte Probleme

- Dynamic-Update-Check meldet jeden Wert außer `Never`, also praktisch jeden Server mit Windows-Standard (`OnClientRequest`). Bewertung/Severity überdenken (vgl. Akamai-Issue #15)

## Ideen für die Zukunft

- Integration mit Microsoft Defender for Identity
- Dashboard für kontinuierliches Monitoring
- Automatische Benachrichtigungen bei neuen Schwachstellen
- Integration mit SIEM-Systemen (Splunk, QRadar, etc.)
- Compliance-Reports (CIS Benchmarks, NIST, etc.)

## Abgeschlossene Aufgaben

- [x] Initiales Skript erstellen
- [x] README.md erstellen
- [x] CHANGELOG.md erstellen
- [x] Git Repository einrichten
- [x] CLAUDE.md für AI-Assistenz erstellen
- [x] Fork auf GitHub angelegt, `main` auf ForumSchlampes Fork umgebaut (SID-basierte Gruppenerkennung, sprachunabhängig, Dynamic-Update-Check)

---

**Hinweis:** Diese TODO-Liste wird kontinuierlich gepflegt. Vorschläge und Beiträge sind willkommen!

**Letzte Aktualisierung:** 2026-10-10
