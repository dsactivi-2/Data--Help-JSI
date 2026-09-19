# JSI CRM Update – Target Architecture & Migration Plan

Stand: 2026-09-19

Dieses Dokument beschreibt den aktuellen Arbeitsentwurf für das neue CRM. Es ist noch nicht final und wird nach Status- und Workflow-Besprechung weiter reduziert.

---

# 1. Architekturprinzip

```text
New UI
   │
API / Service Layer
   │
MCP Servers
   │
PostgreSQL
   │
Structured CRM data + Vector search
   │
Audit / History / Automation
```

Grundregel:

> Eine Person = ein Kandidat = eine individuelle, fortlaufende Kandidaten-ID.

Keine Kopien derselben Person je Modul oder Prozess.

---

# 2. Kandidatenkern

Nur wirklich kandidatenbezogene Kerndaten gehören direkt an den Kandidaten.

Arbeitsentwurf:

```text
candidates
----------
candidate_id
first_name
last_name
date_of_birth
created_at
updated_at
```

Weitere Daten über Relationen:

```text
candidate_contacts
candidate_addresses
candidate_jobs_or_professions
candidate_skills
candidate_languages
candidate_education
candidate_work_experience
candidate_documents
candidate_notes
candidate_communications
```

Die Kandidaten-ID ist unveränderlich.

---

# 3. Berufe

Im fachlichen Modell verwenden wir nur die Begriffe:

- Beruf
- Berufs-ID
- Berufsbezeichnung

Ein echter Beruf wird genau einmal kanonisch geführt.

Beispiel:

```text
Berufs-ID: 471
Berufsbezeichnung EN: Automotive Mechanic
```

Historische Varianten wie `Automehanicar`, `KFZ Mechaniker`, `Auto mehanicar` usw. werden bei der Migration zusammengeführt.

Wichtig:

> Ein Berufs-Merge ändert niemals die Kandidaten-ID.

---

# 4. Ausbildung

Arbeitsentwurf:

```text
candidate_education
-------------------
id
candidate_id
profession_id nullable
qualification_id nullable
institution
country_code
start_date
end_date
graduated
raw_legacy_text nullable
notes nullable
```

Wenn eine Ausbildung fachlich auf einen Beruf zeigt, soll sie dieselbe zentrale Berufs-ID verwenden.

---

# 5. Berufserfahrung

Arbeitsentwurf:

```text
candidate_work_experience
-------------------------
id
candidate_id
profession_id nullable
company_name
country_code
start_date
end_date
experience_months nullable
level_code nullable
raw_legacy_text nullable
details nullable
```

Zusätzliche Skills können separat verknüpft werden.

So kann ein Kandidat mehrere Berufserfahrungen strukturiert besitzen.

---

# 6. Skills

Arbeitsentwurf:

```text
skills
------
id
code
name_en
category_id nullable
active
```

```text
candidate_skills
----------------
candidate_id
skill_id
level_code nullable
verified
```

Beispiele:

```text
Cable installation
Drilling
Router installation
Welding
PLC programming
Customer service
```

---

# 7. Sprachen

```text
languages
---------
id
code
name_en
```

```text
candidate_languages
-------------------
candidate_id
language_id
level_code
verified
```

Sprachniveau als kontrollierter Wert, z. B.:

```text
A1 A2 B1 B2 C1 C2 NATIVE
```

---

# 8. Kunden und Stellen

```text
customers
---------
id
name
status_code
created_at
updated_at
```

```text
jobs
----
id
customer_id
title_en
description
headcount_requested
status_code
created_at
updated_at
```

Eine Stelle ist nicht automatisch ein echter Beruf.

---

# 9. Stellen-/Suchprofile

Für Rollen wie `Internet-Monteur`, die als kundenspezifische oder praktische Suchgruppe dienen, wird ein separates Stellen-/Suchprofil geführt.

Beispiel:

```text
Stellenprofil: Internet-Monteur

passende Berufe:
- Elektriker
- Installateur
- Mechatroniker

passende Skills:
- Kabelverlegung
- Bohren
- Routerinstallation
```

Das Profil verbindet mehrere echte Berufe und Skills, ohne selbst automatisch als Beruf behandelt zu werden.

---

# 10. Gespeicherte Suchprofile pro Kunde/Stelle

Ein Kunde kann für eine konkrete Stelle ein wiederverwendbares Filterprofil haben.

Beispiel:

```text
Kunde: XY GmbH
Stelle: Internet-Monteur
Bedarf: 10 Mitarbeiter
```

Gespeicherte Filter:

```text
Berufe
Skills
Sprachniveau
Führerschein
Ausbildung
Erfahrung
weitere feste Kriterien
```

Flexible Filter können bei einer einzelnen Suche temporär ergänzt oder geändert werden.

Ein Suchprofil ist keine Reservierung und keine feste Kandidatenliste.

---

# 11. Kandidatensuche

Die neue Suche wird kombiniert aufgebaut.

## Strukturierte Filter

```text
Berufs-IDs
Skills
Sprache + Niveau
Ausbildung
Berufserfahrung
Land / Ort
Verfügbarkeit
Prozesszustand
Kunde / Stelle / Suchprofil
```

## Semantische Suche

Für unstrukturierte Inhalte:

```text
Notizen
Gesprächszusammenfassungen
CV-Text
Dokumenttext
Legacy-Freitext
```

## Kombination

Beispiel:

```text
Beruf = Elektriker
AND Deutsch >= B1
AND Führerschein = B
AND semantische Suche = "später offen für Deutschland nach Sprachkurs"
```

---

# 12. Vector Search

Vector Search soll für unstrukturierte Daten genutzt werden, nicht als Ersatz für normale Filter.

Geeignete Quellen:

- Notizen
- Gesprächsverläufe
- Call Summaries
- CV-Text
- Dokumenttext
- historische Freitextfelder

Technisch kann PostgreSQL + pgvector genutzt werden.

Nicht blind alles vektorisieren.

---

# 13. Mehrsprachigkeit

Die Datenbank bleibt kanonisch Englisch.

UI-Zielsprachen:

- Deutsch
- Englisch
- Bosnisch
- Serbisch

Serbisch bei Bedarf Lateinisch und Kyrillisch.

Der MCP darf Suchanfragen in der Sprache des Benutzers erhalten, soll intern aber auf stabile IDs/Codes auflösen.

---

# 14. Kandidaten-ID

Jeder Kandidat erhält genau eine individuelle, fortlaufende Kandidaten-ID.

Die ID darf niemals verändert werden durch:

- Migration
- Merge von Berufsbezeichnungen
- Statusänderung
- Modulwechsel
- neue Stelle
- Kundenbeziehung
- API
- MCP
- UI
- Automationen

Die Datenbank muss Änderungen technisch blockieren.

---

# 15. Kandidaten-Eingangswege

Neue Kandidaten können über verschiedene Wege erfasst werden:

- direkt manuell
- Website / Webformular
- Link / Formular
- API
- Import
- Partner / Agent, falls später benötigt

Die Herkunft soll nicht zwingend als dauerhaftes Kandidatenfeld gespeichert werden.

Besser:

```text
candidate_created_event
- candidate_id
- created_at
- created_by
- channel
- form_id optional
- campaign_id optional
```

Damit bleibt nachvollziehbar, wie der Kandidat ins System kam, ohne das Profil dauerhaft mit einem historischen Herkunftsfeld zu belasten.

---

# 16. Statusmodell

Noch bewusst offen.

Nicht alle alten Status gehören direkt an den Kandidaten.

Zu prüfen sind u. a.:

```text
Kandidaten-Lifecycle
Bewerbungsstatus
Kommunikationsstatus
Kunden-/Stellenbeziehung
Messenger-/Channel-Zustand
Diploma-/Anerkennungsprozess
```

Grundregel:

> Ein Status gehört zu dem Objekt, dessen Zustand er beschreibt.

Das alte Statusmodell wird nicht automatisch übernommen.

---

# 17. Kommunikation

Arbeitsentwurf:

```text
candidate_communications
------------------------
id
candidate_id
channel_code
direction_code
occurred_at
employee_id nullable
summary nullable
raw_text nullable
outcome_code nullable
```

Strukturierte Outcomes kontrolliert erfassen.

Gesprächsinhalte können zusätzlich semantisch indexiert werden.

---

# 18. Notizen

```text
candidate_notes
---------------
id
candidate_id
author_user_id
note_type_code nullable
text
created_at
updated_at nullable
```

Notizen bleiben Freitext, sollen aber für semantische Suche indexierbar sein.

---

# 19. Audit

Kritische Änderungen und Migrationen müssen nachvollziehbar sein.

Arbeitsentwurf:

```text
audit_log
---------
id
actor_type
actor_id
action_code
entity_type
entity_id
before_data
after_data
created_at
```

Berufs-Merges zusätzlich separat protokollieren:

- alter Beruf
- neuer Beruf
- betroffene Kandidaten
- betroffene Ausbildungen
- betroffene Berufserfahrungen
- betroffene Suchprofile
- Freigabe
- Ausführungszeitpunkt

---

# 20. Berufsbereinigung / Merge-Pipeline

## Phase A – Discovery

Alle Berufsquellen scannen:

```text
Hauptprofil
Ausbildung
Berufserfahrung
Diploma/Anerkennung
Dippel/NP
Job-Mappings
Projekt-Mappings
relevante Legacy-Freitexte
```

## Phase B – Begriffe sammeln und normalisieren

Nur zur Analyse:

```text
lowercase
trim
unicode normalization
diacritics handling
punctuation cleanup
```

Der normalisierte Text ist nicht automatisch der neue Beruf.

## Phase C – Semantisch gruppieren

Signale:

- Textähnlichkeit
- mehrsprachige Bedeutung
- Ausbildung
- Berufserfahrung
- Skills
- Projekt-/Stellenkontext
- historische gemeinsame Verwendung

## Phase D – Einen echten Zielberuf bestimmen

Je fachlicher Bedeutung genau einen Beruf festlegen.

## Phase E – Unsichere Fälle prüfen

Automatisch nur bei hoher Sicherheit.

## Phase F – Alle Relationen umstellen

Nicht nur Berufsbezeichnungen umbenennen.

Alle betroffenen Kandidaten-, Ausbildungs-, Berufserfahrungs-, Job- und Suchprofilreferenzen auf den neuen Beruf umstellen.

## Phase G – Referenzen prüfen

Vor Cleanup:

```text
alte Berufsreferenzen = 0
```

## Phase H – Legacy-Mapping entfernen/archivieren

Nur wenn nicht mehr produktiv benötigt.

---

# 21. Personen-Dubletten sind ein separates Thema

Berufsbereinigung und Kandidaten-Deduplizierung dürfen nicht vermischt werden.

Die Kandidaten-ID darf bei Dubletten nicht automatisch überschrieben oder neu nummeriert werden.

Eine fachliche Regel für echte Dubletten wird separat beschlossen.

---

# 22. MCPs

Geplante Trennung:

```text
MCP Internal Read
MCP Internal Controlled Write
MCP Internal Full Access (UI-controlled)
MCP Customer
MCP Candidate
```

Jeder MCP verwendet serverseitige Policies für:

- erlaubte Tools
- erlaubte Felder
- erlaubte Entitäten
- Schreibaktionen
- Freigaben
- Scope

Keine sensible Policy nur im Prompt.

---

# 23. API-/MCP-Schema-Validierung

Alle Tool-Parameter strikt typisieren.

Falsche Typen müssen Fehler erzeugen und dürfen niemals stillschweigend zu einer ungefilterten Suche führen.

---

# 24. Legacy-to-Target Arbeitsmatrix

| Legacy | Bedeutung | Neues Ziel | Entscheidung |
|---|---|---|---|
| `idk_kandidati` | Kandidatenkern + vermischte Zustände | Kandidatenkern + Relationen | prüfen |
| `idk_kandidat_edukacija` | Ausbildung | strukturierte Ausbildung | behalten/bereinigen |
| `idk_kandidat_radno_iskustvo` | Arbeitserfahrung | strukturierte Berufserfahrung | behalten/bereinigen |
| `idk_kandidat_jezici` | Sprachen | strukturierte Sprachen | behalten/bereinigen |
| `idk_task_force` | Queue/Kommunikation/Projektmix | neu definieren | stark prüfen |
| `kandidat_status` | Legacy-Kandidatenstatus | neues Statusmodell | neu bauen |
| `kandidat_status_prijave` | Bewerbung/Projekt/Visum gemischt | pro Prozessobjekt | neu bauen |
| `kandidat_status_messenger` | Messengerprozess | Kommunikation/Channel | prüfen |
| `kandidat_nedostupan` | Erreichbarkeit | Kommunikationszustand | prüfen |
| `idk_nd_kandidata` | vermutlich Diploma/Anerkennung | eigenes Modul | verifizieren |
| `idk_notes` | Notizen | zentrale Notes | behalten/mappen |
| Cron-Strukturen | historische Automationen | neue Rules/Workers | nicht kopieren |

---

# 25. Reihenfolge der Umsetzung

```text
1. Legacy-Workflow analysieren
2. Datenbank-Domänen finalisieren
3. Statussystem gemeinsam neu definieren
4. Filterfelder finalisieren
5. Berufs-/Skill-Struktur bereinigen
6. Ziel-Schema erstellen
7. Read-only Migration Dry Run
8. Merge-Reports erzeugen
9. Unsichere Zuordnungen prüfen
10. Datenmigration
11. Referenzen vollständig verifizieren
12. Search API / MCP
13. Vector Search
14. Neue UI
15. Controlled Cutover
```

---

# 26. Noch nicht vorschnell übernehmen

Bis zur fachlichen Besprechung nicht automatisch übernehmen:

- alte Reservierungslogik
- Taskforce-Status
- Messenger-Status
- alte Cron-Workflows
- alte Kandidaten-Lifecycle-Werte
- historische Berufsvarianten
- alle Legacy-Tabellen

Die neue Struktur soll kleiner und klarer werden.
