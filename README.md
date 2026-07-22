# termin-o-mat

Einfaches Terminfindungs-Tool ohne Anmeldung – Alternative zu Doodle & Co., als einzelne `index.html` ohne Build-Prozess, Backend im eigenen Code oder Server. Datenhaltung übernimmt eine offene Firebase-Realtime-Database, angesprochen per REST (`fetch`), ohne SDK.

## Funktionsweise

**Events statt Accounts.** Es gibt keine Benutzerkonten. Stattdessen ist jeder Termin-Vorschlag ein eigenständiges "Event" mit einer zufälligen ID, referenziert über den URL-Parameter `?e=<id>`. Der Link *ist* der Zugang – wer ihn hat, kann mitmachen; wer ihn nicht hat, kommt nicht rein (die Firebase-Regeln blockieren das Auflisten aller Events, siehe unten).

- **Ohne `?e=...`-Parameter** zeigt die Seite nur einen Button "Neues Event erstellen". Klick darauf legt ein neues, leeres Event mit zufälliger ID an und leitet dorthin weiter.
- **Mit `?e=<id>`** wird das jeweilige Event geladen: eine Tabelle mit Namen (Zeilen) und vorgeschlagenen Terminen (Spalten). Jeder trägt oben seinen Namen ein (wird lokal im Browser gemerkt) und klickt sich durch den Status für jeden Termin: `–` (keine Angabe) → `✓` (kann) → `✗` (kann nicht) → zurück zu `–`. Die eigene Zeile steht immer oben und ist die einzige klickbare.
- Über einen Button können weitere Termin-Vorschläge zur Liste hinzugefügt werden.
- Eine Summenzeile zählt die Zusagen pro Termin und markiert den/die Termin(e) mit den meisten Zusagen mit ★.
- **Begrüßungstext:** Solange ein Event noch leer ist (keine Termine, keine Stimmen), kann ein freier Begrüßungs-/Erklärungstext eingetragen werden. Sobald das Event Daten enthält, wird nur noch der Text angezeigt; bearbeiten darf ihn danach nur noch, wessen Name mit dem des ersten Speichernden (`welcomeAuthor`) übereinstimmt – eine eng begrenzte, rein auf dieses Textfeld beschränkte Berechtigung, keine generelle Admin-Rolle.
- Alle 5 Sekunden wird der Stand per Polling neu geladen (kein Realtime-Listener), damit alle Teilnehmer synchron bleiben.

## Datenmodell

```
events/
  <eventId>/
    termine: ["2026-10-10", ...]
    stimmen: { "Name": { "2026-10-10": "green" | "red" } }
    welcomeText: "..."       (optional)
    welcomeAuthor: "Name"    (optional, gesetzt beim ersten Speichern von welcomeText)
    createdAt: <timestamp>   (verhindert, dass Firebase den leeren Knoten beim Anlegen verwirft)
```

## Sicherheitsmodell

Es gibt keine echte Authentifizierung – Zugriffsschutz basiert allein darauf, dass Event-IDs zufällige, nicht erratbare UUIDs sind ("security by unguessable link", wie bei den meisten Terminfindungs-Tools). Damit das trägt, sind in den Firebase-Regeln (Firebase Console → Realtime Database → Rules, nicht Teil dieses Repos) das Auflisten von `/events` sowie der Root-Pfad `/` gesperrt; einzelne bekannte `events/<id>`-Pfade bleiben lesbar/schreibbar.

## Hosting

Statisches HTML, aktuell über GitHub Pages ausgeliefert – kein Server, kein Build-Schritt nötig.
