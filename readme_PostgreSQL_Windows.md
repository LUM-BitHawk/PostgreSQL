# PostgreSQL Installation auf Windows Server 2022 via PowerShell

> **Voraussetzungen:** Windows Server 2022, PowerShell 5.1+ (als Administrator), Internetzugang

---

## Inhaltsverzeichnis

1. [PowerShell als Administrator starten](#1-powershell-als-administrator-starten)
2. [Ausführungsrichtlinie prüfen](#2-ausführungsrichtlinie-prüfen)
3. [PostgreSQL Installer herunterladen](#3-postgresql-installer-herunterladen)
4. [Stille Installation durchführen](#4-stille-installation-durchführen)
5. [Umgebungsvariablen setzen](#5-umgebungsvariablen-setzen)
6. [PostgreSQL-Dienst konfigurieren](#6-postgresql-dienst-konfigurieren)
7. [Firewall-Regel erstellen](#7-firewall-regel-erstellen)
8. [Verbindung testen](#8-verbindung-testen)
9. [Ersten Datenbankbenutzer anlegen](#9-ersten-datenbankbenutzer-anlegen)
10. [Installation verifizieren](#10-installation-verifizieren)
11. [Deinstallation (optional)](#11-deinstallation-optional)
12. [Fehlerbehebung](#12-fehlerbehebung)

---

## 1. PowerShell als Administrator starten

```powershell
# Rechtsklick auf das Start-Menü → "Windows PowerShell (Administrator)"
# Oder via Ausführen (Win + R):
Start-Process powershell -Verb RunAs
```

Aktuellen Benutzer prüfen:

```powershell
whoami
# Erwartete Ausgabe: <DOMAIN>\Administrator  oder  SYSTEM
```

---

## 2. Ausführungsrichtlinie prüfen

```powershell
# Aktuelle Richtlinie anzeigen
Get-ExecutionPolicy

# Für die Installation temporär anpassen (falls nötig)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

---

## 3. PostgreSQL Installer herunterladen

```powershell
# Zielordner anlegen
$DownloadPath = "C:\Temp\PostgreSQL"
New-Item -ItemType Directory -Path $DownloadPath -Force

# Aktuelle Version und Download-URL festlegen
# Passe die Versionsnummer bei Bedarf an: https://www.postgresql.org/download/windows/
$PGVersion  = "16.3"
$InstallerURL = "https://get.enterprisedb.com/postgresql/postgresql-$PGVersion-1-windows-x64.exe"
$InstallerPath = "$DownloadPath\postgresql-$PGVersion-windows-x64.exe"

# Herunterladen
Write-Host "Lade PostgreSQL $PGVersion herunter..." -ForegroundColor Cyan
Invoke-WebRequest -Uri $InstallerURL -OutFile $InstallerPath -UseBasicParsing

# Datei prüfen
if (Test-Path $InstallerPath) {
    Write-Host "Download erfolgreich: $InstallerPath" -ForegroundColor Green
} else {
    Write-Error "Download fehlgeschlagen!"
}
```

---

## 4. Stille Installation durchführen

> **Hinweis:** Das `superpassword` unbedingt durch ein sicheres Passwort ersetzen!

```powershell
# Installationsparameter definieren
$InstallDir   = "C:\Program Files\PostgreSQL\16"
$DataDir      = "C:\PostgreSQL\data"
$ServiceName  = "postgresql-x64-16"
$SuperPwd     = "DeinSicheresPasswort123!"   # <-- HIER ANPASSEN
$Port         = 5432
$Locale       = "German, Germany"

# Stille Installation starten
$Arguments = @(
    "--unattendedmodeui", "none",
    "--mode", "unattended",
    "--superpassword", $SuperPwd,
    "--serverport", $Port,
    "--prefix", $InstallDir,
    "--datadir", $DataDir,
    "--locale", $Locale,
    "--servicename", $ServiceName
)

Write-Host "Starte PostgreSQL-Installation..." -ForegroundColor Cyan
Start-Process -FilePath $InstallerPath -ArgumentList $Arguments -Wait -NoNewWindow

Write-Host "Installation abgeschlossen." -ForegroundColor Green
```

### Installierte Komponenten (Standard)

| Komponente       | Beschreibung                        |
|------------------|-------------------------------------|
| PostgreSQL Server | Datenbankserver                    |
| pgAdmin 4        | Grafische Verwaltungsoberfläche     |
| Stack Builder    | Erweiterungen & Treiber             |
| Command Line Tools | psql, pg_dump, pg_restore etc.   |

---

## 5. Umgebungsvariablen setzen

```powershell
# PostgreSQL bin-Verzeichnis zum PATH hinzufügen
$PGBin = "C:\Program Files\PostgreSQL\16\bin"

[System.Environment]::SetEnvironmentVariable(
    "Path",
    "$env:Path;$PGBin",
    [System.EnvironmentVariableTarget]::Machine
)

# PGDATA setzen
[System.Environment]::SetEnvironmentVariable(
    "PGDATA",
    "C:\PostgreSQL\data",
    [System.EnvironmentVariableTarget]::Machine
)

# Neue Variablen in aktuelle Session laden
$env:Path    = [System.Environment]::GetEnvironmentVariable("Path","Machine")
$env:PGDATA  = [System.Environment]::GetEnvironmentVariable("PGDATA","Machine")

Write-Host "Umgebungsvariablen gesetzt." -ForegroundColor Green
```

---

## 6. PostgreSQL-Dienst konfigurieren

```powershell
$ServiceName = "postgresql-x64-16"

# Dienststatus prüfen
Get-Service -Name $ServiceName

# Dienst starten (falls nicht aktiv)
Start-Service -Name $ServiceName

# Autostart sicherstellen
Set-Service -Name $ServiceName -StartupType Automatic

# Status bestätigen
Get-Service -Name $ServiceName | Select-Object Name, Status, StartType
```

**Erwartete Ausgabe:**

```
Name                   Status  StartType
----                   ------  ---------
postgresql-x64-16     Running  Automatic
```

---

## 7. Firewall-Regel erstellen

```powershell
# Eingehende Verbindungen auf Port 5432 erlauben
New-NetFirewallRule `
    -DisplayName "PostgreSQL Port 5432" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 5432 `
    -Action Allow `
    -Profile Any `
    -Description "Erlaubt eingehende PostgreSQL-Verbindungen"

# Regel prüfen
Get-NetFirewallRule -DisplayName "PostgreSQL Port 5432" |
    Select-Object DisplayName, Enabled, Direction, Action
```

---

## 8. Verbindung testen

```powershell
# Als postgres-Superuser verbinden und Version anzeigen
$env:PGPASSWORD = "DeinSicheresPasswort123!"   # <-- Dein Passwort

psql -U postgres -c "SELECT version();"
```

**Erwartete Ausgabe (Beispiel):**

```
PostgreSQL 16.3, compiled by Visual C++ build 1939, 64-bit
```

---

## 9. Ersten Datenbankbenutzer anlegen

```powershell
# Neue Datenbank und Benutzer erstellen
$NewUser   = "appuser"
$NewPwd    = "AppUserPasswort456!"
$NewDB     = "appdb"

$env:PGPASSWORD = "DeinSicheresPasswort123!"

# Benutzer anlegen
psql -U postgres -c "CREATE USER $NewUser WITH PASSWORD '$NewPwd';"

# Datenbank anlegen und Rechte vergeben
psql -U postgres -c "CREATE DATABASE $NewDB OWNER $NewUser;"
psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE $NewDB TO $NewUser;"

Write-Host "Benutzer '$NewUser' und Datenbank '$NewDB' erstellt." -ForegroundColor Green
```

---

## 10. Installation verifizieren

```powershell
# PostgreSQL-Version
psql -U postgres -c "SELECT version();"

# Alle Datenbanken auflisten
psql -U postgres -l

# Alle Benutzer auflisten
psql -U postgres -c "\du"

# Dienststatus
Get-Service -Name "postgresql-x64-16"

# Lauschender Port prüfen
netstat -ano | findstr ":5432"
```

---

## 11. Deinstallation (optional)

```powershell
# Dienst stoppen
Stop-Service -Name "postgresql-x64-16" -Force

# Uninstaller aufrufen (Pfad ggf. anpassen)
$Uninstaller = "C:\Program Files\PostgreSQL\16\uninstall-postgresql.exe"

Start-Process -FilePath $Uninstaller `
    -ArgumentList "--mode unattended" `
    -Wait -NoNewWindow

# Datenverzeichnis manuell entfernen (Achtung: löscht alle Daten!)
# Remove-Item -Recurse -Force "C:\PostgreSQL"

# Firewall-Regel entfernen
Remove-NetFirewallRule -DisplayName "PostgreSQL Port 5432"

Write-Host "Deinstallation abgeschlossen." -ForegroundColor Yellow
```

---

## 12. Fehlerbehebung

### Dienst startet nicht

```powershell
# Windows-Ereignisprotokoll prüfen
Get-EventLog -LogName Application -Source "*postgresql*" -Newest 20

# PostgreSQL-Logdatei prüfen
Get-Content "C:\PostgreSQL\data\log\postgresql-*.log" -Tail 50
```

### Port 5432 bereits belegt

```powershell
# Welcher Prozess nutzt Port 5432?
netstat -ano | findstr ":5432"

# PID ermitteln und Prozess anzeigen
Get-Process -Id <PID>
```

### Passwort-Authentifizierungsfehler

```powershell
# pg_hba.conf anpassen (Authentifizierungsmethode)
$HBAPath = "C:\PostgreSQL\data\pg_hba.conf"
notepad $HBAPath

# Nach Änderung: Dienst neu starten
Restart-Service -Name "postgresql-x64-16"
```

### psql nicht gefunden

```powershell
# PATH-Variable prüfen
$env:Path -split ";" | Where-Object { $_ -like "*PostgreSQL*" }

# Manuell hinzufügen
$env:Path += ";C:\Program Files\PostgreSQL\16\bin"
```

---

## Referenzen

- [PostgreSQL offizielle Dokumentation](https://www.postgresql.org/docs/16/)
- [EDB Windows Installer](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
- [pg_hba.conf Dokumentation](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)

---

*Erstellt für Windows Server 2022 · PostgreSQL 16 · PowerShell 5.1+*
