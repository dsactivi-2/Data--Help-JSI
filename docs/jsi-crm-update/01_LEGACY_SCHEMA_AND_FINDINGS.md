# JSI CRM Update – Legacy Schema & Findings

Stand: 2026-09-19

Dieses Dokument fasst bisher bestätigte Legacy-CRM-Strukturen und Analysebefunde zusammen. Bestätigte Fakten und noch zu verifizierende Punkte werden getrennt behandelt.

## Repository-Quelle

Legacy-Repository:

`dsactivi-2/Cursor-old-Crm_Refactor`

Wichtige Bereiche:

- `src/crm`
- `src/idktime`
- `src/jobstep-partner`
- `src/join`
- `src/messenger`
- `src/online-viza`
- `src/website`

Große geschäftslogiklastige Dateien:

- `src/crm/ajax.php`
- `src/crm/ajax_data.php`

Die Repo-Analyse dient nur dazu, den tatsächlichen bisherigen Workflow zu rekonstruieren. Das Legacy-Repo ist nicht die Zielarchitektur.

---

# 1. Datenbanklandschaft

## Supabase CRM

Projekt: `JSI Base`

Projekt-Ref: `oohbgrajwxdotalijmih`

Region: `eu-west-1`

Das eigentliche CRM liegt im Schema `crm`.

Bekannte größere Tabellen:

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
idk_nd_cron_export                  7,581
idk_nd_cron_settings                   13
idk_triggerurl                          10
```

Neuere Agent-/MCP-nahe Tabellen:

```text
crm.status_project_name_rules               17
crm.candidate_status_transition_audit        2
crm.candidate_field_fallbacks                 1
crm.candidate_field_change_audit              2
crm.agent_action_audit                        0
crm.candidate_documents                       0
crm.candidate_document_text                   0
crm.candidate_document_embeddings             0
crm.candidate_status_transition_rules         14
crm.occupation                               702
crm.occupation_alias                        1,239
crm.job_occupation_map                     87,995
crm.occupation_remap_audit                     4
crm.pozicija_occupations                    123
```

Hinweis: Diese Legacy-Tabellennamen werden hier als technische Ist-Bezeichnungen dokumentiert. Im neuen fachlichen Modell soll von Beruf / Berufs-ID gesprochen werden.

Viele `crm.*`-Tabellen haben RLS aktiviert. RLS nicht blind ändern oder deaktivieren.

## Neon CRM

Projekt: `CRM`

Project ID: `green-voice-30543034`

Region: `aws-eu-central-1`

PostgreSQL 18

Default Branch: `prod-copy` / `br-steep-sound-b1qnvllq`

Weitere Branches:

- `production`
- `vercel-dev`
- `mcp-dev`

Installierte Extensions:

- `pg_stat_statements 1.12`
- `neon 1.14`

---

# 2. Performance-Befunde

Auffällige Sequential Scans:

```text
search_synonyms                 ~93,825
idk_kandidat_jezici             ~52,661
idk_kandidat_radno_iskustvo     ~29,360
mcp_tokens                       ~6,328
idk_kandidat_edukacija           ~1,325
idk_kandidati                      694
```

Größere Tabellen/Relationen grob:

```text
idk_logs                     ~499 MB
idk_nd_kandidata_biljeske    ~129 MB
idk_nd_cron_export            ~77 MB
idk_task_force                ~67 MB
idk_kandidati                 ~36 MB
idk_notes                     ~33 MB
```

Bekannter PostgreSQL-Trigger:

`kandidat_status_change` auf `idk_kandidati.kandidat_status`, AFTER UPDATE, über `trg_kandidat_status_change()`.

---

# 3. Legacy-Statusverteilungen

## kandidat_status

```text
0      3,999
1      8,432
2      3,804
3      4,446
4     24,545
5        461
6     74,164
7      1,960
8        193
```

Historisch bekannte Bezeichnungen:

```text
2 = Obrađen
3 = Arhiviran
4 = Kontrola
5 = Dopuna
```

Andere Werte müssen noch sauber aus Repo + Lookup-Tabellen verifiziert werden.

## kandidat_status_prijave

```text
NULL  41,297
0        106
1     54,276
2     22,817
3        170
4        660
5        298
6      2,175
7          6
9         43
10       104
12        17
15         1
18        18
27        16
```

## kandidat_tf_status

```text
NULL 120,300
2        171
8        960
12       193
14       206
16       117
18         1
20         1
22        25
25        11
27        14
35         2
36         1
37         1
39         1
```

Diese Werte werden nicht automatisch in das neue Statusmodell übernommen.

---

# 4. Taskforce / Kommunikation / Reservation

Wichtige Legacy-Tabelle: `crm.idk_task_force`

Bekannte Spalten:

```text
tf_id
tf_candidate_id
tf_agent_id
tf_nalog_id
tf_project_id
tf_casting_id
tf_status_id
tf_call_appointment
tf_note
tf_important_note
tf_last_active_task
tf_brojac_neuspjela_komunikacija
tf_files
tf_doe
tf_vrsta_id
```

Unterstützende Tabellen:

```text
idk_tf_agent_nalog
idk_tf_reservations
idk_tf_stats_reservations
idk_tf_statusi
idk_tf_transfered_candidates_logs
idk_tf_vrste
```

Repo-Befund: Taskforce-Queries kombinieren `tf_status_id`, `tf_reserved_agent`, `kandidat_pogresan_broj`, `kandidat_nedostupan` und Reservations-Tabellen.

Interpretation: Taskforce/Reservation ist historisch über mehrere Felder und Tabellen verteilt. Im neuen System wird davon nichts automatisch übernommen.

---

# 5. Wichtige Korrektur zu „ND“

`idk_nd_kandidata` darf nicht automatisch als „Nedostupan“-Tabelle interpretiert werden.

Tatsächliche Nicht-erreichbar-Logik verwendet u. a.:

```text
idk_kandidati.kandidat_nedostupan
idk_nedostupan_log
brojac
status
doe
```

Taskforce-Queues schließen `kandidat_nedostupan = 1` aus.

Noch offen:

- exakter Write-Pfad
- exakte Zählerlogik
- exakte Schwelle für fehlgeschlagene Kontaktversuche
- Zusammenhang zu `tf_brojac_neuspjela_komunikacija`

Die Erinnerung „nach mehr als 5 Fehlversuchen“ ist noch zu verifizieren.

`idk_nd_kandidata` wirkt im Repo dagegen stark diploma-/anerkennungsbezogen, u. a. über:

```text
kandidat_dipl_id
status_nd_kandidata
pstatus_nd_kandidata
idk_nostrifikovane_diplome
full_recognition
```

---

# 6. Messenger als separater historischer Prozess

Es gibt eine eigene Zustandslogik über `kandidat_status_messenger`.

Verwendung u. a. in:

```text
send_dipl.php
ssdata_search.php
cron_bot_instal.php
cron_bot_obrada.php
src/messenger/app/Idk_kandidati.php
serversidedata2.php
vibersms.php
viber_marketing.php
ajax.php
hammer_prijave.php
dak.php
do_dipl.php
```

Legacy-Queries kombinieren teilweise gleichzeitig:

```text
kandidat_status
kandidat_status_prijave
kandidat_status_messenger
```

Das bestätigt die Vermischung unabhängiger Prozesszustände direkt am Kandidaten.

---

# 7. Bewerbung / Projektstatus

`kandidat_status_prijave` wird in vielen Bereichen verwendet.

Im Code sichtbare Zuordnungen:

```text
2  → u_projektu_nr
3  → casting
4  → zaposlen (in einem Reporting-Kontext)
15 → ceka_termin
18 → ceka_vizu
```

Historische Zuordnungen, noch zu verifizieren:

```text
1  Slobodan
2  U projektu NR
3  Casting
4  Završen / ggf. zaposlen in einzelnen Reports
5  Odbijen
6  U projektu RZ
7  Čeka ugovor
8  Poslan ugovor
9  Potpisan ugovor
10 Početak rada
12 Prikupljanje dokumentacije
15 Čeka termin
18 Čeka vizu
21 Dopuna dokumenata
24 Odbijena viza
27 Dobio vizu
```

Nicht als endgültiges neues Statusmodell verwenden.

---

# 8. Candidate Transfer / Projektzuordnung

Repo-Befund aus `candidateTransferToTF/updateSP.php`:

Beim Transfer werden am Kandidaten gleichzeitig u. a. gesetzt:

```text
kandidat_status_prijave = 6
kandidat_nalog_id
kandidat_latest_reserved_time
```

Damit sind Bewerbungsstatus, Auftrag/Projekt und Reservierungszeitpunkt historisch technisch gekoppelt.

---

# 9. Suche / Filter im Legacy-System

Die alte Suche kombiniert u. a.:

- Archivstatus
- falsche Nummer
- nicht erreichbar
- Bewerbungsstatus
- Messengerstatus
- Auftrag/Projekt
- Beruf
- Sprache
- Alter
- Diploma-/Anerkennungsstatus

Die neue Suche soll stattdessen strukturiert und fachlich sauber aufgebaut werden.

---

# 10. Berufs- und Kandidatenquellen

Berufs- und Kandidatendaten liegen nicht nur in einer Tabelle.

Wichtige Quellen:

```text
idk_kandidati
idk_kandidat_edukacija
idk_kandidat_radno_iskustvo
idk_kandidat_jezici
idk_nd_kandidata
idk_dak_kandidati
idk_project_kandidati
occupation
occupation_alias
job_occupation_map
pozicija_occupations
```

Bekannte Verknüpfung:

`idk_kandidati.kandidat_dipl_id → idk_nd_kandidata.id_broj_nd_kandidata`

ca. 77,616 Hauptkandidaten sind darüber verknüpft.

Weitere Rückverknüpfung:

`idk_nd_kandidata.kandidat_idd → Hauptkandidat`

ca. 5,756 Treffer.

Wichtige Konsequenz:

> Beruf eines Kandidaten niemals nur aus dem Hauptdatensatz ableiten. Ausbildung, Berufserfahrung, Diploma/Anerkennung und weitere Quellen gemeinsam auswerten.

---

# 11. Duplicate-/Qualitätsbefunde

Main candidates:

```text
E-Mail gefüllt:        42,523
Duplicate groups:       2,034
excess rows grob:       3,075

Telefon gefüllt:      119,116
Duplicate groups:       3,692
excess rows grob:       4,172

Name + DOB:            73,734
Duplicate groups:       3,309
excess rows grob:       3,700
```

JMBG ist in der Haupttabelle aktuell kein verlässlicher Identitätsschlüssel, weil viele Werte auf `0` normalisieren.

`idk_nd_kandidata`:

```text
E-Mail:                28,636
Duplicate groups:       1,004
excess rows grob:       1,708

Telefon:               80,252
Duplicate groups:       1,698
excess rows grob:       1,933

JMBG:                   6,548
Duplicate groups:          64
excess rows grob:         460
```

Nicht einfach summieren; Kriterien überschneiden sich.

---

# 12. Legacy Cron

Alte Cron-Strukturen sollen nicht übernommen werden:

- `idk_nd_cron_settings`
- `idk_nd_cron_export`
- `idk_triggerurl`

`idk_nd_cron_export`: 7,581 Zeilen, neuester bekannter Stand 2025-07-09.

Historische Regeln:

```text
2 Obrađen   → 15 Tage
3 Arhiviran → 45 Tage
4 Kontrola  → 5 Tage
5 Dopuna    → 10 Tage
```

Nur Referenz, keine Zielarchitektur.

---

# 13. Search-/MCP-Problem

Bekannter Fehler in `crm_search_candidates`:

Die Funktion erwartet Arrays. Bei falschem Typ kann ein Filter stillschweigend ignoriert werden.

Beispiel aus Tests:

```text
"occupation_terms":"Schweißer" → sehr breite Trefferzahl
["Elektriker"]                  → deutlich gefiltert
"Elektriker"                    → wieder sehr breite Trefferzahl
```

Daraus folgt für die neue API/MCP-Schicht:

> Strikte Schematypen und Validierung. Falscher Parametertyp muss Fehler erzeugen, niemals ungefiltert weiterlaufen.

---

# 14. Offene forensische Untersuchung

Es existiert eine separate Fragestellung zu möglichen größeren Kandidaten-Downloads/Exports ungefähr um 2024 ± Wochen/Monate.

Zu prüfen:

- Export-/Download-Endpunkte
- Excel-/CSV-Generatoren
- `idk_logs`
- Auth-/User-Logs
- Web-/Server-Access-Logs, falls vorhanden
- Benutzer-/Berechtigungsänderungen

Keine Behauptung ohne konkrete Log-Belege.

---

# 15. Noch unklare Zahlenliste

Folgende Namen/Zahlen wurden genannt:

```text
Adil Salkicevic        173
Adis Toromanović        75
Benjamin Bender        134
Emir Bender             67
Faris Sabic            207
Sladjana Stoponja      461
```

Die Summe ist 1.117. Die Bedeutung dieser Zahlen ist NICHT geklärt. Sie dürfen nicht als Mitarbeiteranzahl, Downloads, Kandidatenzahl oder andere Aktion interpretiert werden, bis die zugrunde liegende Query/Quelle bekannt ist.
