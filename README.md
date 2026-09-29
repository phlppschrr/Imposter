# Imposter

Mobile-first Partyspiel als installierbare PWA. **Lokales Spielen funktioniert nach dem ersten Laden komplett offline.** Für Online-Räume genügt klassisches Shared Hosting mit PHP und SQLite – kein Node.js, Docker, WebSocket-Server, npm-Build oder externer Dienst.

## Spielmodi

- **Klassisch:** Alle außer dem Imposter sehen dasselbe geheime Wort.
- **Mit Hinweis:** Der Imposter sieht nicht das Wort, aber die gewählte Kategorie.
- **Chaos:** Für größere Gruppen und mehrere Imposter gedacht.
- **Blitzrunde:** Kurze Runden mit zwei Minuten Diskussion.

Es gibt 8 Kategorien: Alltag, Essen & Trinken, Orte, Tiere, Freizeit & Sport, Medien & Kultur, Reisen sowie Beziehungen & Party. Die App enthält bereits weit über 100 Begriffe. Zusätzlich können pro Runde eigene Wörter verwendet werden.

### Altersfilter

**Jugendlich / familienfreundlich** verwendet ausschließlich den `youth`-Wortschatz. **Alle erwachsen (18+)** behält die normalen Begriffe bei und ergänzt je Kategorie erwachsenere Party-, Dating-, Ausgeh- und Alltagsthemen. Der Filter wird sowohl lokal als auch im Online-Raum angewendet.

## Offline-Modus

„Auf einem Gerät“ benötigt keinen Serverkontakt. Die Namen werden eingegeben, das Gerät wird zur Rollenanzeige herumgereicht, anschließend laufen Diskussion und Auflösung lokal. Der Service Worker speichert die statischen App-Dateien. Nach einem erfolgreichen ersten Aufruf kann dieser Modus deshalb auch ohne Internetverbindung gestartet werden.

Online-Funktionen sind offline naturgemäß nicht verfügbar.

## Online-Modus

Ein Spieler erstellt einen Raum und erhält einen fünfstelligen Code sowie einen teilbaren Link. Weitere Spieler treten mit Anzeigenamen bei. Es gibt keine Benutzerkonten.

Der Browser fragt den Raumzustand etwa alle 1,5 Sekunden über `api/index.php` ab. Das ist bewusst HTTP-Polling statt WebSockets und funktioniert auf gewöhnlichem PHP-Shared-Hosting. SQLite speichert Räume, Spieler, Rollen und Stimmen. Inaktive Räume werden nach sechs Stunden automatisch entfernt.

Sicherheitsrelevant: Das geheime Wort wird für Imposter serverseitig als `NULL` gespeichert und von der API nicht ausgeliefert. Host und Spieler werden über zufällige Sitzungstokens identifiziert.

## Hosting auf ALL-INKL oder anderem PHP-Webspace

Voraussetzungen:

- PHP mit PDO SQLite / SQLite3
- Schreibrecht des PHP-Prozesses im Verzeichnis `data/`
- HTTPS (für eine zuverlässig installierbare PWA dringend empfohlen)
- Apache/`.htaccess` ist hilfreich, aber die App benötigt keine Rewrite-Regeln.

### Installation

1. Repository herunterladen/auschecken.
2. **Den Inhalt des Repositorys unverändert** in das Zielverzeichnis der Domain/Subdomain hochladen, z. B. per FTP/SFTP.
3. Domain im Browser aufrufen. Es gibt keinen Build- oder Composer-Schritt.
4. Einen Online-Raum testweise erstellen. Beim ersten API-Aufruf erzeugt PHP automatisch `data/imposter.sqlite` und die Tabellen.
5. Falls das Erstellen fehlschlägt, Schreibrechte für `data/` prüfen. Die mitgelieferte `data/.htaccess` verhindert direkten Webzugriff auf die Datenbank.
6. HTTPS/SSL für die Domain aktivieren.

ALL-INKL dokumentiert PHP und vollen `.htaccess`-Zugriff für seine Webhosting-Tarife und nennt SQLite ausdrücklich als unterstützte SQL-Alternative. Eine separate MySQL/MariaDB-Datenbank muss daher für Imposter nicht angelegt werden.

### Unterverzeichnis

Die App verwendet überwiegend relative Pfade und kann z. B. unter `https://example.de/imposter/` liegen. `index.html`, `api/`, `src/`, `sw.js` und `manifest.webmanifest` müssen dabei gemeinsam in diesem Verzeichnis bleiben.

## Dateien

- `index.html` – App-Einstieg
- `src/main.js` – Oberfläche, lokaler Modus und Online-Polling
- `src/game.js` – lokale Spiellogik, Varianten und Wortschatz
- `api/index.php` – JSON-API und SQLite-Raumverwaltung
- `api/words.php` – serverseitiger Wortschatz
- `data/` – zur Laufzeit erzeugte SQLite-Datenbank
- `sw.js` / `manifest.webmanifest` – PWA/Offline-Unterstützung

## Entwicklung und Tests

Für die eigentliche Website sind keine Node-Abhängigkeiten erforderlich. Weil ES-Module und Service Worker unter `file://` eingeschränkt sind, sollte lokal ein beliebiger HTTP-Server verwendet werden. PHP bringt einen passenden Entwicklungsserver mit:

```bash
php -S localhost:8080
```

Dann `http://localhost:8080` öffnen.

Die dependency-freien JavaScript-Tests können optional mit Node 20+ ausgeführt werden:

```bash
node --test
```

Für PHP empfiehlt sich zusätzlich:

```bash
php -l api/index.php
php -l api/words.php
```

## Datenhaltung und Skalierung

SQLite ist für kleine private Party-Räume auf einem einzelnen Shared-Hosting-Account bewusst gewählt. WAL-Modus und ein Busy-Timeout reduzieren Konflikte bei gleichzeitigen Requests. Für sehr viele gleichzeitige Räume wäre MariaDB die nächste Ausbaustufe; für den vorgesehenen privaten Einsatz ist kein separater Datenbankserver notwendig.
