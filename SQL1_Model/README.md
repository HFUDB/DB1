|                                             |                          |                               |
| ------------------------------------------- | ------------------------ | ----------------------------- |
| **Informatik\*in / Systemtechniker\*in HF** | **Datenbankentwicklung** | ![logo](../x_gitres/logo.png) |

- [1. Vom Relationenmodell zur SQL-Implementierung](#1-vom-relationenmodell-zur-sql-implementierung)
  - [1.1. Überblick: Der Transformationsprozess](#11-überblick-der-transformationsprozess)
    - [1.1.1. Typische Fragen bei diesem Übergang](#111-typische-fragen-bei-diesem-übergang)
  - [1.2. Schritt 1: Entitätsmengen → CREATE TABLE](#12-schritt-1-entitätsmengen--create-table)
    - [1.2.1. Das Basismuster](#121-das-basismuster)
    - [1.2.2. Die wichtigsten Regeln](#122-die-wichtigsten-regeln)
    - [1.2.3. Namenskonventionen (Best Practice)](#123-namenskonventionen-best-practice)
  - [1.3. Schritt 2: Attribute → Spalten mit Datentypen](#13-schritt-2-attribute--spalten-mit-datentypen)
    - [1.3.1. Die Type Affinity Entscheidung](#131-die-type-affinity-entscheidung)
    - [1.3.2. Entscheidungshilfe: Die Beispiel-Datenbank](#132-entscheidungshilfe-die-beispiel-datenbank)
    - [1.3.3. Type Affinity Faustregel](#133-type-affinity-faustregel)
  - [1.4. Schritt 3: Schlüssel definieren (PRIMARY KEY, UNIQUE)](#14-schritt-3-schlüssel-definieren-primary-key-unique)
    - [1.4.1. PRIMARY KEY (Primärschlüssel)](#141-primary-key-primärschlüssel)
    - [1.4.2. UNIQUE (Eindeutigkeit ohne PK)](#142-unique-eindeutigkeit-ohne-pk)
    - [1.4.3. Zusammengesetzter Primärschlüssel (Zwischentabellen)](#143-zusammengesetzter-primärschlüssel-zwischentabellen)
  - [1.5. Schritt 4: Beziehungen → FOREIGN KEY Constraints](#15-schritt-4-beziehungen--foreign-key-constraints)
    - [1.5.1. Das RM-Notation](#151-das-rm-notation)
    - [1.5.2. Die SQL-Umsetzung](#152-die-sql-umsetzung)
    - [1.5.3. 1:1-, 1:n- und m:n-Beziehungen](#153-11--1n--und-mn-beziehungen)
  - [1.6. Mapping-Übersicht: RM-Konzept → SQL-Konstrukt](#16-mapping-übersicht-rm-konzept--sql-konstrukt)
  - [1.7. Vollständiges Beispiel: Von der Bibliotheks-Anforderung zur SQL-DB](#17-vollständiges-beispiel-von-der-bibliotheks-anforderung-zur-sql-db)
    - [1.7.1. Anforderung (Textform)](#171-anforderung-textform)
    - [1.7.2. Relationenmodell (nach Normalisierung)](#172-relationenmodell-nach-normalisierung)
    - [1.7.3. SQL-Implementierung](#173-sql-implementierung)
    - [1.7.4. Test: Daten einfügen](#174-test-daten-einfügen)
- [2. Übungsaufgaben](#2-übungsaufgaben)
  - [2.1. E-Commerce Shop](#21-e-commerce-shop)

---

</br>

# 1. Vom Relationenmodell zur SQL-Implementierung

## 1.1. Überblick: Der Transformationsprozess

Sie haben in den bisherigen Kapiteln folgende Schritte durchlaufen:

```bash
Anforderung (Textform)
        ↓
ERM (Entity-Relationship-Modell) — Konzeptionell, grafisch
        ↓
Relationenmodell (Tabellenübersicht) — Logisch, Tabellen + Schlüssel
        ↓
Normalisierung (1NF, 2NF, 3NF) — Anomalien beseitigt
        ↓
SQL-Datenbank (CREATE TABLE, Constraints) — Physisch, lauffähig
```

**Dieses Kapitel verbindet Schritte 3 und 4:** Wie wird ein normalisiertes Relationenmodell **konkret** in SQLite umgesetzt?

### 1.1.1. Typische Fragen bei diesem Übergang

- "Ich habe eine Tabelle `KUNDEN` mit Attributen `ID`, `Name`, `Email`. Wie schreibe ich die `CREATE TABLE`-Anweisung?"
- "Welcher Datentyp passt zu welchem Attribut? INTEGER oder TEXT?"
- "Wie definiere ich Primary Keys und Foreign Keys in der SQL-Syntax?"
- "Welche Constraints (`NOT NULL`, `UNIQUE`, `CHECK`) sind sinnvoll?"

Diese Fragen beantworten wir im folgenden Kapitel **systematisch** und an vielen Beispielen.

---

## 1.2. Schritt 1: Entitätsmengen → CREATE TABLE

### 1.2.1. Das Basismuster

Im **Relationenmodell** notieren Sie:

```bash
Relation: KUNDEN
───────────────
- ID (PK)
- Name
- Email (UNIQUE)
- Stadt
```

Die **SQL-Umsetzung:**

```sql
CREATE TABLE kunden (
  id    INTEGER,
  name  TEXT,
  email TEXT,
  stadt TEXT
);
```

### 1.2.2. Die wichtigsten Regeln

| **RM-Konzept**               | **SQL-Umsetzung** | **Erklärung**                                    |
| ---------------------------- | ----------------- | ------------------------------------------------ |
| Relation / Entitätsmenge     | `CREATE TABLE`    | 1:1 Abbildung: Eine Relation = eine Tabelle      |
| Relationenname (z.B. KUNDEN) | Tabellenname      | Eindeutig, aussagekräftig, Singular oder Plural  |
| Attribut (z.B. ID, Name)     | Spalte            | Spaltenname = Attributname (lowercase empfohlen) |
| Beziehung                    | `FOREIGN KEY`     | Wird separat definiert (siehe Schritt 4)         |

### 1.2.3. Namenskonventionen (Best Practice)

- **Tabellennamen:** Singular (`kunden`, `buch`) oder Plural (`kunden`, `bücher`) — **konsistent** im Projekt!
- **Spaltennamen:** lowercase, aussagekräftig (`kunde_id` statt `k_id`)
- **Schlüsselspalten:** `id` oder `<tabelle>_id` (z.B. `kunden_id` in Fremdtabellen)

```sql
-- GUT:
CREATE TABLE kunden (
  id        INTEGER PRIMARY KEY,
  vorname   TEXT NOT NULL,
  nachname  TEXT NOT NULL,
  email     TEXT UNIQUE,
  stadt     TEXT
);

-- SCHLECHT (unkonsistent, unklar):
CREATE TABLE K (
  ID_KUNDE INTEGER,
  V TEXT,
  N TEXT,
  E_MAIL TEXT,
  S TEXT
);
```

---

## 1.3. Schritt 2: Attribute → Spalten mit Datentypen

### 1.3.1. Die Type Affinity Entscheidung

Das **schwierigste** Problem beim Übergang ist: **Welcher Datentyp?**

Im Relationenmodell stand:

```bash
Kunde
- ID: Ganzzahl
- Name: Text
- Geburtsdatum: Datum
- Lagerbestand: Dezimalzahl
```

Die Abbildung auf **SQLite Type Affinity**:

| **RM-Attribute**        | **Bedeutung**   | **SQLite-Typ**      | **Beispiel-Wert**        |
| ----------------------- | --------------- | ------------------- | ------------------------ |
| ID, Anzahl              | Ganzzahlen      | `INTEGER`           | `42`, `1000`             |
| Name, Beschreibung      | Text            | `TEXT`              | `'Anna Müller'`, `'Rot'` |
| Geburtsdatum, Kaufdatum | Datum           | `TEXT` (ISO-Format) | `'1990-03-15'`           |
| Preis, Gewicht          | Dezimalzahlen   | `REAL`              | `29.99`, `3.14`          |
| Große Geldbeträge       | Precise Dezimal | `NUMERIC`           | `1234.56`                |
| Aktiv/Passiv            | Boolean         | `INTEGER` (0/1)     | `0` (false), `1` (true)  |
| Bild, Datei             | Binärdaten      | `BLOB`              | (selten in Lernkursen)   |

### 1.3.2. Entscheidungshilfe: Die Beispiel-Datenbank

Für die **Bibliotheksdatenbank** lauten die Entscheidungen:

```sql
CREATE TABLE autoren (
  id          INTEGER,              -- eindeutige ID
  vorname     TEXT,                  -- Namen = Text
  nachname    TEXT,
  land        TEXT,                  -- Länder = Text
  geburtsjahr INTEGER                -- Jahre = Integer
);

CREATE TABLE buecher (
  id            INTEGER,             -- eindeutige ID
  titel         TEXT,                -- Titel = Text
  autor_id      INTEGER,             -- Fremdschlüssel-Referenz = Integer
  genre         TEXT,                -- Genre = Text
  jahr          INTEGER,             -- Veröffentlichungsjahr = Integer
  preis         REAL,                -- Preis kann Dezimalstellen haben
  lagerbestand  INTEGER              -- Menge = Integer
);

CREATE TABLE kunden (
  id    INTEGER,                     -- eindeutige ID
  name  TEXT,                        -- Name = Text
  email TEXT,                        -- Email = Text
  stadt TEXT                         -- Stadt = Text
);

CREATE TABLE ausleihen (
  id            INTEGER,             -- eindeutige ID
  kunden_id     INTEGER,             -- Fremdschlüssel = Integer
  buch_id       INTEGER,             -- Fremdschlüssel = Integer
  ausleihdatum  TEXT,                -- Datum = TEXT im ISO-Format 'YYYY-MM-DD'
  rueckgabe     TEXT                 -- Datum = TEXT im ISO-Format 'YYYY-MM-DD'
);
```

### 1.3.3. Type Affinity Faustregel

| **Frage**                                      | **Typ wählen**                 |
| ---------------------------------------------- | ------------------------------ |
| Wird damit gerechnet (addiert, multipliziert)? | `INTEGER` oder `REAL`          |
| Kann es Dezimalstellen haben?                  | `REAL`                         |
| Ist es eindeutig Text?                         | `TEXT`                         |
| Ist es ein Datum?                              | `TEXT` (Format `'YYYY-MM-DD'`) |
| Ist es ja/nein?                                | `INTEGER` (0 oder 1)           |

---

## 1.4. Schritt 3: Schlüssel definieren (PRIMARY KEY, UNIQUE)

### 1.4.1. PRIMARY KEY (Primärschlüssel)

Im **Relationenmodell** war gekennzeichnet: **ID (PK)**

In **SQL:**

```sql
CREATE TABLE kunden (
  id    INTEGER PRIMARY KEY AUTOINCREMENT,
  name  TEXT NOT NULL,
  email TEXT UNIQUE,
  stadt TEXT
);
```

**Bedeutung:**

- `PRIMARY KEY`: Diese Spalte ist der **eindeutige Identifikator** der Tabelle
- `AUTOINCREMENT`: Der Wert wird **automatisch** hochgezählt (1, 2, 3, ...)
- Jeder Datensatz hat eine **unique ID**, keine Duplikate möglich

### 1.4.2. UNIQUE (Eindeutigkeit ohne PK)

Manchmal sollen auch **andere Spalten** eindeutig sein (z.B. Email):

```sql
CREATE TABLE kunden (
  id    INTEGER PRIMARY KEY AUTOINCREMENT,
  name  TEXT NOT NULL,
  email TEXT UNIQUE,            -- Keine zwei Kunden mit gleicher Email
  stadt TEXT
);
```

**Unterschied:**

- `PRIMARY KEY`: Nur eine pro Tabelle, kann `NULL` nicht enthalten
- `UNIQUE`: Mehrere pro Tabelle möglich, kann `NULL` enthalten (aber nur einmal!)

### 1.4.3. Zusammengesetzter Primärschlüssel (Zwischentabellen)

Bei m:n-Beziehungen braucht man oft einen **Schlüssel aus zwei Spalten**:

```sql
-- Zwischentabelle: Schüler → Kurse (m:n)
CREATE TABLE schueler_kurse (
  schueler_id INTEGER NOT NULL,
  kurs_id     INTEGER NOT NULL,
  einschreib_datum TEXT,
  PRIMARY KEY (schueler_id, kurs_id)  -- Kombinierter Schlüssel!
);
```

**Bedeutung:** Ein Schüler kann sich pro Kurs nur **einmal** einschreiben (kombinierte Eindeutigkeit).

---

## 1.5. Schritt 4: Beziehungen → FOREIGN KEY Constraints

### 1.5.1. Das RM-Notation

Im Relationenmodell:

```bash
Ausleihen
- id (PK)
- kunden_id (FK → KUNDEN)
- buch_id (FK → BUECHER)
- ausleihdatum
- rueckgabe
```

### 1.5.2. Die SQL-Umsetzung

```sql
CREATE TABLE ausleihen (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  kunden_id     INTEGER NOT NULL,
  buch_id       INTEGER NOT NULL,
  ausleihdatum  TEXT NOT NULL,
  rueckgabe     TEXT,
  FOREIGN KEY (kunden_id) REFERENCES kunden(id),
  FOREIGN KEY (buch_id) REFERENCES buecher(id)
);
```

### 1.5.3. 1:1-, 1:n- und m:n-Beziehungen

| **Beziehungstyp**                  | **SQL-Umsetzung**                           |
| ---------------------------------- | ------------------------------------------- |
| **1:1** (Person ↔ Personalausweis) | FK + UNIQUE in einer Tabelle                |
| **1:n** (Kunde → Ausleihen)        | FK in der "vielen"-Seite                    |
| **m:n** (Schüler ↔ Kurse)          | Zwischentabelle mit kombiniertem PK + 2 FKs |

**Zwischentabelle (m:n-Beispiel):**

```sql
CREATE TABLE schueler_kurse (
  schueler_id INTEGER NOT NULL REFERENCES schueler(id),
  kurs_id     INTEGER NOT NULL REFERENCES kurs(id),
  PRIMARY KEY (schueler_id, kurs_id)
);
```

---

## 1.6. Mapping-Übersicht: RM-Konzept → SQL-Konstrukt

Diese Tabelle ist deine **Schnell-Referenz** beim Implementieren:

| **Im Relationenmodell**             | **In SQL schreiben**                      | **Beispiel**                            |
| ----------------------------------- | ----------------------------------------- | --------------------------------------- |
| **Relation KUNDEN**                 | `CREATE TABLE kunden ( ... )`             | `CREATE TABLE kunden (id INTEGER, ...)` |
| **Attribut: ID (PK)**               | `id INTEGER PRIMARY KEY AUTOINCREMENT`    | ✓ oben                                  |
| **Attribut: Name (Text)**           | `name TEXT NOT NULL`                      | ✓ oben                                  |
| **Attribut: Email (eindeutig)**     | `email TEXT UNIQUE`                       | ✓ oben                                  |
| **Attribut: Stadt (optional)**      | `stadt TEXT`                              | (kein NOT NULL)                         |
| **Attribut: Preis (Dezimal)**       | `preis REAL`                              | `preis REAL CHECK(preis > 0)`           |
| **Attribut: Geburtsdatum**          | `geburtsdatum TEXT`                       | (Format: 'YYYY-MM-DD')                  |
| **Fremdschlüssel → andere Tabelle** | `kunden_id INTEGER REFERENCES kunden(id)` | ✓ siehe FOREIGN KEY                     |
| **1:1-Beziehung**                   | FK + UNIQUE in einer Tabelle              | `UNIQUE REFERENCES ...`                 |
| **1:n-Beziehung**                   | FK in der "vielen"-Seite                  | Ausleihen → Kunden                      |
| **m:n-Beziehung**                   | Zwischentabelle mit kombiniertem PK       | schueler_kurse                          |
| **Geschäftsregel (Bedingung)**      | `CHECK (bedingung)`                       | `CHECK(preis > 0)`                      |
| **Standardwert**                    | `DEFAULT wert`                            | `DEFAULT 'aktiv'`                       |

---

## 1.7. Vollständiges Beispiel: Von der Bibliotheks-Anforderung zur SQL-DB

### 1.7.1. Anforderung (Textform)

> Eine Bibliothek verwaltet Bücher mit Autoren. Kunden können Bücher ausleihen. Pro Ausleihe wird das Ausleihdatum und das Rückgabedatum erfasst.

### 1.7.2. Relationenmodell (nach Normalisierung)

```bash
AUTOREN
─────────────────
ID (PK)
Vorname
Nachname
Land
Geburtsjahr (optional)

BUECHER
─────────────────
ID (PK)
Titel
AutorID (FK → AUTOREN)
Genre
Jahr
Preis (muss > 0 sein)
Lagerbestand

KUNDEN
─────────────────
ID (PK)
Name
Email (eindeutig)
Stadt

AUSLEIHEN
─────────────────
ID (PK)
KundenID (FK → KUNDEN)
BuchID (FK → BUECHER)
Ausleihdatum
Rückgabedatum (optional)
```

### 1.7.3. SQL-Implementierung

```sql
-- Tabelle 1: Autoren
CREATE TABLE autoren (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  vorname     TEXT NOT NULL,
  nachname    TEXT NOT NULL,
  land        TEXT,
  geburtsjahr INTEGER CHECK(geburtsjahr > 1400 AND geburtsjahr <= 2026)
);

-- Tabelle 2: Bücher
CREATE TABLE buecher (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  titel         TEXT NOT NULL,
  autor_id      INTEGER NOT NULL REFERENCES autoren(id),
  genre         TEXT,
  jahr          INTEGER CHECK(jahr >= 1450),
  preis         REAL NOT NULL CHECK(preis > 0),
  lagerbestand  INTEGER NOT NULL DEFAULT 0 CHECK(lagerbestand >= 0)
);

-- Tabelle 3: Kunden
CREATE TABLE kunden (
  id    INTEGER PRIMARY KEY AUTOINCREMENT,
  name  TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  stadt TEXT
);

-- Tabelle 4: Ausleihen
CREATE TABLE ausleihen (
  id             INTEGER PRIMARY KEY AUTOINCREMENT,
  kunden_id      INTEGER NOT NULL REFERENCES kunden(id),
  buch_id        INTEGER NOT NULL REFERENCES buecher(id),
  ausleihdatum   TEXT NOT NULL DEFAULT date('now'),
  rueckgabedatum TEXT,
  FOREIGN KEY (kunden_id) REFERENCES kunden(id),
  FOREIGN KEY (buch_id) REFERENCES buecher(id)
);
```

### 1.7.4. Test: Daten einfügen

```sql
-- Autor einfügen
INSERT INTO autoren (vorname, nachname, land, geburtsjahr)
VALUES ('Miguel', 'Cervantes', 'Spanien', 1547);

-- Buch einfügen (mit Verweis auf Autor)
INSERT INTO buecher (titel, autor_id, genre, jahr, preis)
VALUES ('Don Quixote', 1, 'Roman', 1605, 12.99);

-- Kunde einfügen
INSERT INTO kunden (name, email, stadt)
VALUES ('Anna Müller', 'anna.mueller@example.com', 'Zürich');

-- Ausleihe einfügen
INSERT INTO ausleihen (kunden_id, buch_id, ausleihdatum)
VALUES (1, 1, '2026-03-15');
```

---

</br>

# 2. Übungsaufgaben

## 2.1. E-Commerce Shop

| **Vorgabe**             | **Beschreibung**                                              |
| :---------------------- | :------------------------------------------------------------ |
| **Lernziele**           | Kann ein relationales Datenbankmodell mit SQL implementieren. |
| **Sozialform**          | Einzelarbeit                                                  |
| **Auftrag**             | siehe unten                                                   |
| **Hilfsmittel**         |                                                               |
| **Erwartete Resultate** |                                                               |
| **Zeitbedarf**          | 30 min                                                        |
| **Lösungselemente**     | Fehlerfreie SQL-Skriptdateien                                 |
|                         | `e_commerce_shop_create_schema.sql`                           |

**Aufgaben:**

1. Leite aus dem Relationen Modell die Tabellennamen und Attributbezeichnungen ab
2. Lege die korrekten Datentypen zu den Attributen fest
3. Besteimme die Primärschlüssel- u. Fremdschlüsselattribute
4. Lege die Pflichtfelder fest

```bash
KUNDEN
- ID (PK)
- Name
- Email (eindeutig)
- Strasse
- PLZ
- Ort

PRODUKTE
- ID (PK)
- Name
- Beschreibung
- Preis (muss > 0)
- Lagerbestand (muss >= 0)
- Kategorie_ID (FK → KATEGORIEN)

KATEGORIEN
- ID (PK)
- Name (eindeutig)
- Beschreibung

BESTELLUNGEN
- ID (PK)
- Kunde_ID (FK → KUNDEN)
- Bestelldatum
- Status (DEFAULT: 'offen')

BESTELLPOSITIONEN (m:n)
- Bestellung_ID (FK → BESTELLUNGEN)
- Produkt_ID (FK → PRODUKTE)
- Menge (muss > 0)
- Preis_pro_Einheit
```

---

© 2026 Lukas Müller – Licensed under CC BY-NC-ND 4.0
See [LICENSE](../license.md) file for details.
