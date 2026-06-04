# PostgreSQL – Installationsanleitung für Ubuntu 26.04 LTS

> **Ziel:** PostgreSQL auf einem frischen Ubuntu 26.04 LTS-System installieren, absichern und in Betrieb nehmen.

---

## Voraussetzungen

- Ubuntu 26.04 LTS (Noble Numbat oder neuer)
- Benutzer mit `sudo`-Rechten
- Internetzugang

---

## Schritt 1 – System aktualisieren

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Schritt 2 – Offizielle PostgreSQL-Paketquelle hinzufügen

Ubuntu enthält PostgreSQL in den Standard-Repos, aber das offizielle PGDG-Repository liefert immer die aktuellste Version.

```bash
# Benötigte Hilfspakete installieren
sudo apt install -y curl ca-certificates gnupg

# Signaturschlüssel herunterladen und speichern
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/pgdg.gpg

# Repository zur Paketliste hinzufügen
echo "deb [signed-by=/etc/apt/trusted.gpg.d/pgdg.gpg] \
  https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  | sudo tee /etc/apt/sources.list.d/pgdg.list

# Paketlisten aktualisieren
sudo apt update
```

---

## Schritt 3 – PostgreSQL installieren

```bash
# Aktuelle Stable-Version installieren (z. B. PostgreSQL 17)
sudo apt install -y postgresql postgresql-contrib

# Installierte Version prüfen
psql --version
```

> `postgresql-contrib` enthält nützliche Erweiterungen (z. B. `pg_stat_statements`, `uuid-ossp`).

---

## Schritt 4 – Dienst starten und aktivieren

```bash
# Dienststatus prüfen (läuft meist sofort nach der Installation)
sudo systemctl status postgresql

# Dienst starten (falls nicht aktiv)
sudo systemctl start postgresql

# Autostart beim Systemstart aktivieren
sudo systemctl enable postgresql
```

---

## Schritt 5 – Mit PostgreSQL verbinden

Nach der Installation existiert automatisch ein Systembenutzer und ein Datenbankbenutzer namens `postgres`.

```bash
# Zur postgres-Shell wechseln
sudo -i -u postgres

# PostgreSQL interaktive Konsole öffnen
psql
```

In der `psql`-Konsole:

```sql
-- PostgreSQL-Version anzeigen
SELECT version();

-- Alle Datenbanken auflisten
\l

-- Konsole beenden
\q
```

Zurück zum normalen Benutzer:

```bash
exit
```

---

## Schritt 6 – Passwort für den postgres-Benutzer setzen

```bash
sudo -i -u postgres psql
```

```sql
ALTER USER postgres WITH PASSWORD 'SicheresPasswort123!';
\q
```

---

## Schritt 7 – Neue Datenbank und Benutzer anlegen

```bash
sudo -i -u postgres psql
```

```sql
-- Neuen Datenbankbenutzer anlegen
CREATE USER myuser WITH PASSWORD 'MeinPasswort456!';

-- Neue Datenbank anlegen
CREATE DATABASE mydb;

-- Dem Benutzer alle Rechte auf die Datenbank geben
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;

\q
```

---

## Schritt 8 – Authentifizierungsmethode anpassen (optional)

Die Konfigurationsdatei `pg_hba.conf` steuert, wer sich wie verbinden darf.

```bash
# Pfad zur Konfigurationsdatei finden
sudo -u postgres psql -c "SHOW hba_file;"

# Datei bearbeiten (Versionsnummer ggf. anpassen)
sudo nano /etc/postgresql/17/main/pg_hba.conf
```

Typische Zeile für lokale Passwort-Authentifizierung:

```
# TYPE  DATABASE  USER    ADDRESS     METHOD
local   all       all                 scram-sha-256
host    all       all     127.0.0.1/32  scram-sha-256
host    all       all     ::1/128       scram-sha-256
```

Nach Änderungen PostgreSQL neu laden:

```bash
sudo systemctl reload postgresql
```

---

## Schritt 9 – Externe Verbindungen erlauben (optional)

> **Hinweis:** Nur aktivieren, wenn der Server aus dem Netzwerk erreichbar sein soll. Firewall vorher einrichten!

### 9.1 – postgresql.conf anpassen

```bash
sudo nano /etc/postgresql/17/main/postgresql.conf
```

Zeile suchen und anpassen:

```
listen_addresses = '*'
```

### 9.2 – pg_hba.conf erweitern

```bash
sudo nano /etc/postgresql/17/main/pg_hba.conf
```

Zeile am Ende hinzufügen (für alle IPs):

```
host    all    all    0.0.0.0/0    scram-sha-256
```

Oder für ein bestimmtes Subnetz (empfohlen):

```
host    all    all    192.168.1.0/24    scram-sha-256
```

### 9.3 – Firewall-Port freischalten (UFW)

```bash
sudo ufw allow 5432/tcp
sudo ufw reload
sudo ufw status
```

### 9.4 – PostgreSQL neu starten

```bash
sudo systemctl restart postgresql
```

---

## Schritt 10 – Verbindung testen

### Lokal

```bash
psql -U myuser -d mydb -h 127.0.0.1
```

### Von einem anderen Rechner

```bash
psql -U myuser -d mydb -h <SERVER-IP>
```

---

## Nützliche Befehle im Überblick

| Befehl | Beschreibung |
|--------|--------------|
| `sudo systemctl start postgresql` | Dienst starten |
| `sudo systemctl stop postgresql` | Dienst stoppen |
| `sudo systemctl restart postgresql` | Dienst neu starten |
| `sudo systemctl status postgresql` | Dienststatus anzeigen |
| `sudo -i -u postgres psql` | Als postgres-Admin einloggen |
| `psql -U user -d db -h host` | Als bestimmter Benutzer verbinden |
| `\l` | Alle Datenbanken auflisten (in psql) |
| `\du` | Alle Benutzer auflisten (in psql) |
| `\c dbname` | Zu einer Datenbank wechseln (in psql) |
| `\dt` | Alle Tabellen anzeigen (in psql) |
| `\q` | psql beenden |

---

## Konfigurationsdateien

| Datei | Zweck |
|-------|-------|
| `/etc/postgresql/17/main/postgresql.conf` | Hauptkonfiguration (Port, Memory, Logging …) |
| `/etc/postgresql/17/main/pg_hba.conf` | Authentifizierungsregeln |
| `/etc/postgresql/17/main/pg_ident.conf` | Benutzer-Mapping |
| `/var/lib/postgresql/17/main/` | Datenbankdateien (Data Directory) |
| `/var/log/postgresql/` | Log-Dateien |

---

## Deinstallation (falls nötig)

```bash
# Dienst stoppen
sudo systemctl stop postgresql

# Pakete entfernen
sudo apt remove --purge postgresql postgresql-* -y

# Datenbankdaten löschen (ACHTUNG: unwiderruflich!)
sudo rm -rf /var/lib/postgresql/
sudo rm -rf /etc/postgresql/
sudo rm -rf /var/log/postgresql/

# Paketliste bereinigen
sudo apt autoremove -y
```

---

## Weiterführende Links

- [PostgreSQL Offizielle Dokumentation](https://www.postgresql.org/docs/)
- [PGDG Apt Repository](https://wiki.postgresql.org/wiki/Apt)
- [PostgreSQL pg_hba.conf Referenz](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)

---

*Erstellt für Ubuntu 26.04 LTS · PostgreSQL 17 · Stand: Juni 2026*
