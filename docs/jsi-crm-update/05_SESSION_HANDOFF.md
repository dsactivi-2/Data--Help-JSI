# JSI CRM Update – Handoff für neue Session

Stand: 2026-09-19

## 1. Ziel des Projekts

Das bestehende Legacy-CRM soll **nicht 1:1 nachgebaut** werden.

Ziel ist eine neue, deutlich einfachere, saubere und skalierbare CRM-Architektur für:

- Kandidaten
- Kunden
- Stellen
- Berufe
- Skills
- Ausbildung
- Berufserfahrung
- Sprachen
- Suche und Filter
- Notizen
- Gesprächsverläufe
- Dokumente
- MCPs
- neue UI
- spätere Automationen

Grundprinzip:

> Eine Person = ein Kandidat = eine individuelle Kandidaten-ID.

Keine Kopien derselben Person in unterschiedlichen Modulen oder Datenbanken.

---

# 2. Repository-Struktur

## Legacy-CRM

Nur noch zur Analyse und Rekonstruktion alter Abläufe:

`dsactivi-2/Cursor-old-Crm_Refactor`

Dieses Repo soll NICHT mit neuer Architektur-Dokumentation vermischt werden.

## Neue CRM-/Datenarchitektur

Neue Dokumentation liegt hier:

`dsactivi-2/Data--Help-JSI`

Ordner:

`docs/jsi-crm-update/`

Dort liegen aktuell:

- `README.md`
- `00_MASTER_SUMMARY.md`
- `01_LEGACY_SCHEMA_AND_FINDINGS.md`
- `02_TARGET_ARCHITECTURE_AND_MIGRATION.md`
- `03_CANDIDATE_ID_IMMUTABILITY.md`
- `04_CANDIDATE_ENTRY_CHANNELS.md`
- `05_SESSION_HANDOFF.md`

Alle neuen Architekturentscheidungen sollen ab jetzt dort dokumentiert werden.

---

# 3. Wichtigste Regel: Kandidaten-ID

Jeder Kandidat bekommt eine eigene, eindeutige und fortlaufende Kandidaten-ID.

Beispiel:

```text
Kandidat A → 100001
Kandidat B → 100002
Kandidat C → 100003
```

Diese Kandidaten-ID:

- ist für jeden Kandidaten individuell
- wird fortlaufend vergeben
- bleibt während der gesamten Lebenszeit des Kandidaten im System gleich
- darf nie geändert werden
- darf bei Migration nicht neu nummeriert werden
- darf durch Berufs-Merge nicht geändert werden
- darf durch Statusänderungen nicht geändert werden
- darf durch Job-/Kunden-/Projektzuordnung nicht geändert werden
- darf durch MCP oder UI nicht geändert werden

Die ID soll technisch geschützt werden:

- Primärschlüssel / Unique
- nicht editierbar in der UI
- keine MCP-/API-Funktion zum Ändern
- DB-Schutz gegen UPDATE der Kandidaten-ID
- Migration prüft `alte ID = neue ID`

---

# 4. Kandidaten können über mehrere Wege ins System kommen

Neue Kandidaten können z. B. angelegt werden durch:

- direkten manuellen Eintrag
- Website
- Webformular
- speziellen Link / Landingpage
- API
- Import
- später eventuell weitere Kanäle

Der Eintragsweg soll nicht zwangsläufig ein wichtiges festes Kandidatenfeld sein.

Besser als Audit-/Event-Information, z. B.:

```text
candidate_created_event
- candidate_id
- created_at
- created_by
- channel
- optional form_id
- optional campaign_id
```

Damit bleibt nachvollziehbar, wie jemand ins System kam, ohne das Kernprofil unnötig aufzublähen.

---

# 5. Land / Herkunftsland bleibt notwendig

Land bzw. Herkunftsland darf NICHT entfernt werden.

Es wird fachlich benötigt, weil davon unter anderem unterschiedliche Prozesse abhängen.

Beispiele:

```text
EU
Bosnien und Herzegowina
Serbien
andere Drittstaaten
```

Die Logik kann unterschiedlich sein:

- EU-Kandidat → möglicherweise kein Visum notwendig
- Kandidat aus Bosnien und Herzegowina → ggf. Visa-/Botschaftsprozess über Sarajevo
- Kandidat aus Serbien → ggf. Botschaft / Prozess über Belgrad
- andere Herkunftsländer → ggf. andere Botschaft, Visum- oder Anerkennungslogik

Land darf deshalb nicht nur als dekorative Adresse betrachtet werden.

Es kann relevant sein für:

- Visa-Prozess
- Botschaft
- Arbeitsgenehmigung
- Anerkennung
- Recruiting-Prozess
- Dokumentanforderungen
- Filter
- Kundenanforderungen

Zu prüfen bzw. sauber zu trennen:

```text
nationality
residence_country
country_of_origin
```

Diese Werte dürfen nicht automatisch gleichgesetzt werden.

Beispiel:

```text
nationality = Bosnia and Herzegovina
residence_country = Germany
```

Das kann andere Prozesse auslösen als:

```text
nationality = Bosnia and Herzegovina
residence_country = Bosnia and Herzegovina
```

Die genaue Visa-/Botschaftslogik muss später fachlich gemeinsam besprochen werden.

---

# 6. Datenbank-Standardsprache

Neue Datenbank:

> Englisch als Standardsprache.

Das gilt für:

- Tabellen
- Spalten
- technische Codes
- kanonische Systemwerte

Die UI wird übersetzt.

Zielsprachen:

- Deutsch
- Englisch
- Bosnisch
- Serbisch

Bei Serbisch ggf.:

- Latein
- Kyrillisch

Wichtig:

Der Benutzer darf in seiner Sprache suchen.

Der technische Kern arbeitet aber mit festen IDs/Codes.

---

# 7. Fachliche Benennung: Beruf, nicht „Occupation“

In der fachlichen Dokumentation soll NICHT mehr mit `occupation` gearbeitet werden.

Verwenden:

- Beruf
- Berufs-ID
- Berufsbezeichnung

Warum:

Es gibt im Legacy-System echte Berufe und künstlich erzeugte Sammel-/Suchberufe.

Das darf nicht vermischt werden.

---

# 8. Echter Beruf vs. Stellen-/Suchprofil

Beispiel:

Ein Kunde sucht Mitarbeiter, die Internetanschlüsse installieren.

Im alten CRM wurde teilweise ein künstlicher „Beruf“ wie:

```text
Internet-Monteur
```

angelegt.

Darunter wurden verschiedene echte Berufe zusammengefasst:

- Elektriker
- Installateur
- Mechatroniker
- andere handwerklich geeignete Personen

Im neuen System soll das getrennt werden.

## Echter Beruf

Beispiele:

```text
Automechaniker
Elektriker
Installateur
Mechatroniker
Schweißer
```

## Stellen-/Suchprofil

Beispiel:

```text
Internet-Monteur
```

Das ist keine zwingend anerkannte Ausbildung.

Dazu können passende Berufe hinterlegt werden:

```text
Stellenprofil: Internet-Monteur

Passende Berufe:
- Elektriker
- Installateur
- Mechatroniker
- Telekommunikationstechniker
```

Zusätzlich:

```text
Skills:
- Bohren
- Kabelverlegung
- Routerinstallation
- Kundenkontakt
```

Diese beiden Ebenen dürfen nicht vermischt werden.

---

# 9. Berufs-Chaos im Legacy-System

Über 10–12 Jahre wurden Berufsbezeichnungen teilweise frei eingetragen.

Dadurch existieren für denselben Beruf:

- verschiedene Schreibweisen
- Tippfehler
- Abkürzungen
- unterschiedliche Sprachen
- Synonyme
- teilweise völlig andere Bezeichnungen

Beispiel:

```text
Automehanicar
Auto mehanicar
Automehaničar
KFZ Mechaniker
Kfz-Mechaniker
Automechaniker
```

Ziel:

> Ein echter Beruf existiert im neuen System genau einmal.

Beispiel:

```text
Beruf: Automechaniker
Berufs-ID: 471
```

Alle Kandidaten, Ausbildungseinträge und Berufserfahrungen, die fachlich dazu gehören, werden auf denselben Beruf gemappt.

---

# 10. Wichtig: Nicht nur die Berufsliste mergen

Beim Merge muss nicht nur der Name der Berufsliste geändert werden.

Alle Kandidaten-Zuordnungen müssen aktualisiert werden.

Beispiel:

```text
Kandidat A → alte Berufsvariante 1
Kandidat B → alte Berufsvariante 2
Kandidat C → alte Berufsvariante 3
```

Nach Bereinigung:

```text
Kandidat A → Beruf Automechaniker
Kandidat B → Beruf Automechaniker
Kandidat C → Beruf Automechaniker
```

Kandidaten-ID bleibt selbstverständlich unverändert.

---

# 11. Berufszuordnung darf nicht nur im Haupt-CRM gesucht werden

Berufsinformationen können in verschiedenen Bereichen stehen:

- Haupt-Kandidatenprofil
- Ausbildung / `idk_kandidat_edukacija`
- Berufserfahrung / `idk_kandidat_radno_iskustvo`
- Diploma-/Anerkennungsbereich
- Dippel / NP
- Job-/Positions-Mappings
- weitere Legacy-Bereiche
- Freitext

Beispiel:

```text
Hauptprofil:
kein Automechaniker eingetragen

Ausbildung:
Automehaničar

Berufserfahrung:
KFZ servis, 8 Jahre

Dippel/NP:
Automechaniker
```

Diese Person darf nicht übersehen werden.

Darum:

> Alle relevanten Bereiche gleichzeitig scannen.

---

# 12. Ausbildung und Berufserfahrung getrennt speichern

Beispiel Kandidat:

```text
Beruf:
Automechaniker

Ausbildung:
Automechaniker

Arbeitserfahrung:
Automechaniker – 8 Jahre
```

Diese drei Informationen sind nicht dasselbe.

Zielstruktur fachlich:

```text
candidate
candidate_professions
candidate_education
candidate_work_experience
```

Alle können auf denselben Beruf verweisen.

---

# 13. Strukturierte Eingaben statt Freitext

Alles, was später gesucht oder gefiltert werden soll, soll möglichst NICHT frei eingetippt werden.

Stattdessen:

- Dropdown
- Multi-Select
- Checkboxen
- Choice
- Toggle
- definierte Listen

Freitext nur für:

- Notizen
- ergänzende Beschreibung
- Gesprächszusammenfassung
- besondere Details

Beispiel Berufserfahrung:

```text
Beruf:
[ Automechaniker ]

Tätigkeitsbereiche:
[x] Motor
[x] Bremsen
[x] Diagnose
[ ] Karosserie

Erfahrung:
[ 5–10 Jahre ]

Land:
[ Deutschland ]

Niveau:
[ selbstständig ]

Zusatzdetails:
[ Freitext optional ]
```

---

# 14. Kunden und Stellen

Ein Kunde kann mehrere Stellen / Suchaufträge haben.

Beispiel:

```text
Kunde: XY GmbH
Stelle: Internet-Monteur
Bedarf: 10 Mitarbeiter
```

Dafür soll ein Filterprofil gespeichert werden.

Beispiel:

```text
Passende Berufe:
- Elektriker
- Installateur
- Mechatroniker

Skills:
- Bohren
- Kabel
- Router

Sprache:
Deutsch B1

Führerschein:
B
```

---

# 15. Gespeicherte Filterprofile

Ein Filterprofil ist nur eine gespeicherte Suchkonfiguration.

Es bedeutet NICHT:

- Kandidat ist reserviert
- Kandidat ist fest für diesen Kunden vorgesehen
- Kandidat gehört nur zu diesem Kunden

Beispiel:

```text
Kunde XY
→ Stelle Internet-Monteur
→ Suchprofil laden
→ passende Filter automatisch setzen
```

Danach können flexible Filter ergänzt werden.

Beispiel:

```text
Standardprofil:
Elektriker + Installateur

Heute zusätzlich:
Wohnort München
```

Das Standardprofil bleibt unverändert, sofern nicht bewusst gespeichert wird.

---

# 16. „Reserviert“ nicht automatisch übernehmen

Das alte CRM hat einen Status bzw. Prozess „reserviert“.

Dieser soll aktuell NICHT automatisch in die neue Struktur übernommen werden.

Grundregel:

> Erst Datenbankstruktur besprechen, danach Statusmodell.

Danach entscheiden:

- behalten
- ändern
- entfernen

Sehr viele alte Status werden wahrscheinlich komplett wegfallen.

---

# 17. Statussystem komplett neu aufbauen

Das Legacy-CRM hat mehrere Statussysteme vermischt:

- `kandidat_status`
- `kandidat_status_prijave`
- `kandidat_status_messenger`
- `kandidat_tf_status`
- weitere Prozessstatus

Diese sollen nicht 1:1 übernommen werden.

Erst fachlich klären:

```text
Was beschreibt dieser Status?
Zu welchem Objekt gehört er?
Brauchen wir ihn noch?
```

Beispiel:

Ein Bewerbungsstatus gehört eher zu:

```text
application
```

und nicht zwingend direkt zu:

```text
candidate
```

---

# 18. Vector Search / semantische Suche

Zusätzlich zur normalen Filterung soll Vector Search eingesetzt werden.

Ideal für:

- Notizen
- Gesprächsverläufe
- Call Summaries
- CV-Texte
- Dokumenttexte
- historische Freitextfelder

Beispiel Suche:

> Zeige mir Kandidaten, die gesagt haben, dass sie erst Deutsch lernen und später nach Deutschland möchten.

Vector Search kann semantisch ähnliche Inhalte finden.

Aber:

> Vector Search ersetzt keine strukturierten Filter.

Ideal:

```text
Beruf = Elektriker
Deutsch >= A2
+
semantische Suche:
"später offen für Deutschland"
```

---

# 19. MCP-Architektur

Geplant sind unterschiedliche MCP-Zugriffe:

## Interner Read-MCP

Nur lesen.

## Interner kontrollierter Write-MCP

Schreiben nur über definierte Aktionen und ggf. Freigaben.

## Interner Full-Access-MCP

Nur intern, per UI ein-/ausschaltbar.

## Kunden-MCP

Sehr eingeschränkt.

Kunde sieht nur erlaubte Felder.

## Kandidaten-MCP

Für Kandidatenzugriff / Self-Service.

Wichtig:

> Sicherheit muss serverseitig erzwungen werden.

Nicht nur im Prompt.

---

# 20. MCP-Sprache

Der User kann auf Deutsch, Bosnisch, Englisch oder Serbisch suchen.

Technisch wird die Suchanfrage auf strukturierte Werte aufgelöst.

Beispiel:

```text
User:
"Suche Automechaniker"

→ Beruf Automechaniker
→ Berufs-ID
→ strukturierte DB-Abfrage
```

Nicht direkt tausende Schreibvarianten durchsuchen.

---

# 21. Bekannte Legacy-Daten

Supabase Projekt:

```text
JSI Base
```

Schema:

```text
crm
```

Wichtige Tabellen / Mengen:

```text
crm.idk_kandidati                  122,004
crm.idk_nd_kandidata               81,330
crm.idk_dak_kandidati                 670
crm.idk_kandidat_edukacija        149,473
crm.idk_kandidat_jezici            82,474
crm.idk_kandidat_radno_iskustvo    87,844
crm.idk_documents                  23,614
crm.idk_logs                    ~2,997,184
crm.idk_nd_kandidata_biljeske     928,882
crm.idk_task_force                 452,568
crm.idk_project_kandidati          207,679
```

---

# 22. Wichtige ND-Korrektur

`idk_nd_kandidata` darf NICHT automatisch mit „Nedostupan“ gleichgesetzt werden.

Repo-Analyse deutet darauf hin:

```text
idk_nd_kandidata
```

hängt eher mit:

- Diploma
- Anerkennung
- Nostrifikation

zusammen.

Tatsächliche „nicht erreichbar“-Logik scheint eher zu verwenden:

```text
kandidat_nedostupan
idk_nedostupan_log
tf_brojac_neuspjela_komunikacija
```

Die genaue Schwelle / Logik ist noch zu verifizieren.

---

# 23. Taskforce

Legacy:

```text
crm.idk_task_force
```

ca. 452.568 Datensätze.

Dort sind vermischt:

- Kandidat
- Agent
- Auftrag
- Projekt
- Casting
- Status
- Termin
- Notiz
- Fehlversuche
- Dateien
- Typ

Taskforce soll nicht automatisch kopiert werden.

Erst fachlich rekonstruieren.

---

# 24. Messenger

Legacy enthält:

```text
kandidat_status_messenger
```

und mehrere Messenger-/Bot-Prozesse.

Auch das wird nicht automatisch übernommen.

Erst entscheiden:

- wird Messenger überhaupt noch separat benötigt?
- ist es nur ein Kommunikationskanal?
- braucht es eigene Status?
- kann vieles entfernt werden?

---

# 25. Notizen

Legacy:

```text
idk_notes
```

und weitere große Notiztabellen.

Im neuen System sollen Notizen zentral und sauber an Kandidaten / Prozesse gebunden werden.

Freitext bleibt erlaubt.

Zusätzlich Vector Search.

---

# 26. Kandidatensuche

Neue Suche soll zwei Ebenen haben:

## Strukturierte Filter

- Beruf
- Skills
- Ausbildung
- Berufserfahrung
- Sprache
- Sprachniveau
- Land
- Wohnort
- Nationalität
- Führerschein
- Verfügbarkeit
- relevante Status
- Kunde
- Stelle
- gespeichertes Suchprofil

## Semantische Suche

- Notizen
- Gespräche
- CV
- Dokumente
- Freitext

---

# 27. Alte Suchprobleme

Legacy-Funktion `crm_search_candidates` hatte Typ-Probleme.

Beispiel:

```text
occupation_terms als String
```

konnte dazu führen, dass der Filter praktisch ignoriert wurde.

Im neuen MCP / API:

> falsche Typen müssen einen Fehler erzeugen.

Keine stillschweigend ungefilterte Suche.

---

# 28. Migrationsprinzip

Nicht sofort Tabellen kopieren.

Reihenfolge:

```text
1. Legacy-Workflow verstehen
2. Datenbankstruktur besprechen
3. Statusmodell besprechen
4. entscheiden: behalten / ändern / entfernen
5. Berufs-/Skill-/Filtermodell definieren
6. neue Zielstruktur bauen
7. Read-only Migration Dry Run
8. Berufs- und Daten-Mappings prüfen
9. unsichere Fälle manuell prüfen
10. Daten migrieren
11. Kandidaten-ID prüfen
12. Relationen prüfen
13. Suche / MCP bauen
14. Vector Search
15. neue UI
```

---

# 29. Entscheidungsprinzip für jeden Legacy-Bereich

Für jedes Modul:

```text
Was macht es heute?
↓
Warum gibt es das?
↓
Brauchen wir das noch?
↓
BEHALTEN / ÄNDERN / ENTFERNEN
```

Nicht fragen:

> Wie bauen wir das alte System nach?

Sondern:

> Was brauchen wir heute wirklich?

---

# 30. Was noch offen ist

Als Nächstes fachlich besprechen:

1. endgültige Kandidaten-Grunddaten
2. Land / Nationalität / Wohnsitz / Visa-Logik
3. komplettes neues Statusmodell
4. welche alten Status entfallen
5. Taskforce
6. „reserviert“
7. Messenger
8. Diploma / Anerkennung
9. Dippel / NP
10. Kundenstruktur
11. Stellenstruktur
12. Berufe
13. Skills
14. Ausbildung
15. Berufserfahrung
16. Dokumente
17. Notizen
18. Kommunikationshistorie
19. Vector Search
20. MCP-Rechte
21. neue UI

---

# 31. Wichtig für die nächste Session

Nicht vorschnell Datenmodell oder Statuswerte implementieren.

Der User möchte zuerst:

> Datenbankstruktur gemeinsam Schritt für Schritt besprechen.

Danach:

> Statussystem vollständig neu besprechen.

Sehr viel Legacy-Logik wird wahrscheinlich entfernt.

Bei Unklarheiten lieber erst Repo / DB verifizieren, statt alte Namen zu interpretieren.

Neue Architektur-Dokumentation ausschließlich im Repo:

`dsactivi-2/Data--Help-JSI`

Legacy-Code nur im Repo:

`dsactivi-2/Cursor-old-Crm_Refactor`
