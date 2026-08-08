|                                             |                          |                               |
| ------------------------------------------- | ------------------------ | ----------------------------- |
| **Informatik\*in / Systemtechniker\*in HF** | **Datenbankentwicklung** | ![logo](../x_gitres/logo.png) |

- [1. DML – Data Manipulation Language](#1-dml--data-manipulation-language)
  - [1.1. Lernziele](#11-lernziele)
  - [1.2. Einführung: Was ist DML?](#12-einführung-was-ist-dml)
  - [1.3. Die Beispiel-Datenbank](#13-die-beispiel-datenbank)
  - [1.4. INSERT INTO – Daten einfügen](#14-insert-into--daten-einfügen)
    - [1.4.1. Syntax](#141-syntax)
    - [1.4.2. Beispiele](#142-beispiele)
  - [1.5. UPDATE – Daten ändern](#15-update--daten-ändern)
    - [1.5.1. Syntax](#151-syntax)
    - [1.5.2. Beispiele](#152-beispiele)
    - [1.5.3. Best Practice: Erst SELECT, dann UPDATE](#153-best-practice-erst-select-dann-update)
  - [1.6. DELETE – Daten löschen](#16-delete--daten-löschen)
    - [1.6.1. Syntax](#161-syntax)
    - [1.6.2. Beispiele](#162-beispiele)
    - [1.6.3. DELETE vs. DROP TABLE – der Unterschied](#163-delete-vs-drop-table--der-unterschied)
    - [1.6.4. Fremdschlüssel-Abhängigkeiten beachten](#164-fremdschlüssel-abhängigkeiten-beachten)
  - [1.7. Transaktionen – Sicherheitsnetz bei DML](#17-transaktionen--sicherheitsnetz-bei-dml)
    - [1.7.1. Syntax](#171-syntax)
    - [1.7.2. Praxisbeispiel: Ausleihe verbuchen](#172-praxisbeispiel-ausleihe-verbuchen)
- [2. Übungsaufgaben](#2-übungsaufgaben)
  - [2.1. Mutationen Kundendaten](#21-mutationen-kundendaten)
  - [2.2. Mutationen Blumendaten](#22-mutationen-blumendaten)
  - [2.3. Mutationen Schulverwaltung](#23-mutationen-schulverwaltung)

---

</br>

# 1. DML – Data Manipulation Language

## 1.1. Lernziele

- Daten mit `INSERT INTO` in eine Tabelle einfügen (einzeln & mehrfach)
- Bestehende Datensätze mit `UPDATE` gezielt verändern
- Datensätze mit `DELETE` selektiv oder vollständig löschen
- Die `WHERE`-Klausel korrekt einsetzen, um ungewollte Massenoperationen zu vermeiden
- Typische Fehler und Best Practices bei DML-Operationen benennen

---

## 1.2. Einführung: Was ist DML?

**SQL (Structured Query Language)** gliedert sich in verschiedene Unterbereiche. Die **Data Manipulation Language (DML)** umfasst alle Befehle, die den *Inhalt* von Tabellen verändern – also das Einfügen, Ändern und Löschen von Datensätzen.

| **SQL-Bereich**                | **Abkürzung** | **Wichtigste Befehle**           | **Zweck**              |
| ------------------------------ | ------------- | -------------------------------- | ---------------------- |
| Data Query Language            | DQL           | `SELECT`                         | Daten lesen/abfragen   |
| **Data Manipulation Language** | **DML**       | **`INSERT`, `UPDATE`, `DELETE`** | **Daten verändern**    |
| Data Definition Language       | DDL           | `CREATE`, `ALTER`, `DROP`        | Struktur definieren    |
| Data Control Language          | DCL           | `GRANT`, `REVOKE`                | Berechtigungen steuern |

> **Hinweis:** SQLite kennt kein Berechtigungssystem – `GRANT`/`REVOKE` gibt es dort **nicht** (Details siehe Kapitel „Schema implementieren"). `COMMIT`/`ROLLBACK` (Abschnitt 1.7) funktionieren in SQLite hingegen normal.

In diesem Theorieblock fokussieren wir uns auf die drei zentralen DML-Befehle: `INSERT INTO`, `UPDATE` und `DELETE`.

---

## 1.3. Die Beispiel-Datenbank

Alle Codebeispiele arbeiten mit der bereits bekannten Bibliotheksdatenbank (siehe Kapitel „Schema implementieren"), insbesondere den Tabellen `kunden` und `buecher`:

| **Spaltenname** | **Datentyp** | **Constraint**              | **Beschreibung** |
| --------------- | ------------ | --------------------------- | ---------------- |
| `id`            | `INTEGER`    | `PRIMARY KEY AUTOINCREMENT` | Eindeutige ID    |
| `name`          | `TEXT`       | `NOT NULL`                  | Name des Kunden  |
| `email`         | `TEXT`       | `UNIQUE`                    | E-Mail-Adresse   |
| `stadt`         | `TEXT`       |                             | Wohnort          |

```sql
-- Tabelle erstellen (DDL – zur Referenz)
CREATE TABLE kunden (
    id     INTEGER PRIMARY KEY AUTOINCREMENT,
    name   TEXT    NOT NULL,
    email  TEXT    UNIQUE,
    stadt  TEXT
);
```

Für Beispiele mit relativen Mengenänderungen (z.B. „Lagerbestand verringern") verwenden wir zusätzlich `buecher.lagerbestand`, und für Transaktionsbeispiele die Tabelle `ausleihen` (`id`, `kunden_id`, `buch_id`, `ausleihdatum`, `rueckgabe`) – beide bereits aus dem vorherigen Kapitel bekannt.

---

</br>

## 1.4. INSERT INTO – Daten einfügen

Der Befehl `INSERT INTO` fügt **einen oder mehrere neue Datensätze (Zeilen)** in eine bestehende Tabelle ein.

### 1.4.1. Syntax

```sql
-- Variante A: Alle Spalten (Reihenfolge muss exakt stimmen!)
INSERT INTO tabellenname
VALUES (wert1, wert2, wert3, ...);

-- Variante B: Explizite Spaltenangabe (empfohlen!)
INSERT INTO tabellenname (spalte1, spalte2, spalte3)
VALUES (wert1, wert2, wert3);

-- Variante C: Mehrere Zeilen auf einmal (ab SQLite 3.7.11)
INSERT INTO tabellenname (spalte1, spalte2)
VALUES
    (wert1a, wert2a),
    (wert1b, wert2b),
    (wert1c, wert2c);
```

> **Empfehlung:** Immer **Variante B** verwenden! Durch die explizite Spaltenangabe ist der Code robuster gegenüber späteren Schemaänderungen und sofort lesbar – man sieht direkt, welcher Wert wohin gehört.

### 1.4.2. Beispiele

**Einzelnen Datensatz einfügen:**

```sql
-- Einen neuen Kunden einfügen
INSERT INTO kunden (name, email, stadt)
VALUES ('Anna Meier', 'anna.meier@beispiel.ch', 'Zürich');

-- id wird automatisch vergeben (AUTOINCREMENT)
```

**Mehrere Datensätze einfügen:**

```sql
-- Drei Kunden in einem Befehl einfügen
INSERT INTO kunden (name, email, stadt)
VALUES
    ('Beat Huber',    'beat.huber@beispiel.ch',    'Bern'),
    ('Corina Schmid', 'corina.schmid@beispiel.ch', 'Basel'),
    ('David Keller',  'david.keller@beispiel.ch',  'Luzern');
```

**Einfügen mit DEFAULT-Wert:**

```sql
-- lagerbestand weglassen → DEFAULT 0 wird verwendet
INSERT INTO buecher (titel, autor_id, genre, jahr, preis)
VALUES ('Der Report', 3, 'Thriller', 2020, 24.90);
```

---

## 1.5. UPDATE – Daten ändern

Mit `UPDATE` werden **bestehende Datensätze** in einer Tabelle geändert. Die `WHERE`-Klausel bestimmt, welche Zeilen betroffen sind.

> **Gefahr ohne WHERE:** Wird `UPDATE` ohne `WHERE` ausgeführt, werden **alle** Zeilen der Tabelle geändert!
>
> ```sql
> UPDATE kunden SET stadt = 'unbekannt';  -- Alle Kunden haben plötzlich denselben Wohnort!
> ```
>
> Immer zuerst mit `SELECT` prüfen, welche Zeilen betroffen wären.

### 1.5.1. Syntax

```sql
UPDATE tabellenname
  SET spalte1 = neuer_wert1,
      spalte2 = neuer_wert2
  WHERE bedingung;
```

### 1.5.2. Beispiele

**Einzelnen Wert ändern:**

```sql
-- E-Mail von Kundin Anna Meier aktualisieren
UPDATE kunden
  SET email = 'a.meier@neuedomain.ch'
  WHERE id = 1;
```

**Mehrere Felder gleichzeitig ändern:**

```sql
-- Kundin ist umgezogen: Wohnort und E-Mail gleichzeitig aktualisieren
UPDATE kunden
  SET stadt = 'Luzern',
      email = 'anna.meier@neuedomain.ch'
  WHERE id = 1;
```

**Berechnung auf bestehenden Wert:**

```sql
-- Lagerbestand um 1 verringern (z.B. Ausleihe verbuchen)
UPDATE buecher
  SET lagerbestand = lagerbestand - 1
  WHERE id = 12;

-- Alle Buchpreise um 5% erhöhen (kein WHERE = alle Zeilen betroffen!)
UPDATE buecher
  SET preis = preis * 1.05;
```

**Update mit komplexer WHERE-Bedingung:**

```sql
-- Rückgabe eines Buches verbuchen (Ausleihe abschliessen)
UPDATE ausleihen
  SET rueckgabe = date('now')
  WHERE id = 42;

-- Alle Bücher eines bestimmten Genres im Preis reduzieren
UPDATE buecher
  SET preis = preis * 0.9
  WHERE genre = 'Sachbuch';
```

### 1.5.3. Best Practice: Erst SELECT, dann UPDATE

Vor jedem UPDATE empfiehlt es sich, mit `SELECT` zu prüfen, welche Zeilen betroffen sein werden:

```sql
-- Schritt 1: Prüfen welche Zeilen betroffen sind
SELECT * FROM buecher
  WHERE lagerbestand < 0;

-- Schritt 2: Erst wenn das Ergebnis stimmt, UPDATE ausführen
UPDATE buecher
  SET lagerbestand = 0
  WHERE lagerbestand < 0;
```

---

## 1.6. DELETE – Daten löschen

Mit `DELETE FROM` werden Datensätze aus einer Tabelle entfernt. Das Löschen ist permanent und – ohne Transaktion – nicht rückgängig zu machen.

> **Gefahr ohne WHERE:** `DELETE` ohne `WHERE` löscht **alle** Zeilen der Tabelle – die Tabellenstruktur bleibt erhalten, aber alle Daten sind weg!
>
> ```sql
> DELETE FROM kunden;  -- Alle Kunden sind weg!
> ```

### 1.6.1. Syntax

```sql
-- Bestimmte Zeilen löschen
DELETE FROM tabellenname
  WHERE bedingung;

-- Alle Zeilen löschen (Tabellenstruktur bleibt erhalten)
DELETE FROM tabellenname;
```

### 1.6.2. Beispiele

**Einzelnen Datensatz löschen:**

```sql
-- Kunden mit ID 5 löschen
DELETE FROM kunden
  WHERE id = 5;
```

**Mehrere Datensätze löschen:**

```sql
-- Alle Kunden ohne E-Mail-Adresse löschen
DELETE FROM kunden
  WHERE email IS NULL;

-- Alte, nicht mehr vorrätige Bücher aussortieren
DELETE FROM buecher
  WHERE lagerbestand = 0 AND jahr < 1990;
```

**Alle Zeilen einer Tabelle löschen:**

```sql
-- Alle Datensätze entfernen, Tabellenstruktur bleibt
DELETE FROM kunden;

-- Hinweis: TRUNCATE gibt es in SQLite nicht.
-- DROP TABLE + CREATE TABLE löscht die Tabelle neu
-- und setzt dabei auch den AUTOINCREMENT-Zähler zurück.
```

### 1.6.3. DELETE vs. DROP TABLE – der Unterschied

|                        | `DELETE FROM kunden`   | `DROP TABLE kunden` |
| ---------------------- | ---------------------- | ------------------- |
| **Was wird gelöscht?** | Nur die Zeilen (Daten) | Die gesamte Tabelle |
| **Struktur**           | Bleibt erhalten        | Weg                 |
| **SQL-Bereich**        | DML                    | DDL                 |
| **WHERE möglich?**     | Ja                     | Nein                |
| **Rückgängig?**        | Mit Transaktion ja     | In SQLite schwierig |

### 1.6.4. Fremdschlüssel-Abhängigkeiten beachten

SQLite ignoriert Fremdschlüssel standardmässig – sie müssen explizit aktiviert werden:

```sql
-- Fremdschlüsselprüfung aktivieren (muss pro Session gesetzt werden)
PRAGMA foreign_keys = ON;

-- Beispiel: Tabelle ausleihen mit Fremdschlüsseln auf kunden und buecher
-- CREATE TABLE ausleihen (
--     id         INTEGER PRIMARY KEY AUTOINCREMENT,
--     kunden_id  INTEGER NOT NULL REFERENCES kunden(id),
--     buch_id    INTEGER NOT NULL REFERENCES buecher(id)
-- );

-- Mit aktivierten Fremdschlüsseln schlägt das Löschen fehl,
-- wenn Kunde ID 1 noch offene Ausleihen hat:
DELETE FROM kunden WHERE id = 1;  -- FOREIGN KEY constraint failed!
```

> **Lösungsstrategien bei Fremdschlüsseln:**
>
> 1. Zuerst abhängige Datensätze löschen: `DELETE FROM ausleihen WHERE kunden_id = 1;`
> 2. Dann den Hauptdatensatz: `DELETE FROM kunden WHERE id = 1;`
> 3. Oder: `ON DELETE CASCADE` beim Erstellen der Tabelle definieren

---

## 1.7. Transaktionen – Sicherheitsnetz bei DML

**Transaktionen** fassen mehrere DML-Befehle zu einer **atomaren Einheit** zusammen: Entweder werden alle ausgeführt, oder keiner. Das ist das zentrale Sicherheitsnetz bei Datenmanipulationen.

### 1.7.1. Syntax

```sql
BEGIN TRANSACTION;  -- Transaktion starten

    -- DML-Befehle hier ausführen
    UPDATE buecher SET lagerbestand = lagerbestand - 1 WHERE id = 12;
    INSERT INTO ausleihen (kunden_id, buch_id, ausleihdatum) VALUES (3, 12, date('now'));

COMMIT;    -- Alle Änderungen dauerhaft speichern
-- oder:
ROLLBACK;  -- Alle Änderungen rückgängig machen
```

### 1.7.2. Praxisbeispiel: Ausleihe verbuchen

Ein typischer Anwendungsfall: Ein Buch wird ausgeliehen – dabei müssen **zwei** Änderungen atomar zusammenpassen: der Lagerbestand sinkt, und die Ausleihe wird protokolliert. Würde nur einer der beiden Schritte ausgeführt (z.B. weil die Applikation dazwischen abstürzt), wäre die Datenbank inkonsistent.

```sql
BEGIN TRANSACTION;

-- Schritt 1: Lagerbestand verringern
UPDATE buecher
SET lagerbestand = lagerbestand - 1
WHERE id = 12 AND lagerbestand > 0;

-- Schritt 2: Ausleihe protokollieren
INSERT INTO ausleihen (kunden_id, buch_id, ausleihdatum)
VALUES (3, 12, date('now'));

-- Nur wenn beide Schritte ohne Fehler verlaufen: speichern
COMMIT;
```

> Schlägt Schritt 1 fehl (z.B. weil `lagerbestand` bereits 0 ist und die `WHERE`-Bedingung keine Zeile trifft), lässt sich das vor dem `COMMIT` mit einem `SELECT changes();` oder einer Kontrollabfrage prüfen und stattdessen ein `ROLLBACK` ausführen – genau dafür ist die Transaktion da.
>
> **ACID-Prinzip:** Transaktionen folgen dem ACID-Prinzip:
>
> - **A**tomicity – Alles oder nichts
> - **C**onsistency – Datenbank bleibt in einem gültigen Zustand
> - **I**solation – Transaktionen beeinflussen sich nicht gegenseitig
> - **D**urability – Gespeicherte Daten bleiben dauerhaft erhalten

---

</br>

# 2. Übungsaufgaben

## 2.1. Mutationen Kundendaten

| **Vorgabe**             | **Beschreibung**                                         |
| :---------------------- | :------------------------------------------------------- |
| **Lernziele**           | Kann SQL DDL und DML-Befehle ausführen                   |
|                         | Kann Daten in eine Tabelle einfügen, ändern und löschen. |
|                         | Kann Daten in einer Tabelle abfragen                     |
| **Sozialform**          | Einzelarbeit                                             |
| **Auftrag**             | siehe unten                                              |
| **Hilfsmittel**         |                                                          |
| **Erwartete Resultate** |                                                          |
| **Zeitbedarf**          | 20 min                                                   |
| **Lösungselemente**     | SQL Skript File                                          |

Erstellen Sie die Tabelle `kunden` und lösen Sie folgende Aufgaben in Letos.

```sql
-- Tabelle erstellen (DDL – zur Referenz)
CREATE TABLE kunden (
    id     INTEGER  PRIMARY KEY AUTOINCREMENT,
    name   TEXT     NOT NULL,
    email  TEXT     UNIQUE,
    stadt  TEXT
);
```

**Aufgabe 1 – INSERT INTO:**

**a)** Fügen Sie folgende drei Kunden **einzeln** ein:

- Franziska Gross, <franziska.gross@hf.ch>, Bern
- Marco Brun, <marco.brun@hf.ch>, Basel
- Selina Vogel (keine E-Mail), Chur

**b)** Fügen Sie alle drei Kunden in einem **einzigen** `INSERT`-Befehl ein.

---

**Aufgabe 2 – UPDATE:**

**a)** Ändern Sie den Wohnort von Franziska Gross auf „Thun".

**b)** Ändern Sie die E-Mail von Marco Brun auf: `m.brun@neuedomain.ch`

**c)** Setzen Sie den Wohnort **aller** Kunden ohne Stadt-Angabe auf „unbekannt".

**d)** Führen Sie zuerst einen `SELECT` aus, um zu prüfen, welche Kunden betroffen wären.

---

**Aufgabe 3 – DELETE:**

**a)** Löschen Sie den Kunden mit der ID 2.

**b)** Löschen Sie alle Kunden ohne E-Mail-Adresse (`WHERE email IS NULL`).

**c)** Versuchen Sie, alle Datensätze zu löschen – nutzen Sie dafür eine Transaktion und führen Sie anschliessend ein `ROLLBACK` durch.

---

**Aufgabe 4 – Transaktion (Bonusaufgabe):**

Sofern bereits vorhanden, verwenden Sie zusätzlich die Tabellen `buecher` und `ausleihen` aus dem vorherigen Kapitel und verbuchen Sie die Ausleihe von Buch-ID 5 durch Kunde 1:

1. Starten Sie eine Transaktion.
2. Verringern Sie den `lagerbestand` des Buchs um 1.
3. Fügen Sie einen neuen Datensatz in `ausleihen` ein (`kunden_id`, `buch_id`, `ausleihdatum`).
4. Prüfen Sie das Ergebnis mit `SELECT`.
5. Erst dann: `COMMIT`.

---

## 2.2. Mutationen Blumendaten

| **Vorgabe**             | **Beschreibung**                                         |
| :---------------------- | :------------------------------------------------------- |
| **Lernziele**           | Kann SQL DDL und DML-Befehle ausführen                   |
|                         | Kann Daten in eine Tabelle einfügen, ändern und löschen. |
|                         | Kann Daten in einer Tabelle abfragen                     |
| **Sozialform**          | Einzelarbeit                                             |
| **Auftrag**             | siehe unten                                              |
| **Hilfsmittel**         | Kursunterlagen, SQL-Management Studio                    |
| **Erwartete Resultate** |                                                          |
| **Zeitbedarf**          | 20 min                                                   |
| **Lösungselemente**     | Vollständige SQL-Skript-Datei                            |

Erstelle zu den nachfolgenden Aufgaben die korrekten und vollständigen SQL-Befehle. Die
Lösungen müssen mit Aufgabentext in einer SQL-Skriptdatei abgespeichert werden.

**A1:** Kreiere mit SQL-Statements eine Tabelle BLUME mit den Attributen:

```sql
CREATE TABLE IF NOT EXISTS Blume
(
    ID    INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    Name  TEXT    NOT NULL,
    Preis REAL    NOT NULL
);
```

**A2:** Füge zwei Datensätze zu mit den Werten:

| **ID** | **Name** | **Preis** |
| :----- | :------- | :-------- |
| 10     | Nelke    | 2.35      |
| 11     | Rose     | 4.50      |

**A3:** Überprüfe diese Einträge mit einem `SELECT`-Statement.

**A4:** Erhöhe den Preis der Rose um 10%.

**A5:** Überprüfe das Update in A4 mit einem `SELECT`-Statement.

**A6:** Lösche den Datensatz mit der **Nelke**

**A7:** Erweitere die `BLUME` Tabelle mit Spalte SORTE (TEXT)).

**A8:** Stelle sicher, dass nicht zweimal derselbe Blumenname eingetragen werden kann.

**A9:** Stelle sicher, dass der Blumenpreis immer > 0 sein muss.

**A10:** Entferne die gesamte Tabelle `BLUME`, inklusive Metadaten.

---

## 2.3. Mutationen Schulverwaltung

| **Vorgabe**             | **Beschreibung**                                         |
| :---------------------- | :------------------------------------------------------- |
| **Lernziele**           | Kann SQL DDL und DML-Befehle ausführen                   |
|                         | Kann Daten in eine Tabelle einfügen, ändern und löschen. |
|                         | Kann Daten in einer Tabelle abfragen                     |
| **Sozialform**          | Einzelarbeit                                             |
| **Auftrag**             | siehe unten                                              |
| **Hilfsmittel**         |                                                          |
| **Erwartete Resultate** |                                                          |
| **Zeitbedarf**          | 30 min                                                   |
| **Lösungselemente**     | Fehlerfreie SQL-Skriptdateien                            |
|                         | `insert_data.sql`                                        |

**Ausgangssituation:**

- Sie verwenden das Datenbank Modell vorangegangener Aufgabe.

Fügen Sie per SQL Befehl (insert into …) alle Datenzeilen aus der Tabelle unten in Ihre Datenbank ein.

```sql
INSERT INTO [user.]tabelle [ (column [,column] ...) ]
VALUES (value [,value] ...)
```

---

© 2026 Lukas Müller – Licensed under CC BY-NC-ND 4.0
See [LICENSE](../license.md) file for details.
