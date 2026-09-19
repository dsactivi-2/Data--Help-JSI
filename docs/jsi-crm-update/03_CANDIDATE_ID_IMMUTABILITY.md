# Candidate ID – unveränderliche Identität

Stand: 2026-09-19

## Feste Regel

Jeder Kandidat erhält genau eine individuelle, fortlaufende Kandidaten-ID.

Beispiel:

```text
Kandidat A → 100001
Kandidat B → 100002
Kandidat C → 100003
```

Die Kandidaten-ID bleibt unverändert, solange der Kandidat im System geführt wird.

Sie darf NICHT verändert werden durch:

- Migration
- Datenbereinigung
- Berufs-Merge
- Statusänderung
- Modulwechsel
- Bewerbung auf eine andere Stelle
- Zuordnung zu einem Kunden
- UI-Bearbeitung
- API
- MCP
- Automationen
- Skripte

---

# Datenbank-Schutz

Die Kandidaten-ID soll nach dem Anlegen nicht mehr updatebar sein.

Empfohlene Schutzschichten:

- Primary Key / Unique Constraint
- Foreign Keys auf abhängigen Tabellen
- DB-Trigger oder vergleichbarer Schutz gegen ID-Änderung
- keine `ON UPDATE CASCADE`-Logik für die Kandidaten-ID
- keine normale API-Funktion zum Ändern der ID

Sinngemäß:

```text
OLD.candidate_id != NEW.candidate_id
→ UPDATE ABLEHNEN
```

---

# API- und MCP-Schutz

Die Kandidaten-ID darf in normalen Update-Tools nicht als veränderbares Feld angeboten werden.

Erlaubt:

```text
update_candidate(
  candidate_id=12345,
  phone=...,
  status=...
)
```

Nicht erlaubt:

```text
update_candidate(
  candidate_id=12345,
  new_candidate_id=67890
)
```

---

# UI-Schutz

Die Kandidaten-ID darf angezeigt werden, aber nur read-only.

```text
Candidate ID: 12345
[read only]
```

---

# Migrations-Schutz

Vor Migration:

- vollständige Liste aller Kandidaten-IDs sichern
- Duplikate / ungültige IDs prüfen
- Mapping Alt-ID → Neu-ID erzeugen
- Zielregel: Alt-ID = Neu-ID

Nach Migration:

- Anzahl vergleichen
- jede ID gegen Quelle prüfen
- fehlende IDs melden
- unerwartete neue IDs melden
- geänderte IDs als kritischen Fehler behandeln

Beispiel:

```text
source candidate_id = 12345
new candidate_id    = 12345
→ OK

source candidate_id = 12345
new candidate_id    = 98765
→ CRITICAL ERROR
```

---

# Dubletten

Auch bei Dubletten darf keine Kandidaten-ID stillschweigend überschrieben oder neu nummeriert werden.

Bis dafür eine fachliche Regel beschlossen ist:

> Keine automatische ID-Zusammenführung und keine automatische Löschung historischer Kandidaten-IDs.

---

# Berufs-Merge

Wenn Berufe zusammengeführt werden, ändert sich nur die Berufszuordnung.

Beispiel:

```text
Kandidat 12345
Beruf alt: KFZ-Mechaniker
Beruf neu: Automechaniker
```

Kandidaten-ID bleibt:

```text
12345 → 12345
```

---

# Architekturprinzip

```text
Berufe
Status
Skills
Stellen
Kunden
Bewerbungen
Notizen
Kommunikation
Dokumente
Module

→ dürfen sich ändern

Kandidaten-ID
→ darf sich niemals ändern
```
