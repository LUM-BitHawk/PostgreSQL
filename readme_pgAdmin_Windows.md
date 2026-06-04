# pgAdmin Installation auf Windows Server 2022 via PowerShell

> **Zielgruppe:** Systemadministratoren  
> **Betriebssystem:** Windows Server 2022  
> **Installationsmethode:** PowerShell (als Administrator)  
> **Stand:** Juni 2026

---

## Voraussetzungen

Vor der Installation sicherstellen, dass folgendes vorhanden ist:

- Windows Server 2022 (Standard oder Datacenter)
- PowerShell 5.1 oder neuer (vorinstalliert)
- Internetverbindung oder Zugriff auf den Installationsordner
- Administratorrechte auf dem Server

---

## Schritt 1 – PowerShell als Administrator öffnen

1. Drücke `Win + X` → **Windows PowerShell (Administrator)** oder **Terminal (Administrator)**
2. Alternativ via Suche: `powershell` → Rechtsklick → **Als Administrator ausführen**
3. Bestätige die UAC-Abfrage mit **Ja**

Prüfen ob PowerShell korrekt läuft:

```powershell
$PSVersionTable.PSVersion
```

---

## Schritt 2 – Ausführungsrichtlinie prüfen und anpassen

```powershell
# Aktuelle Richtlinie anzeigen
Get-ExecutionPolicy

# Falls "Restricted" → temporär anpassen
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

> **Hinweis:** Nach der Installation kann die Richtlinie wieder zurückgesetzt werden.

---

## Schritt 3 – Chocolatey (Paketmanager) installieren (empfohlen)

Chocolatey ermöglicht die einfache Installation und spätere Aktualisierung von pgAdmin.

```powershell
# Prüfen ob Chocolatey bereits installiert ist
if (!(Get-Command choco -ErrorAction SilentlyContinue)) {
    Write-Host "Installiere Chocolatey..." -ForegroundColor Cyan
    Set-ExecutionPolicy Bypass -Scope Process -Force
    [System.Net.ServicePointManager]::SecurityProtocol = `
        [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
    Invoke-Expression ((New-Object System.Net.WebClient).DownloadString(
        'https://community.chocolatey.org/install.ps1'))
} else {
    Write-Host "Chocolatey ist bereits installiert." -ForegroundColor Green
}
```

Installation überprüfen:

```powershell
choco --version
```

---

## Schritt 4 – pgAdmin 4 installieren

### Option A: Installation via Chocolatey (empfohlen)

```powershell
choco install pgadmin4 -y
```

Chocolatey lädt automatisch die neueste stabile Version herunter und installiert sie.

### Option B: Manuelle Installation via PowerShell (ohne Chocolatey)

```powershell
# Neueste Versionsnummer anpassen (aktuell prüfen auf https://www.pgadmin.org/download/)
$pgAdminVersion = "8.14"
$installerUrl = "https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v$pgAdminVersion/windows/pgadmin4-$pgAdminVersion-x64.exe"
$installerPath = "$env:TEMP\pgadmin4-installer.exe"

# Installer herunterladen
Write-Host "Lade pgAdmin 4 v$pgAdminVersion herunter..." -ForegroundColor Cyan
Invoke-WebRequest -Uri $installerUrl -OutFile $installerPath -UseBasicParsing

# Installer prüfen
if (Test-Path $installerPath) {
    Write-Host "Download erfolgreich: $installerPath" -ForegroundColor Green
} else {
    Write-Error "Download fehlgeschlagen!"
    exit 1
}

# Silent-Installation starten
Write-Host "Starte Silent-Installation..." -ForegroundColor Cyan
Start-Process -FilePath $installerPath -ArgumentList "/SILENT /NORESTART" -Wait

Write-Host "Installation abgeschlossen." -ForegroundColor Green
```

---

## Schritt 5 – Installation überprüfen

```powershell
# Prüfen ob pgAdmin im Startmenü / Programmverzeichnis vorhanden ist
$pgAdminPath = "${env:ProgramFiles}\pgAdmin 4\runtime\pgAdmin4.exe"

if (Test-Path $pgAdminPath) {
    Write-Host "pgAdmin 4 erfolgreich installiert: $pgAdminPath" -ForegroundColor Green
} else {
    Write-Warning "pgAdmin 4 wurde nicht unter dem Standardpfad gefunden."
    # Alternativsuche
    Get-ChildItem "C:\Program Files" -Recurse -Filter "pgAdmin4.exe" -ErrorAction SilentlyContinue
}

# Installierte Programme via Registry prüfen
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Where-Object { $_.DisplayName -like "*pgAdmin*" } |
    Select-Object DisplayName, DisplayVersion, InstallDate
```

---

## Schritt 6 – pgAdmin 4 starten

### Manuell starten

```powershell
Start-Process "${env:ProgramFiles}\pgAdmin 4\runtime\pgAdmin4.exe"
```

### pgAdmin als Windows-Dienst konfigurieren (Server-Modus)

Für den Einsatz als Webserver auf dem Server (empfohlen für Remote-Zugriff):

```powershell
# pgAdmin im Server-Modus starten (öffnet Browser-Interface)
$serverPath = "${env:ProgramFiles}\pgAdmin 4\runtime\pgAdmin4.exe"
Start-Process $serverPath
```

pgAdmin startet dann als lokaler Webserver und öffnet sich im Browser unter:

```
http://127.0.0.1:PORT
```

> Der Port wird beim ersten Start automatisch gewählt und in der Konsole angezeigt.

---

## Schritt 7 – Windows Firewall anpassen (optional, für Remote-Zugriff)

Falls pgAdmin im Server-Modus von anderen Maschinen erreichbar sein soll:

```powershell
# Firewallregel für pgAdmin-Port freischalten (Beispiel: Port 5050)
$pgAdminPort = 5050

New-NetFirewallRule `
    -DisplayName "pgAdmin 4 Web Server" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort $pgAdminPort `
    -Action Allow `
    -Profile Domain,Private `
    -Description "Erlaubt eingehende Verbindungen zu pgAdmin 4"

Write-Host "Firewallregel für Port $pgAdminPort wurde erstellt." -ForegroundColor Green
```

Firewallregel überprüfen:

```powershell
Get-NetFirewallRule -DisplayName "pgAdmin 4 Web Server" | Select-Object DisplayName, Enabled, Action
```

---

## Schritt 8 – pgAdmin aktualisieren (Chocolatey)

```powershell
# pgAdmin auf die neueste Version aktualisieren
choco upgrade pgadmin4 -y

# Oder alle installierten Pakete aktualisieren
choco upgrade all -y
```

---

## Schritt 9 – pgAdmin deinstallieren (falls nötig)

### Via Chocolatey

```powershell
choco uninstall pgadmin4 -y
```

### Manuell via PowerShell

```powershell
$uninstallKey = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Where-Object { $_.DisplayName -like "*pgAdmin*" }

if ($uninstallKey) {
    Start-Process $uninstallKey.UninstallString -ArgumentList "/SILENT" -Wait
    Write-Host "pgAdmin wurde deinstalliert." -ForegroundColor Green
} else {
    Write-Warning "pgAdmin nicht in der Registry gefunden."
}
```

---

## Fehlerbehebung

| Problem | Ursache | Lösung |
|---|---|---|
| `Invoke-WebRequest` schlägt fehl | TLS-Version | `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12` |
| Port bereits belegt | Anderer Prozess läuft auf dem Port | `netstat -ano \| findstr :5050` – Prozess identifizieren und beenden |
| pgAdmin öffnet keinen Browser | Server-Modus nicht aktiv | pgAdmin manuell im Browser unter `http://127.0.0.1:PORT` aufrufen |
| Chocolatey nicht gefunden | PATH nicht aktualisiert | PowerShell-Fenster schließen und neu öffnen |
| Installation schlägt fehl | Fehlende Adminrechte | PowerShell explizit als Administrator starten |

---

## Nützliche PowerShell-Befehle auf einen Blick

```powershell
# pgAdmin-Version prüfen (Registry)
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Where-Object { $_.DisplayName -like "*pgAdmin*" } |
    Select-Object DisplayName, DisplayVersion

# Laufende pgAdmin-Prozesse anzeigen
Get-Process | Where-Object { $_.Name -like "*pgAdmin*" }

# pgAdmin-Prozess beenden
Stop-Process -Name "pgAdmin4" -Force

# Installationspfad ermitteln
(Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Where-Object { $_.DisplayName -like "*pgAdmin*" }).InstallLocation
```

---

## Weiterführende Links

- [pgAdmin Offizielle Website](https://www.pgadmin.org/)
- [pgAdmin Download-Seite](https://www.pgadmin.org/download/pgadmin-4-windows/)
- [pgAdmin Dokumentation](https://www.pgadmin.org/docs/)
- [Chocolatey pgAdmin Paket](https://community.chocolatey.org/packages/pgadmin4)
- [PostgreSQL Download Windows](https://www.postgresql.org/download/windows/)

---

*Erstellt für Windows Server 2022 – PowerShell-basierte Installation*
