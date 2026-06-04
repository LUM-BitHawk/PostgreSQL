# pgAdmin Installation auf macOS – Schritt-für-Schritt-Anleitung

> **pgAdmin** ist das beliebteste Open-Source-Verwaltungstool für PostgreSQL.  
> Diese Anleitung führt dich durch die Installation auf macOS (Intel & Apple Silicon).

---

## Voraussetzungen

| Anforderung | Details |
|---|---|
| macOS-Version | macOS 11 Big Sur oder neuer |
| PostgreSQL | Bereits installiert (empfohlen: Version 14+) |
| Speicherplatz | mind. 500 MB frei |
| Internetverbindung | Für den Download erforderlich |

> **Tipp:** Falls PostgreSQL noch nicht installiert ist, kannst du es über [postgresql.org](https://www.postgresql.org/download/macosx/) oder Homebrew (`brew install postgresql`) nachholen.

---

## Option A – Installation via DMG-Datei (empfohlen für Einsteiger)

### Schritt 1 – pgAdmin herunterladen

1. Öffne deinen Browser und gehe zu:  
   👉 [https://www.pgadmin.org/download/pgadmin-4-macos/](https://www.pgadmin.org/download/pgadmin-4-macos/)
2. Klicke auf die neueste stabile Version (z. B. `pgAdmin 4 v8.x`).
3. Lade die Datei **`pgadmin4-8.x.dmg`** herunter (ca. 200–300 MB).

---

### Schritt 2 – DMG-Datei öffnen

1. Öffne den **Finder** und navigiere zu **Downloads**.
2. Doppelklicke auf die heruntergeladene `.dmg`-Datei.
3. Ein neues Fenster öffnet sich mit dem pgAdmin-Logo und einem Pfeil zum **Applications**-Ordner.

---

### Schritt 3 – pgAdmin in den Applications-Ordner verschieben

1. Ziehe das **pgAdmin 4**-Symbol per Drag & Drop in den **Applications**-Ordner.
2. Warte, bis der Kopiervorgang abgeschlossen ist.
3. Schliesse das DMG-Fenster.
4. Optional: Wirf das gemountete Laufwerk im Finder aus (Rechtsklick → *Auswerfen*).

---

### Schritt 4 – pgAdmin starten

1. Öffne den **Finder** → **Programme (Applications)**.
2. Doppelklicke auf **pgAdmin 4**.
3. Bei der ersten Ausführung erscheint möglicherweise eine macOS-Sicherheitswarnung:

   > *„pgAdmin 4" kann nicht geöffnet werden, weil es von einem nicht verifizierten Entwickler stammt.*

4. **Lösung für die Sicherheitswarnung:**
   - Öffne **Systemeinstellungen** → **Datenschutz & Sicherheit**.
   - Scrolle nach unten zum Abschnitt **Sicherheit**.
   - Klicke auf **„Trotzdem öffnen"** neben dem Hinweis zu pgAdmin 4.
   - Bestätige mit deinem macOS-Passwort.

---

### Schritt 5 – Erstkonfiguration im Browser

1. pgAdmin öffnet sich automatisch in deinem Standardbrowser (z. B. unter `http://127.0.0.1:5050`).
2. Du wirst aufgefordert, ein **Master-Passwort** zu setzen:
   - Wähle ein sicheres Passwort und merke es dir gut.
   - Dieses Passwort schützt alle gespeicherten Serververbindungen.
3. Klicke auf **OK**.

---

### Schritt 6 – PostgreSQL-Server verbinden

1. Klicke im linken Bereich (Browser-Panel) mit der rechten Maustaste auf **Servers**.
2. Wähle **Register → Server…**
3. Fülle die Felder im Dialog aus:

   **Tab „General":**
   | Feld | Wert |
   |---|---|
   | Name | z. B. `Lokaler PostgreSQL` |

   **Tab „Connection":**
   | Feld | Wert |
   |---|---|
   | Host name/address | `localhost` |
   | Port | `5432` |
   | Maintenance database | `postgres` |
   | Username | `postgres` (oder dein DB-Benutzername) |
   | Password | Dein PostgreSQL-Passwort |

4. Aktiviere optional **„Save password?"** für automatisches Einloggen.
5. Klicke auf **Save**.

✅ Der Server erscheint jetzt im linken Browser-Panel. Du kannst ihn aufklappen und deine Datenbanken verwalten.

---

## Option B – Installation via Homebrew (für Entwickler)

```bash
# Homebrew installieren (falls noch nicht vorhanden)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# pgAdmin 4 über Homebrew Cask installieren
brew install --cask pgadmin4

# pgAdmin starten
open /Applications/pgAdmin\ 4.app
```

> **Hinweis:** Mit Homebrew bleibt pgAdmin leicht aktualisierbar:  
> ```bash
> brew upgrade --cask pgadmin4
> ```

---

## Häufige Probleme & Lösungen

### pgAdmin öffnet sich nicht (Sicherheitswarnung)

```
Systemeinstellungen → Datenschutz & Sicherheit → "Trotzdem öffnen"
```

### Verbindung zu PostgreSQL schlägt fehl

- Stelle sicher, dass PostgreSQL läuft:
  ```bash
  brew services list | grep postgresql
  # oder
  pg_ctl status -D /usr/local/var/postgresql@14
  ```
- Starte PostgreSQL falls nötig:
  ```bash
  brew services start postgresql@14
  ```

### Port 5432 bereits belegt

```bash
# Prüfen, welcher Prozess den Port nutzt
sudo lsof -i :5432
```

### pgAdmin lädt nicht im Browser

- Prüfe, ob pgAdmin im Hintergrund läuft (Menüleiste oben rechts).
- Öffne manuell: [http://127.0.0.1:5050](http://127.0.0.1:5050)
- Starte pgAdmin neu über das App-Symbol im Dock.

---

## pgAdmin deinstallieren

```bash
# Option A: Manuell
# 1. pgAdmin in den Papierkorb ziehen (Programme → pgAdmin 4)
# 2. Konfigurationsdateien entfernen:
rm -rf ~/Library/Application\ Support/pgadmin
rm -rf ~/Library/Preferences/pgadmin*
rm -rf ~/Library/Logs/pgadmin*

# Option B: Via Homebrew
brew uninstall --cask pgadmin4
```

---

## Nützliche Links

- 📖 [Offizielle pgAdmin-Dokumentation](https://www.pgadmin.org/docs/)
- 📥 [pgAdmin Download-Seite](https://www.pgadmin.org/download/)
- 🐘 [PostgreSQL Download für macOS](https://www.postgresql.org/download/macosx/)
- 💬 [pgAdmin Community Forum](https://www.pgadmin.org/support/)

---

*Erstellt für macOS · pgAdmin 4 · Stand: Juni 2025*
