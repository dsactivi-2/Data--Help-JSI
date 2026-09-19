# Candidate Entry Channels – neue Kandidaten erfassen

Stand: 2026-09-19

## Grundsatz

Neue Kandidaten können über mehrere Wege ins System kommen.

Beispiele:

- direkte manuelle Erfassung durch Mitarbeiter
- Website-/Webformular
- Link-/Formulareintrag
- API
- Import
- Partner-/Agenten-Eintrag, falls später benötigt

Jeder neu angelegte Kandidat erhält genau eine neue individuelle, fortlaufende Kandidaten-ID.

Die Art des Eintrags verändert die Kandidaten-ID nicht.

---

# Herkunft nicht als dauerhaftes Kernfeld

Historisch wurde die Herkunft genutzt, um nachzuverfolgen, woher ein Kandidat kam.

Für das neue System ist ein dauerhaftes Feld wie:

```text
candidate.source = website
```

wahrscheinlich nicht nötig.

Besser ist ein Erstell-/Audit-Ereignis.

Arbeitsentwurf:

```text
candidate_created_event
-----------------------
id
candidate_id
created_at
created_by_user_id nullable
channel_code
form_id nullable
campaign_id nullable
request_id nullable
metadata nullable
```

Mögliche `channel_code`-Werte:

```text
MANUAL
WEBSITE
FORM_LINK
API
IMPORT
PARTNER
AGENT
```

---

# Warum Event statt festem Herkunftsfeld?

Ein Kandidat kann später über mehrere Wege erneut mit dem System interagieren.

Beispiel:

```text
2026-01-10 → Website-Anmeldung
2026-03-01 → Mitarbeiter aktualisiert Profil
2026-06-15 → Kandidat öffnet neuen Bewerbungslink
```

Ein einzelnes Feld `source = website` bildet diese Historie nicht sauber ab.

Ein Event-/Audit-Verlauf schon.

---

# Was bleibt im Kandidatenkern?

Im Kandidatenkern bleibt nur die dauerhafte Identität und echte Stammdaten.

Zum Beispiel:

```text
candidate_id
first_name
last_name
date_of_birth
created_at
updated_at
```

Der technische Erstellweg wird separat protokolliert.

---

# Sicherheitsregel

Egal über welchen Kanal der Kandidat angelegt wird:

1. neue Kandidaten-ID genau einmal erzeugen
2. Kandidaten-ID danach unveränderlich sperren
3. Erstellkanal separat protokollieren
4. spätere Aktionen nur an dieselbe Kandidaten-ID anhängen

---

# Noch offen

Vor finaler Umsetzung prüfen:

- welche Eintragskanäle tatsächlich benötigt werden
- ob Kampagnen-/Formulartracking fachlich ausgewertet werden soll
- ob `created_by` für Website/API-Systemkonten benötigt wird
- wie Dublettenprüfung beim Neueintrag abläuft
- wann ein neuer Datensatz angelegt und wann ein bestehender Kandidat aktualisiert wird
