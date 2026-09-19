# JSI CRM Update – Documentation Index

Stand: 2026-09-19

Dieses Repository enthält ab jetzt die Dokumentation für die neue CRM-/Datenarchitektur. Das alte CRM-Repository dient nur noch als Legacy-Quelle zur Analyse.

## Dokumente

- `00_MASTER_SUMMARY.md` – fachliche Leitplanken und Gesamtbild
- `01_LEGACY_SCHEMA_AND_FINDINGS.md` – bestätigte Legacy-Strukturen und Analysebefunde
- `02_TARGET_ARCHITECTURE_AND_MIGRATION.md` – Zielarchitektur und Migrationsprinzipien
- `03_CANDIDATE_ID_IMMUTABILITY.md` – unveränderliche Kandidaten-ID und Schutzregeln
- `04_CANDIDATE_ENTRY_CHANNELS.md` – neue Kandidaten-Eingangswege und Audit statt festem Herkunftsfeld

## Wichtige Sprachregel

Für echte Berufe verwenden wir in der fachlichen Dokumentation nur:

- Beruf
- Berufs-ID
- Berufsbezeichnung

Begriffe wie `occupation` / `occupation_id` sollen in der fachlichen Zielbeschreibung nicht mehr verwendet werden, damit echte Berufe nicht mit künstlichen Stellen-/Suchprofilen verwechselt werden.

## Grundprinzip

Das Legacy-CRM wird nicht 1:1 nachgebaut. Jeder Bereich wird mit `BEHALTEN`, `ÄNDERN` oder `ENTFERNEN` bewertet.
