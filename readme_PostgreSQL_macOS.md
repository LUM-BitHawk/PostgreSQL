# PostgreSQL auf macOS installieren

Eine vollständige Schritt-für-Schritt-Anleitung zur Installation und Einrichtung von PostgreSQL auf macOS.

---

## Voraussetzungen

- macOS 12 (Monterey) oder neuer
- Administratorrechte
- Internetverbindung
- Terminal-Kenntnisse (Grundlagen)

---

## Methode 1: Installation via Homebrew (empfohlen)

### Schritt 1 – Homebrew installieren

Falls Homebrew noch nicht installiert ist:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Installation prüfen:

```bash
brew --version
```

---

### Schritt 2 – PostgreSQL installieren

```bash
brew install postgresql@16
```

> **Hinweis:** Ersetze `16` durch die gewünschte Version (z. B. `@15`, `@17`). Die aktuelle Standardversion erhältst du mit `brew install postgresql`.

---

### Schritt 3 – PATH konfigurieren

Damit die PostgreSQL-Befehle im Terminal verfügbar sind, muss der Pfad gesetzt werden.

**Für zsh (Standard ab macOS Catalina):**

```bash
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Für bash:**

```bash
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```

> **Hinweis für Intel-Macs:** Der Pfad lautet `/usr/local/opt/postgresql@16/bin` statt `/opt/homebrew/opt/postgresql@16/bin`.

---

### Schritt 4 – PostgreSQL-Dienst starten

```bash
brew services start postgresql@16
```

Status prüfen:

```bash
brew services list
```

PostgreSQL sollte als `started` angezeigt werden.

---

### Schritt 5 – Installation prüfen

```bash
psql --version
```

Ausgabe (Beispiel):

```
psql (PostgreSQL) 16.x
```

---

### Schritt 6 – Mit PostgreSQL verbinden

Direkt als aktueller macOS-Benutzer verbinden (kein Passwort nötig):

```bash
psql postgres
```

Du bist jetzt in der PostgreSQL-Shell (`postgres=#`).

---

## Methode 2: Installation via Postgres.app (grafisch)

Eine einfache Alternative ohne Terminal-Kenntnisse.

1. Gehe zu [https://postgresapp.com](https://postgresapp.com)
2. Lade die aktuelle Version herunter
3. Verschiebe `Postgres.app` in den Ordner `/Programme`
4. Öffne die App und klicke auf **Initialize**
5. Füge den Pfad hinzu (einmalig im Terminal):

```bash
sudo mkdir -p /etc/paths.d && echo /Applications/Postgres.app/Contents/Versions/latest/bin | sudo tee /etc/paths.d/postgresapp
```

---

## Grundlegende Einrichtung

### Neuen Datenbankbenutzer erstellen

```sql
-- In der psql-Shell (psql postgres):
CREATE USER meinbenutzer WITH PASSWORD 'sicherespasswort';
```

### Neue Datenbank erstellen

```sql
CREATE DATABASE meinedatenbank OWNER meinbenutzer;
```

### Berechtigungen vergeben

```sql
GRANT ALL PRIVILEGES ON DATABASE meinedatenbank TO meinbenutzer;
```

### Verbindung mit der neuen Datenbank

```bash
psql -U meinbenutzer -d meinedatenbank
```

### psql-Shell beenden

```sql
\q
```

---

## Dienstverwaltung

| Aktion | Befehl |
|---|---|
| Starten | `brew services start postgresql@16` |
| Stoppen | `brew services stop postgresql@16` |
| Neu starten | `brew services restart postgresql@16` |
| Status prüfen | `brew services list` |

---

## Nützliche psql-Befehle

| Befehl | Beschreibung |
|---|---|
| `\l` | Alle Datenbanken auflisten |
| `\c datenbankname` | Zu einer Datenbank wechseln |
| `\dt` | Alle Tabellen der aktuellen Datenbank |
| `\du` | Alle Benutzer/Rollen auflisten |
| `\d tabellenname` | Tabellenstruktur anzeigen |
| `\?` | Alle psql-Befehle anzeigen |
| `\q` | psql beenden |

---

## Konfigurationsdatei anpassen (optional)

Die Hauptkonfigurationsdatei von PostgreSQL:

```bash
# Pfad zur postgresql.conf anzeigen
psql -U postgres -c 'SHOW config_file;'
```

Typische Einstellungen in `postgresql.conf`:

```ini
# Verbindungen
max_connections = 100

# Speicher
shared_buffers = 128MB

# Logging
log_destination = 'stderr'
logging_collector = on
```

Nach Änderungen PostgreSQL neu starten:

```bash
brew services restart postgresql@16
```

---

## Deinstallation

```bash
# Dienst stoppen
brew services stop postgresql@16

# PostgreSQL entfernen
brew uninstall postgresql@16

# Datenbankdaten löschen (Vorsicht: unwiderruflich!)
rm -rf /opt/homebrew/var/postgresql@16
```

---

## Fehlerbehebung

### Problem: `psql: command not found`
→ PATH wurde nicht korrekt gesetzt. Schritt 3 wiederholen und Terminal neu starten.

### Problem: `FATAL: role "username" does not exist`
→ Benutzer existiert in PostgreSQL nicht. Einen neuen Benutzer erstellen (siehe Abschnitt *Neuen Datenbankbenutzer erstellen*).

### Problem: Dienst startet nicht
```bash
# Logs prüfen
brew services log postgresql@16

# Oder direkt:
cat /opt/homebrew/var/log/postgresql@16.log
```

### Problem: Port 5432 bereits belegt
```bash
# Prozess auf Port 5432 finden
lsof -i :5432

# Prozess beenden (PID ersetzen)
kill -9 <PID>
```

---

## Weiterführende Links

- [Offizielle PostgreSQL-Dokumentation](https://www.postgresql.org/docs/)
- [Homebrew Formulae: postgresql](https://formulae.brew.sh/formula/postgresql@16)
- [Postgres.app](https://postgresapp.com)

---

*Erstellt für macOS · PostgreSQL 16 · Homebrew*
