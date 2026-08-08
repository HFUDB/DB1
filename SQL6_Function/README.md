|                                             |                          |                               |
| ------------------------------------------- | ------------------------ | ----------------------------- |
| **Informatik\*in / Systemtechniker\*in HF** | **Datenbankentwicklung** | ![logo](../x_gitres/logo.png) |

- [1. Funktionen](#1-funktionen)
  - [1.1. Lernziele](#11-lernziele)
  - [1.2. Zeichenketten-Funktionen (String-Funktionen)](#12-zeichenketten-funktionen-string-funktionen)
  - [1.3. Datums- und Zeitfunktionen](#13-datums--und-zeitfunktionen)
    - [1.3.1. Format-Codes für strftime](#131-format-codes-für-strftime)
    - [1.3.2. Modifikatoren bei date()/datetime()](#132-modifikatoren-bei-datedatetime)
  - [1.4. Mathematische Funktionen](#14-mathematische-funktionen)
  - [1.5. CASE – bedingte Ausdrücke](#15-case--bedingte-ausdrücke)
  - [1.6. Umgang mit NULL: COALESCE und IFNULL](#16-umgang-mit-null-coalesce-und-ifnull)
- [2. Aufgaben](#2-aufgaben)
  - [2.1. Funktionen kombinieren (Bibliothek)](#21-funktionen-kombinieren-bibliothek)
  - [2.2. Funktionen kombinieren (Schulverwaltungsdatenbank)](#22-funktionen-kombinieren-schulverwaltungsdatenbank)

---

# 1. Funktionen

## 1.1. Lernziele

Nach diesem Kapitel können Sie:

- [ ] die wichtigsten Zeichenketten- (String-) Funktionen von SQLite anwenden
- [ ] Datums- und Zeitfunktionen mit `strftime`/`date` korrekt einsetzen
- [ ] mathematische Funktionen sowie `CASE`-Ausdrücke in Abfragen nutzen
- [ ] `NULL`-Werte mit `COALESCE`/`IFNULL` behandeln
- [ ] zwischen zeilenbezogenen Funktionen und Aggregatfunktionen unterscheiden

---

## 1.2. Zeichenketten-Funktionen (String-Funktionen)

| **Funktion**            | **Bedeutung**                 | **Beispiel**                            | **Ergebnis**     |
| ----------------------- | ----------------------------- | --------------------------------------- | ---------------- |
| `LENGTH(x)`             | Länge einer Zeichenkette      | `LENGTH('Roman')`                       | `5`              |
| `UPPER(x)` / `LOWER(x)` | Gross-/Kleinschreibung        | `UPPER('zürich')`                       | `'ZÜRICH'`       |
| `SUBSTR(x, start, len)` | Teilzeichenkette              | `SUBSTR('978-3-16', 1, 4)`              | `'978-'`         |
| `TRIM(x)`               | Leerzeichen am Rand entfernen | `TRIM('  Meier  ')`                     | `'Meier'`        |
| `REPLACE(x, alt, neu)`  | Ersetzen                      | `REPLACE('St. Gallen', 'St.', 'Sankt')` | `'Sankt Gallen'` |
| `\|\|`                  | Verkettung (Konkatenation)    | `vorname \|\| ' ' \|\| nachname`        | `'Peter Meier'`  |

Ergänzend stehen `LTRIM(x)`/`RTRIM(x)` zur Verfügung, um Leerzeichen ausschliesslich links bzw. rechts zu entfernen, sowie `INSTR(x, y)`, das die Position der ersten Fundstelle von `y` innerhalb von `x` zurückgibt (oder `0`, falls nicht gefunden) – nützlich, um z.B. zu prüfen, ob eine Zeichenkette einen bestimmten Bestandteil enthält, bevor mit `SUBSTR` weiterverarbeitet wird.

**Praxisbeispiel:**

```sql
SELECT name,
       UPPER(SUBSTR(name, 1, 1)) || LOWER(SUBSTR(name, 2)) AS NameFormatiert
FROM kunden;
```

Dieses Beispiel demonstriert, wie sich einfache Funktionen **verschachteln** lassen: Zuerst wird der erste Buchstabe (`SUBSTR(name, 1, 1)`) in Grossbuchstaben umgewandelt, danach der Rest des Namens (`SUBSTR(name, 2)`) in Kleinbuchstaben, und beide Teile werden anschliessend mit `||` wieder zusammengefügt. Diese Fähigkeit zur Verschachtelung ist ein zentrales Prinzip von SQL-Funktionen und begegnet uns auch bei Datums- und mathematischen Funktionen wieder.

**Weiteres Beispiel – vollständiger Autorenname aus zwei Spalten:**

```sql
SELECT autor_id,
       vorname || ' ' || nachname AS AutorVollstaendig
FROM autoren;
```

Dieses Beispiel zeigt, dass `||` nicht nur zum Formatieren einzelner Werte dient, sondern auch, um mehrere Spalten (hier `vorname` und `nachname` aus `autoren`) zu einer einzigen, lesbaren Ausgabe zusammenzuführen.

---

## 1.3. Datums- und Zeitfunktionen

SQLite speichert Datumswerte üblicherweise als `TEXT` im ISO-8601-Format (`'YYYY-MM-DD'` bzw. `'YYYY-MM-DD HH:MM:SS'`). Die eingebauten Datumsfunktionen arbeiten direkt mit diesem Format.

| **Funktion**          | **Bedeutung**                                                                 | **Beispiel**                          |
| --------------------- | ----------------------------------------------------------------------------- | ------------------------------------- |
| `date('now')`         | aktuelles Datum                                                               | `date('now')` → `'2026-07-30'`        |
| `datetime('now')`     | aktuelles Datum + Zeit                                                        | `datetime('now')`                     |
| `date(x, '+N days')`  | Datum verschieben                                                             | `date('2026-03-01', '+30 days')`      |
| `strftime(format, x)` | frei formatieren                                                              | `strftime('%Y', ausleihdatum)` → Jahr |
| `julianday(x)`        | Datum in Julianische Tageszahl umwandeln (nützlich für Differenzberechnungen) | siehe unten                           |

### 1.3.1. Format-Codes für strftime

`strftime` erlaubt eine sehr flexible, individuelle Formatierung von Datumswerten. Die wichtigsten Platzhalter im Überblick:

| **Platzhalter** | **Bedeutung**         |
| --------------- | --------------------- |
| `%Y`            | vierstelliges Jahr    |
| `%m`            | Monat (01–12)         |
| `%d`            | Tag (01–31)           |
| `%H`            | Stunde (00–23)        |
| `%M`            | Minute (00–59)        |
| `%W`            | Wochennummer im Jahr  |
| `%j`            | Tag im Jahr (001–366) |

**Beispiel - Ausleihen der letzten 90 Tage:**

```sql
SELECT * FROM ausleihen
WHERE ausleihdatum >= date('now', '-90 days');
```

**Beispiel - Anzahl Tage seit einer Ausleihe berechnen:**

```sql
SELECT id,
       julianday('now') - julianday(ausleihdatum) AS TageSeitAusleihe
FROM ausleihen;
```

**Beispiel - Ausleihen nach Jahr und Monat gruppieren:**

```sql
SELECT strftime('%Y-%m', ausleihdatum) AS Monat, COUNT(*) AS Anzahl
FROM ausleihen
GROUP BY Monat
ORDER BY Monat;
```

### 1.3.2. Modifikatoren bei date()/datetime()

Sowohl `date()` als auch `datetime()` akzeptieren nach dem eigentlichen Datumswert beliebig viele zusätzliche „Modifikatoren", die nacheinander angewendet werden, z.B. `'+1 month'`, `'-7 days'`, `'start of month'`, `'weekday 1'`. Diese Modifikatoren lassen sich auch kombinieren:

```sql
-- Erster Tag des aktuellen Monats
SELECT date('now', 'start of month');

-- Letzter Tag des Vormonats
SELECT date('now', 'start of month', '-1 day');
```

Diese Technik ist in der Praxis sehr nützlich für periodische Auswertungen (z.B. „alle Ausleihen des laufenden Monats"), ohne das aktuelle Datum manuell berechnen zu müssen.

---

## 1.4. Mathematische Funktionen

| **Funktion**                    | **Bedeutung**                                                                                        |
| ------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `ROUND(x, n)`                   | auf n Nachkommastellen runden                                                                        |
| `ABS(x)`                        | Absolutwert                                                                                          |
| `MAX(x, y, …)` / `MIN(x, y, …)` | Grösster/kleinster Wert **innerhalb einer Zeile** (nicht zu verwechseln mit den Aggregatfunktionen!) |

```sql
SELECT titel, ROUND(preis * 1.025, 2) AS PreisInklMwSt
FROM buecher;
```

> **Achtung, häufige Verwechslung:** `MAX(a, b)` mit mehreren Argumenten vergleicht **Werte innerhalb derselben Zeile** und liefert den grösseren zurück. `MAX(Spalte)` mit einem Argument ist die **Aggregatfunktion** und vergleicht Werte **über mehrere Zeilen hinweg**. SQLite unterscheidet dies automatisch anhand der Anzahl Argumente, aber es lohnt sich, den Unterschied bewusst zu machen.

Neben den genannten Funktionen stehen u.a. auch `RANDOM()` (Zufallszahl), `POWER(x, y)` bzw. der Operator-Ersatz über wiederholte Multiplikation sowie – abhängig von der SQLite-Version – erweiterte mathematische Funktionen wie `SQRT`, `SIN`, `COS` zur Verfügung, sofern die entsprechende SQLite-Erweiterung (math functions) aktiviert ist. Für die Zwecke dieses Kurses genügen `ROUND` und `ABS` vollständig.

## 1.5. CASE – bedingte Ausdrücke

`CASE` erlaubt einfache Wenn-Dann-Logik direkt innerhalb einer Abfrage – vergleichbar mit einem `switch` in C oder C#.

```sql
SELECT titel, lagerbestand,
       CASE
           WHEN lagerbestand = 0        THEN 'Nicht verfügbar'
           WHEN lagerbestand < 3        THEN 'Knapp'
           ELSE 'Ausreichend'
       END AS Verfuegbarkeit
FROM buecher;
```

Die `WHEN`-Bedingungen werden **der Reihe nach** von oben nach unten geprüft; sobald eine Bedingung zutrifft, wird der zugehörige `THEN`-Wert verwendet und alle weiteren `WHEN`-Zeilen werden nicht mehr geprüft. Diese Reihenfolge ist wichtig: Vertauscht man im Beispiel die beiden `WHEN`-Zeilen, würde `lagerbestand < 3` auch für `lagerbestand = 0` zutreffen (0 ist ja kleiner als 3) und die spezifischere Regel „Nicht verfügbar" käme nie mehr zum Zug. Als Faustregel gilt daher: die **spezifischste** Bedingung zuerst, die allgemeinste zuletzt (oft als `ELSE`-Fall).

Neben dieser „gesuchten" Form (`CASE WHEN Bedingung THEN …`) existiert auch eine kompaktere „einfache" Form für reine Wertvergleiche:

```sql
SELECT titel,
       CASE genre
           WHEN 'Krimi'    THEN 'Spannung'
           WHEN 'Sachbuch' THEN 'Wissen'
           ELSE 'Sonstiges'
       END AS Kategorie
FROM buecher;
```

## 1.6. Umgang mit NULL: COALESCE und IFNULL

```sql
-- IFNULL: Ersatzwert, falls NULL (genau 2 Argumente)
SELECT name, IFNULL(stadt, 'nicht erfasst') AS Stadt
FROM kunden;

-- COALESCE: liefert den ersten Nicht-NULL-Wert aus einer beliebigen Anzahl Argumente
SELECT name, COALESCE(stadt, email, 'keine Angabe') AS Info
FROM kunden;
```

`IFNULL` ist funktional ein Sonderfall von `COALESCE` mit genau zwei Argumenten und in SQLite eine reine Komfortfunktion. `COALESCE` ist der SQL-Standard-Befehl und funktioniert identisch in praktisch allen anderen DBMS (SQL Server, PostgreSQL usw.) – wer plant, später mit mehreren DBMS zu arbeiten, gewöhnt sich daher am besten von Anfang an `COALESCE` an.

---

</br>

# 2. Aufgaben

## 2.1. Funktionen kombinieren (Bibliothek)

| **Vorgabe**             | **Beschreibung**                                                                      |
| :---------------------- | :------------------------------------------------------------------------------------ |
| **Lernziele**           | String-, Datums- und mathematische Funktionen praktisch anwenden                      |
|                         | `CASE`-Ausdrücke zur Klassifizierung von Daten einsetzen                              |
| **Sozialform**          | Einzelarbeit                                                                          |
| **Auftrag**             | siehe unten                                                                           |
| **Hilfsmittel**         | Laptop mit der Bibliotheksdatenbank (Schema `autoren`/`buecher`/`kunden`/`ausleihen`) |
| **Erwartete Resultate** | 4 lauffähige SQL-Abfragen mit korrekt angewendeten Funktionen                         |
| **Zeitbedarf**          | 30 min                                                                                |
| **Lösungselemente**     | Vollständiges SQL-Skript mit allen 4 Abfragen                                         |

Nutzen Sie die Bibliotheksdatenbank.

1. Erstellen Sie eine Abfrage, die den Namen jedes Kunden in Grossbuchstaben sowie die Länge des Namens ausgibt.
2. Erstellen Sie eine Abfrage, die für jede Ausleihe berechnet, wie viele Tage seit der Ausleihe vergangen sind (bezogen auf das heutige Datum), sortiert von der ältesten zur neuesten Ausleihe.
3. Klassifizieren Sie jedes Buch mit einem `CASE`-Ausdruck nach Erscheinungsjahr:
   - Jahr ab 2020: `'Neu'`
   - Jahr 2010–2019: `'Mittel'`
   - Jahr vor 2010: `'Alt'`
4. Geben Sie für jeden Kunden die Stadt aus – falls nicht erfasst, soll stattdessen `'unbekannt'` erscheinen.

---

## 2.2. Funktionen kombinieren (Schulverwaltungsdatenbank)

| **Vorgabe**             | **Beschreibung**                                    |
| :---------------------- | :-------------------------------------------------- |
| **Lernziele**           | Komplexe SQL-Abfragen für statistische Auswertungen |
| **Sozialform**          | Einzelarbeit                                        |
| **Auftrag**             | siehe unten                                         |
| **Hilfsmittel**         |                                                     |
| **Erwartete Resultate** |                                                     |
| **Zeitbedarf**          | 30 min                                              |
| **Lösungselemente**     | SQL Abfragebefehle                                  |

**A1:**

- Erstelle eine Abfrage, welche die Vornamen und Nachnamen der Studenten kommagetrennt zusammengefügt und als eine Spalte mit Überschrift "*StudentName*"  aufsteigend listet.
- > Hinweis: SQLite verwendet || für String-Konkatenation; CONCAT() wird ab SQLite 3.44 unterstützt. Die ||-Variante ist universell sicherer.

**A2:**

- Ändere die Abfrage aus der A3.1 und zeige Vorname u. Name in Grossschrift an.
- > Hinweis: UPPER() funktioniert in SQLite nur zuverlässig für ASCII-Zeichen. Umlaute (ä, ö, ü) werden nicht konvertiert (SQLite-Limitation).

**A3:**

- Liste alle Studenten mit *Name, Vorname*.
- Die Ausgabe soll nach der Zeichenlänge (Anzahl Zeichen) des *Namens* absteigend sortiert sein.
- > Hinweis: SQLite verwendet LENGTH() statt LEN()

**A4:**

- Liste alle Studenten mit der Kurzbezeichnung (**Erster Buchstabe aus Name u. Vorname**) aufsteigend.
- Die Ausgabe soll *Name, Vorname* und die Spaltenüberschrift "*Kurzzeichen*" anzeigen.
- > Hinweis: SUBSTR() in SQLite

**A5:**

- Liste alle Studenten mit *Vorname, Name und dem Geburtsjahr (z.B. 1990)*. Sortiere die Ausgabe nach dem *Geburtsjahr* absteigend.
- > Hinweis: YEAR() → STRFTIME('%Y', ...) in SQLite; liefert Text, CAST für Sortierung

**A6:**

- Ermittle mit einer Abfrage die Studenten (*Vorname, Name, Geburtsdatum*) welche im Jahr *1989* geboren sind.

**A7:**

- Ermittle mit einer Abfrage die Studenten (*Vorname, Name, Geburtsdatum*) und deren *Alter* in Jahren.
- > Hinweis: DATE('now') liefert das aktuelle Datum.
- > Die Berechnung berücksichtigt, ob der Geburtstag dieses Jahr bereits war.

---

© 2026 Lukas Müller – Licensed under CC BY-NC-ND 4.0
See [LICENSE](..\license.md) file for details.
