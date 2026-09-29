# Imposter

Mobile-first, offline-fähiges Partyspiel mit optionalem selbstgehostetem Online-Modus. Keine Accounts, keine externe Datenbank und keine SaaS-Abhängigkeit.

## Features
- Offline lokal für 3–12 Personen auf einem Gerät
- Online-Räume mit Raumcode/Share-Link, Reconnect, Host-Steuerung, privaten Rollen und Abstimmung
- Kategorien, eigene Wörter, 1–3 Imposter und Diskussionstimer
- Installierbare PWA mit Service Worker und Dark/Light Mode
- Self-hosted Node.js + WebSocket; aktive Räume liegen nur im RAM

## Entwicklung
Voraussetzung: Node.js 20+.

```bash
npm install
npm run dev
```

## Tests & Produktion
```bash
npm test
npm run build
npm start
```

`npm start` serviert den Build und WebSockets gemeinsam auf Port 3000. In Produktion HTTPS verwenden.

## Docker
```bash
docker build -t imposter .
docker run --rm -p 3000:3000 imposter
```

## Architektur
Der lokale Modus läuft vollständig im Browser. Im Online-Modus bleiben Räume und Rollen im Node-Prozess. Das geheime Wort wird nur an Nicht-Imposter gesendet; Imposter erhalten serverseitig `word: null`. Für mehrere Serverinstanzen oder dauerhafte Statistiken kann später SQLite ergänzt werden.
