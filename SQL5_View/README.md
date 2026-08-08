|                                             |                          |                               |
| ------------------------------------------- | ------------------------ | ----------------------------- |
| **Informatik\*in / Systemtechniker\*in HF** | **Datenbankentwicklung** | ![logo](../x_gitres/logo.png) |

- [1. Views (Virtuelle Tabellen)](#1-views-virtuelle-tabellen)
  - [1.1. Lernziele](#11-lernziele)
  - [1.2. Was ist ein View?](#12-was-ist-ein-view)
  - [1.3. View erstellen](#13-view-erstellen)
  - [1.4. View ändern und löschen](#14-view-ändern-und-löschen)
  - [1.5. Praxisbeispiel: View mit Aggregation](#15-praxisbeispiel-view-mit-aggregation)
  - [1.6. Grenzen von Views](#16-grenzen-von-views)
- [2. Aufgaben](#2-aufgaben)
  - [2.1. Views erstellen und nutzen](#21-views-erstellen-und-nutzen)
  - [2.2. Abschluss-Fallstudie (Repetition)](#22-abschluss-fallstudie-repetition)

---

# 1. Views (Virtuelle Tabellen)

## 1.1. Lernziele

Nach diesem Kapitel können Sie:

- [ ] den Zweck von Views erklären und von echten Tabellen abgrenzen
- [ ] Views erstellen, verwenden und löschen
- [ ] einschätzen, wann sich ein View lohnt und wo seine Grenzen liegen
- [ ] die DML-Befehle `INSERT`, `UPDATE`, `DELETE` sicher anwenden (Wiederholung/Vertiefung)
- [ ] referentielle Integrität beim Löschen verknüpfter Datensätze berücksichtigen

---

## 1.2. Was ist ein View?

Ein **View** (deutsch: Sicht) ist eine **gespeicherte Abfrage**, die wie eine Tabelle angesprochen werden kann – er enthält jedoch selbst **keine eigenen Daten**. Bei jedem Zugriff auf einen View wird die zugrundeliegende Abfrage neu ausgeführt.

**Vorteile von Views:**

- **Komplexität kapseln:** Ein komplizierter Join muss nur einmal geschrieben werden und lässt sich danach einfach wiederverwenden
- **Wiederverwendbarkeit:** Mehrere Personen/Programme greifen auf dieselbe, geprüfte Abfragelogik zu
- **Sicherheit/Datenschutz:** Ein View kann gezielt nur bestimmte Spalten oder gefilterte Zeilen zeigen (z.B. ohne sensible Spalten wie `email`)
- **Stabile Schnittstelle:** Ändert sich die zugrundeliegende Tabellenstruktur, kann der View oft angepasst werden, ohne dass abhängige Abfragen/Programme geändert werden müssen

Ein hilfreiches Bild: Ein View verhält sich wie ein „Fenster" auf die zugrundeliegenden Tabellen. Man sieht durch dieses Fenster stets die aktuellen Daten – ändert sich der zugrundeliegende Datenbestand, ändert sich beim nächsten Zugriff automatisch auch, was durch den View sichtbar ist, ohne dass der View selbst angepasst werden müsste. Diese Eigenschaft unterscheidet einen View fundamental von einer „Momentaufnahme" (z.B. einer exportierten Excel-Tabelle), die den Zustand zu einem bestimmten Zeitpunkt einfriert.

---

## 1.3. View erstellen

```sql
CREATE VIEW AusleihUebersicht AS
SELECT a.id AS AusleiheId, a.ausleihdatum, k.name AS Kunde, b.titel AS Buch
FROM ausleihen a
INNER JOIN kunden k  ON a.kunden_id = k.id
INNER JOIN buecher b ON a.buch_id   = b.id;
```

**Verwendung – wie eine ganz normale Tabelle:**

```sql
SELECT * FROM AusleihUebersicht WHERE ausleihdatum >= '2026-01-01';

SELECT Kunde, COUNT(*) AS Anzahl
FROM AusleihUebersicht
GROUP BY Kunde;
```

Bemerkenswert an diesem zweiten Beispiel: Ein View lässt sich in weiteren Abfragen genauso mit `WHERE`, `GROUP BY` oder sogar mit weiteren `JOIN`s kombinieren wie eine gewöhnliche Tabelle. Für die abfragende Person ist es dabei völlig unerheblich, ob es sich um eine physische Tabelle oder um einen View handelt – dieser Umstand wird als **Datenunabhängigkeit** bezeichnet und ist einer der grossen konzeptionellen Vorteile relationaler Datenbanken.

---

## 1.4. View ändern und löschen

SQLite kennt kein `ALTER VIEW` – ein View muss gelöscht und neu erstellt werden:

```sql
DROP VIEW AusleihUebersicht;

CREATE VIEW AusleihUebersicht AS
SELECT ...  -- angepasste Definition
```

Möchte man sicherstellen, dass ein Skript auch bei mehrfacher Ausführung fehlerfrei durchläuft, empfiehlt sich `DROP VIEW IF EXISTS AusleihUebersicht;` vor dem `CREATE VIEW` – so entsteht kein Fehler, falls der View beim ersten Durchlauf noch gar nicht existierte.

---

## 1.5. Praxisbeispiel: View mit Aggregation

```sql
CREATE VIEW BuchVerfuegbarkeit AS
SELECT b.id, b.titel,
       COALESCE(SUM(CASE WHEN a.rueckgabe IS NULL THEN 1 ELSE 0 END), 0) AS AktuellAusgeliehen,
       b.lagerbestand
FROM buecher b
LEFT JOIN ausleihen a ON b.id = a.buch_id
GROUP BY b.id, b.titel, b.lagerbestand;
```

```sql
-- Bücher, von denen aktuell kein Exemplar mehr verfügbar ist (Nachbestellung/Warteliste)
SELECT * FROM BuchVerfuegbarkeit
WHERE AktuellAusgeliehen >= Lagerbestand;
```

Dieses Beispiel zeigt eindrücklich den praktischen Nutzen von Views: Die relativ komplexe Logik (Join, Zählen der noch nicht zurückgegebenen Ausleihen mittels `CASE`, Behandlung von `NULL`-Werten mit `COALESCE`) muss nur ein einziges Mal sauber formuliert werden. Alle nachfolgenden Auswertungen – etwa für ein Dashboard am Empfangstresen oder einen periodischen Bericht – greifen einfach auf `BuchVerfuegbarkeit` zu, ohne die zugrundeliegende Komplexität erneut nachvollziehen zu müssen.

---

## 1.6. Grenzen von Views

- Ein View ist **kein Ersatz für Performance-Optimierung**: eine komplexe, langsame Abfrage bleibt langsam, auch als View verpackt (SQLite berechnet den View bei jedem Zugriff neu)
- Views, die `GROUP BY`, `DISTINCT` oder Joins über mehrere Tabellen enthalten, sind oft **nicht direkt beschreibbar** (kein `INSERT`/`UPDATE` auf den View möglich, da nicht eindeutig wäre, in welche Basistabelle geschrieben werden müsste)
- Für aktualisierbare Views gelten in SQLite (wie in den meisten DBMS) enge Einschränkungen – für diesen Kurs werden Views ausschliesslich lesend (`SELECT`) eingesetzt

---

</br>

# 2. Aufgaben

## 2.1. Views erstellen und nutzen

| **Vorgabe**             | **Beschreibung**                                                           |
| :---------------------- | :------------------------------------------------------------------------- |
| **Lernziele**           | Views mit Aggregation und Join erstellen und sinnvoll nutzen               |
|                         | Views bei Bedarf wieder löschen                                            |
| **Sozialform**          | Partnerarbeit                                                              |
| **Auftrag**             | siehe unten                                                                |
| **Hilfsmittel**         | Laptop mit der Datenbank (Schema `autoren`/`buecher`/`kunden`/`ausleihen`) |
| **Erwartete Resultate** | Zwei funktionsfähige Views mit korrekten Testabfragen                      |
| **Zeitbedarf**          | 40 min                                                                     |
| **Lösungselemente**     | Vollständiges SQL-Skript mit beiden Views inkl. Testabfragen               |

1. Erstellen Sie einen View `BuchMitLetzterAusleihe`, der pro Buch das Datum der letzten Ausleihe anzeigt (Tipp: `MAX(ausleihdatum)` mit `GROUP BY`).
2. Nutzen Sie diesen View, um alle Bücher zu finden, deren letzte Ausleihe mehr als 180 Tage zurückliegt (oder die noch nie ausgeliehen wurden) – z.B. als Grundlage für einen Aussonderungs-/Rabattvorschlag.
3. Erstellen Sie einen View `KundenAktivitaet`, der pro Kunde die Anzahl Ausleihen sowie den Gesamtwert (Summe der Bücherpreise) der ausgeliehenen Bücher anzeigt.
4. Löschen Sie anschliessend beide Views wieder.

---

## 2.2. Abschluss-Fallstudie (Repetition)

| **Vorgabe**             | **Beschreibung**                                                                        |
| :---------------------- | :-------------------------------------------------------------------------------------- |
| **Lernziele**           | Den gesamten Stoff (ERM → Relationenmodell → DDL → Abfrage → View) anwenden             |
|                         | Ein bestehendes Datenmodell eigenständig um eine neue Entitätsmenge erweitern           |
| **Sozialform**          | Einzelarbeit (empfohlen als Lernkontrolle) oder Partnerarbeit                           |
| **Auftrag**             | siehe unten                                                                             |
| **Hilfsmittel**         | Laptop mit der Bibliotheksdatenbank, gesamtes Kursskript                                |
| **Erwartete Resultate** | ERM-Erweiterung, Relationenmodell, SQL-Skript (DDL/DML) sowie ein funktionierender View |
| **Zeitbedarf**          | 50 min                                                                                  |
| **Lösungselemente**     | Vollständige Musterlösung zu allen 5 Teilaufgaben                                       |

Die Bibliothek möchte die Datenbank um Verlage erweitern. Anforderung:

> „Jedes Buch wird von genau einem Verlag verlegt. Ein Verlag kann mehrere Bücher verlegen. Wir möchten zu jedem Verlag Name, Land und Gründungsjahr erfassen."

1. Erweitern Sie das ERM um die Entitätsmenge `Verlag` und die passende Beziehung (inkl. Kardinalität).
2. Leiten Sie das entsprechende Relationenmodell ab (welche Tabelle erhält den Fremdschlüssel?).
3. Schreiben Sie das `CREATE TABLE`-Statement für `verlage` sowie das `ALTER TABLE`-Statement, um `buecher` um den Fremdschlüssel zu erweitern.
4. Schreiben Sie eine Abfrage, die pro Verlag die Anzahl verlegter Bücher sowie deren durchschnittlichen Preis anzeigt.
5. Erstellen Sie daraus einen View `VerlagsUebersicht`.

---

© 2026 Lukas Müller – Licensed under CC BY-NC-ND 4.0
See [LICENSE](../license.md) file for details.
