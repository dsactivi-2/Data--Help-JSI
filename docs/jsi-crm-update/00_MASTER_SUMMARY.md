# JSI CRM Update – Master Summary

Stand: 2026-09-19

## Ziel

Das bestehende Legacy-CRM wird nicht 1:1 nachgebaut. Ziel ist eine neue, deutlich einfachere, zentrale und skalierbare CRM-Architektur für Kandidaten, Kunden, Stellen, Suche, MCPs und neue UI.

Grundprinzip:

> Eine Person = ein Kandidat = eine individuelle, fortlaufende Kandidaten-ID. Diese ID bleibt dauerhaft unverändert, solange die Person im System geführt wird.

Die Reihenfolge der Neustrukturierung:

1. Datenbank-Grundstruktur festlegen
2. Statusmodell komplett neu besprechen
3. Berufe / Skills / Qualifikationen / Filterstruktur bereinigen
4. Suche und gespeicherte Filterprofile definieren
5. Workflows aus dem Legacy-CRM rekonstruieren und nur notwendige Funktionen übernehmen
6. MCPs und UI auf das neue Modell setzen
7. Migration erst nach fachlicher Freigabe durchführen

---

# 1. Standardsprache der Datenbank

Die neue Datenbank soll technisch und fachlich standardmäßig Englisch verwenden.

- Tabellen- und Spaltennamen: Englisch
- kanonische Codes: Englisch / sprachneutral
- kanonische fachliche Werte: Englisch
- technische Beziehungen über IDs / Codes, nicht über angezeigten Freitext

Für echte Berufe verwenden wir fachlich die Begriffe:

- Beruf
- Berufs-ID
- Berufsbezeichnung

Keine Vermischung mit künstlichen Stellen-/Suchprofilen.

Die UI kann Deutsch, Englisch, Bosnisch und Serbisch anzeigen. Die UI-Übersetzung ist Darstellung, nicht die fachliche Identität des Datensatzes.

---

# 2. Berufsbereinigung

Über 10–12 Jahre sind viele Berufsbezeichnungen als Freitext entstanden: Tippfehler, Abkürzungen, Übersetzungen, unterschiedliche Schreibweisen und teilweise völlig andere Bezeichnungen für fachlich denselben Beruf.

Ziel:

- gleiche Bedeutungen erkennen
- auf einen kanonischen Beruf zusammenführen
- alle Kandidaten-, Ausbildungs-, Berufserfahrungs- und weiteren Berufsreferenzen auf diesen Beruf umstellen
- alte Varianten nur temporär als Migrations-/Auditinformation nutzen
- im neuen Kernsystem keine tausenden Schreibvarianten mehr führen

Beispiel:

```text
Automehanicar
Auto mehanicar
KFZ Mechaniker
Kfz-Mechaniker
Automechaniker
Automotive Mechanic

→ Beruf: Automotive Mechanic
→ eine Berufs-ID
```

Wichtig: Ein Merge bedeutet nicht nur Umbenennen. Alle Personen und abhängigen Relationen müssen auf denselben neuen Beruf zeigen.

---

# 3. Echter Beruf vs. Stellen-/Suchprofil

Historisch wurden teilweise künstliche „Berufe“ angelegt, die fachlich eigentlich nur Suchgruppen waren.

Beispiel:

`Internet-Monteur` kann ein Stellen-/Suchprofil sein, unter dem mehrere echte Berufe zusammengefasst werden:

- Elektriker
- Installateur
- Mechatroniker
- weitere passende Berufe

Deshalb trennen:

```text
Beruf        = echter Beruf / fachliche Berufszuordnung
Skill        = konkrete Fähigkeit
Stellenprofil / Suchprofil = kundenspezifische oder tätigkeitsbezogene Suchdefinition
```

Ein Stellen-/Suchprofil darf mehrere Berufe und Skills referenzieren.

---

# 4. Berufszuordnung eines Kandidaten aus allen relevanten Quellen

Ein Kandidat darf nicht nur anhand eines einzelnen Haupt-Berufsfeldes bewertet werden.

Berufsinformationen können im Legacy-System an mehreren Stellen vorkommen:

- Haupt-Kandidatenprofil
- Ausbildung / `idk_kandidat_edukacija`
- Berufserfahrung / `idk_kandidat_radno_iskustvo`
- Diplom-/Anerkennungsbereich
- Dippel/NP bzw. weitere historische Kandidatenbereiche
- Job-/Positions-Mappings
- weitere Freitextfelder

Feste Migrationsregel:

> Berufszuordnung niemals aus nur einer Tabelle ableiten. Alle relevanten Kandidatenquellen, Ausbildung, Berufserfahrung und beruflichen Zuordnungen gemeinsam auswerten.

Eine Person kann mehrere Berufe haben.

---

# 5. Strukturierte Datenerfassung statt Freitext

Alles, was später gesucht, gefiltert, verglichen oder automatisiert werden soll, soll möglichst strukturiert gespeichert werden.

Geeignete UI-Komponenten:

- Dropdown
- Multi-Select
- Checkboxen
- Choice / Radio
- Toggle
- definierte Kategorien

Freitext bleibt für Notizen und ergänzende Beschreibungen.

Grundregel:

> Freitext ist Zusatzinformation, nicht die primäre Quelle für Filterlogik.

Auch Berufserfahrung soll soweit möglich strukturiert erfasst werden: Beruf, Tätigkeitsbereiche, Erfahrungsdauer, Niveau, Land, Führungsverantwortung, Führerschein, Skills usw.

---

# 6. Kunden, Stellen und gespeicherte Filterprofile

Ein Kunde kann mehrere Stellen / Suchbedarfe haben.

Beispiel:

```text
Kunde: XY GmbH
Stelle: Internet-Monteur
Bedarf: 10 Mitarbeiter
```

Dazu kann ein gespeichertes Suchprofil gehören:

```text
passende Berufe:
- Elektriker
- Installateur
- Mechatroniker

Skills:
- Kabelverlegung
- Bohren
- Routerinstallation

Sprache:
- Deutsch B1+

Führerschein:
- B
```

Die Suche soll zwei Wege unterstützen:

1. normale manuelle Filterung
2. gespeichertes Kunden-/Stellenprofil per Klick laden

Das Profil ist nur eine Suchdefinition. Filter dürfen für eine konkrete Suche temporär ergänzt oder geändert werden, ohne das gespeicherte Profil automatisch zu überschreiben.

---

# 7. Vector Search / semantische Suche

Neben strukturierter Suche soll eine semantische Vektorsuche für unstrukturierte Inhalte vorgesehen werden.

Geeignete Inhalte:

- Notizen
- Gesprächsverläufe
- Call Summaries
- CV-Text
- Dokumenttext
- Freitext in Berufserfahrung
- historische Kommentare

Nicht als primäre Vector-Suche verwenden:

- IDs
- Status
- Datum
- Beruf
- Skills
- Sprachen
- strukturierte Qualifikationen

Empfohlen:

```text
strukturierte Filter
+
semantische / Vector Search
```

---

# 8. Kandidaten-ID – unveränderliche Identität

Jeder Kandidat bekommt genau eine individuelle, fortlaufende Kandidaten-ID.

Beispiel:

```text
Kandidat A → 100001
Kandidat B → 100002
Kandidat C → 100003
```

Diese ID bleibt unverändert.

Sie darf nicht durch Migration, Statuswechsel, Berufs-Merge, Modulwechsel, Kunden-/Stellenzuordnung, API, MCP, UI oder Automationen geändert werden.

Die Datenbank soll Änderungen der Kandidaten-ID technisch blockieren.

Bei Migrationen gilt:

```text
alte Kandidaten-ID = neue Kandidaten-ID
```

Abweichung = kritischer Fehler.

---

# 9. Kandidaten-Eingangswege / Herkunft

Es wird mehrere Wege geben, wie neue Kandidaten ins System kommen:

- direkte manuelle Erfassung
- Website-/Web-Eintrag
- Link-/Formular-Eintrag
- API
- Import
- ggf. Partner oder Agent

Die Herkunft soll voraussichtlich NICHT als dauerhaftes Kernfeld am Kandidaten geführt werden.

Stattdessen wird der Erstellweg als Audit-/Eventinformation gespeichert, z. B.:

```text
candidate_created_event
- candidate_id
- created_at
- created_by
- channel
- form_id optional
- campaign_id optional
```

Damit bleibt die Entstehung nachvollziehbar, ohne das Kandidatenprofil mit einem historischen Feld zu belasten.

---

# 10. Statusmodell – noch nicht festgelegt

Das bestehende Statussystem ist historisch gewachsen und enthält mehrere sich überschneidende Statusarten.

Es soll NICHT 1:1 übernommen werden.

Vor Umsetzung müssen wir fachlich entscheiden:

- welche Status noch benötigt werden
- welche alten Status doppelt oder technisch entstanden sind
- welche Status zu Bewerbung, Kommunikation, Stelle oder einem anderen Prozess gehören
- welche Status vollständig entfallen

Bestehende Begriffe wie „reserviert“ werden nicht automatisch übernommen. Erst nach fachlicher Besprechung wird entschieden, ob sie erhalten, geändert oder entfernt werden.

---

# 11. MCP-Zielbild

Geplante MCP-Flächen:

1. interner MCP – read only
2. interner MCP – read/write mit Freigabeschalter und begrenzten Aktionen
3. interner vollständig offener MCP – über UI aktivierbar/deaktivierbar
4. Kunden-MCP – serverseitig streng eingeschränkte Felder und Aktionen
5. Kandidaten-MCP

Sicherheitsprinzip:

> Berechtigungen und Feldsichtbarkeit müssen serverseitig erzwungen werden, nicht nur über Prompts.

Sprachprinzip:

Ein Benutzer kann in seiner Sprache suchen. Der MCP löst Suchbegriffe auf kanonische Werte / IDs auf und führt die eigentliche Datenbankabfrage sprachneutral aus.

---

# 12. Arbeitsprinzip

Bei jedem alten Modul wird geprüft:

```text
Was macht es fachlich?
↓
Brauchen wir es weiterhin?
↓
BEHALTEN / ÄNDERN / ENTFERNEN
↓
Welche Daten braucht die neue UI dafür?
↓
Welche Relation / Tabelle braucht das neue Modell?
```

Das Legacy-CRM ist Referenz für den bisherigen Ablauf, aber nicht die Zielarchitektur.
