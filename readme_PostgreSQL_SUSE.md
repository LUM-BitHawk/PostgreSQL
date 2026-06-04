# PostgreSQL – Installationsanleitung für openSUSE Leap

> Getestet auf openSUSE Leap 15.5 / 15.6 · PostgreSQL 15/16

---

## Inhaltsverzeichnis

1. [Voraussetzungen](#1-voraussetzungen)
2. [Repository & Pakete installieren](#2-repository--pakete-installieren)
3. [Datenbank-Cluster initialisieren](#3-datenbank-cluster-initialisieren)
4. [Dienst starten & aktivieren](#4-dienst-starten--aktivieren)
5. [Firewall konfigurieren](#5-firewall-konfigurieren)
6. [Erster Login & postgres-Passwort setzen](#6-erster-login--postgres-passwort-setzen)
7. [Neue Datenbank & Benutzer anlegen](#7-neue-datenbank--benutzer-anlegen)
8. [Grundlegende Konfiguration](#8-grundlegende-konfiguration)
9. [Verbindung testen](#9-verbindung-testen)
10. [Nützliche Befehle](#10-nützliche-befehle)
11. [Deinstallation](#11-deinstallation)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. Voraussetzungen

- openSUSE Leap 15.5 oder 15.6 (64-Bit)
- Root-Zugriff oder `sudo`-Rechte
- Aktive Internetverbindung

System-Paketliste aktualisieren:

```bash
sudo zypper refresh
sudo zypper update -y
```

---

## 2. Repository & Pakete installieren

### Option A – Paket aus den offiziellen openSUSE-Repos (empfohlen für den Einstieg)

```bash
# Verfügbare PostgreSQL-Versionen anzeigen
sudo zypper search postgresql

# Hauptpaket + Client + Contrib-Module installieren (Beispiel: Version 16)
sudo zypper install -y postgresql16 postgresql16-server postgresql16-contrib
```

### Option B – PostgreSQL-eigenes Repository (für neueste Versionen)

```bash
# Repo hinzufügen (Leap 15)
sudo zypper addrepo https://download.postgresql.org/pub/repos/zypp/15/openSUSE-Leap-15/ PGDG

# Schlüssel importieren & Repo aktivieren
sudo zypper --gpg-auto-import-keys refresh PGDG

# Pakete installieren
sudo zypper install -y postgresql16 postgresql16-server postgresql16-contrib
```

> **Hinweis:** Ersetze `16` durch die gewünschte Versionsnummer (z. B. `15`, `17`).

---

## 3. Datenbank-Cluster initialisieren

Vor dem ersten Start muss der Datenbank-Cluster angelegt werden:

```bash
sudo postgresql-setup --initdb
# oder bei explizit versioniertem Befehl:
sudo /usr/lib/postgresql16/bin/initdb -D /var/lib/pgsql/data
```

Standarddatenverzeichnis: `/var/lib/pgsql/data`

---

## 4. Dienst starten & aktivieren

```bash
# Dienst sofort starten
sudo systemctl start postgresql

# Dienst beim Booten automatisch starten
sudo systemctl enable postgresql

# Status prüfen
sudo systemctl status postgresql
```

Erwartete Ausgabe (Auszug):

```
● postgresql.service - PostgreSQL Database Server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled)
     Active: active (running) ...
```

---

## 5. Firewall konfigurieren

Falls die integrierte Firewall (`firewalld`) aktiv ist, PostgreSQL-Port freigeben:

```bash
# Status prüfen
sudo systemctl status firewalld

# PostgreSQL-Port 5432 dauerhaft öffnen
sudo firewall-cmd --permanent --add-service=postgresql
# oder manuell:
sudo firewall-cmd --permanent --add-port=5432/tcp

# Firewall-Regeln neu laden
sudo firewall-cmd --reload

# Prüfen ob Regel aktiv ist
sudo firewall-cmd --list-ports
```

> **Sicherheitshinweis:** Öffne Port 5432 nur, wenn externe Verbindungen benötigt werden. In Produktionsumgebungen sollte der Zugriff auf bestimmte IP-Adressen beschränkt werden.

---

## 6. Erster Login & postgres-Passwort setzen

PostgreSQL legt automatisch den Systembenutzer und Datenbankbenutzer `postgres` an.

```bash
# Zum postgres-Systembenutzer wechseln
sudo -i -u postgres

# PostgreSQL-Shell öffnen
psql

# Passwort für den postgres-Datenbankbenutzer setzen
ALTER USER postgres WITH PASSWORD 'MeinSicheresPasswort!';

# psql beenden
\q

# Zurück zum normalen Benutzer
exit
```

---

## 7. Neue Datenbank & Benutzer anlegen

```bash
sudo -i -u postgres psql
```

Innerhalb von `psql`:

```sql
-- Neuen Benutzer anlegen
CREATE USER meinbenutzer WITH PASSWORD 'GeheimesPasswort!';

-- Neue Datenbank anlegen
CREATE DATABASE meinedatenbank;

-- Alle Rechte an Benutzer vergeben
GRANT ALL PRIVILEGES ON DATABASE meinedatenbank TO meinbenutzer;

-- Überprüfen
\l          -- Datenbanken auflisten
\du         -- Benutzer auflisten

-- Beenden
\q
```

---

## 8. Grundlegende Konfiguration

### 8.1 postgresql.conf – Server-Einstellungen

```bash
sudo nano /var/lib/pgsql/data/postgresql.conf
```

Wichtige Parameter:

| Parameter | Standardwert | Empfehlung |
|---|---|---|
| `listen_addresses` | `'localhost'` | `'*'` für externe Verbindungen |
| `port` | `5432` | Standardwert beibehalten |
| `max_connections` | `100` | Je nach Last anpassen |
| `shared_buffers` | `128MB` | 25 % des RAM (Faustregel) |
| `log_destination` | `'stderr'` | `'csvlog'` für strukturiertes Logging |

Beispiel – externe Verbindungen erlauben:

```ini
listen_addresses = '*'
```

### 8.2 pg_hba.conf – Authentifizierung

```bash
sudo nano /var/lib/pgsql/data/pg_hba.conf
```

Zeile am Ende einfügen, um Verbindungen aus einem lokalen Netz zu erlauben:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             192.168.1.0/24          scram-sha-256
```

Nach Änderungen Dienst neu starten:

```bash
sudo systemctl restart postgresql
```

---

## 9. Verbindung testen

### Lokal (als postgres-Benutzer)

```bash
psql -U postgres -c "SELECT version();"
```

### Mit spezifischem Benutzer und Datenbank

```bash
psql -U meinbenutzer -d meinedatenbank -h 127.0.0.1 -p 5432
```

### Von einem Remote-Host

```bash
psql -U meinbenutzer -d meinedatenbank -h <SERVER-IP> -p 5432
```

---

## 10. Nützliche Befehle

### Dienstverwaltung

```bash
sudo systemctl start postgresql     # Starten
sudo systemctl stop postgresql      # Stoppen
sudo systemctl restart postgresql   # Neustart
sudo systemctl reload postgresql    # Konfiguration neu laden (kein Neustart)
sudo systemctl status postgresql    # Status anzeigen
```

### psql-Kurzreferenz

| Befehl | Beschreibung |
|--------|-------------|
| `\l` | Alle Datenbanken auflisten |
| `\c datenbankname` | Datenbank wechseln |
| `\dt` | Tabellen der aktuellen DB anzeigen |
| `\du` | Alle Benutzer/Rollen auflisten |
| `\d tabellenname` | Tabellenstruktur anzeigen |
| `\h` | SQL-Hilfe |
| `\?` | psql-Befehle Hilfe |
| `\q` | psql beenden |

### Backup & Restore

```bash
# Einzelne Datenbank sichern
pg_dump -U postgres meinedatenbank > backup.sql

# Alle Datenbanken sichern
pg_dumpall -U postgres > alle_datenbanken.sql

# Datenbank wiederherstellen
psql -U postgres meinedatenbank < backup.sql
```

---

## 11. Deinstallation

```bash
# Dienst stoppen und deaktivieren
sudo systemctl stop postgresql
sudo systemctl disable postgresql

# Pakete entfernen
sudo zypper remove postgresql16 postgresql16-server postgresql16-contrib

# Daten löschen (ACHTUNG: unwiderruflich!)
sudo rm -rf /var/lib/pgsql/data
```

---

## 12. Troubleshooting

### Dienst startet nicht

```bash
# Logs prüfen
sudo journalctl -u postgresql -n 50 --no-pager

# Oder direkt im PostgreSQL-Logverzeichnis
ls -lh /var/lib/pgsql/data/log/
```

### Verbindung verweigert

- Prüfen ob der Dienst läuft: `sudo systemctl status postgresql`
- Prüfen ob Port offen ist: `ss -tlnp | grep 5432`
- `listen_addresses` in `postgresql.conf` kontrollieren
- Firewall-Regeln prüfen: `sudo firewall-cmd --list-all`

### Authentifizierung schlägt fehl

- `pg_hba.conf` auf korrekte Einträge prüfen
- Nach Änderungen Dienst neu starten
- Authentifizierungsmethode (`md5`, `scram-sha-256`, `trust`) kontrollieren

### Datenverzeichnis-Berechtigungen

```bash
# Korrekte Rechte setzen
sudo chown -R postgres:postgres /var/lib/pgsql/data
sudo chmod 700 /var/lib/pgsql/data
```

---

## Weiterführende Links

- [PostgreSQL Offizielle Dokumentation](https://www.postgresql.org/docs/)
- [openSUSE PostgreSQL Wiki](https://en.opensuse.org/PostgreSQL)
- [PostgreSQL Global Development Group – Linux Downloads](https://www.postgresql.org/download/linux/suse/)

---

*Erstellt für openSUSE Leap 15.5 / 15.6 · Stand: Juni 2026*
