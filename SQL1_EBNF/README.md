|                                             |                          |                               |
| ------------------------------------------- | ------------------------ | ----------------------------- |
| **Informatik\*in / Systemtechniker\*in HF** | **Datenbankentwicklung** | ![logo](../x_gitres/logo.png) |

- [1. EBNF — Syntaxnotation verstehen](#1-ebnf--syntaxnotation-verstehen)
  - [1.1. Lernziele](#11-lernziele)
  - [1.2. Warum braucht es EBNF überhaupt?](#12-warum-braucht-es-ebnf-überhaupt)
  - [1.3. Die wichtigsten Symbole](#13-die-wichtigsten-symbole)
  - [1.4. Konkretes Beispiel: SELECT in EBNF](#14-konkretes-beispiel-select-in-ebnf)
  - [1.5. Brücke zu den Syntaxdiagrammen](#15-brücke-zu-den-syntaxdiagrammen)
- [2. Aufgaben](#2-aufgaben)
  - [2.1. EBNF lesen und anwenden](#21-ebnf-lesen-und-anwenden)

---

</br>

# 1. EBNF — Syntaxnotation verstehen

## 1.1. Lernziele

Nach diesem Kapitel können Sie:

- [ ] erklären, wozu EBNF (Extended Backus-Naur Form) dient
- [ ] die wichtigsten EBNF-Symbole (`::=`, `|`, `[ ]`, `{ }`, `( )`, Anführungszeichen) korrekt deuten
- [ ] zwischen Terminal- und Nichtterminalsymbolen unterscheiden
- [ ] anhand einer gegebenen EBNF-Regel selbstständig beurteilen, ob eine konkrete SQL-Anweisung syntaktisch gültig ist
- [ ] den Zusammenhang zwischen EBNF und den Syntaxdiagrammen (z.B. auf sqlite.org) herstellen

## 1.2. Warum braucht es EBNF überhaupt?

„SELECT Name, Vorname FROM Kunde WHERE Ort = 'Bern';" — diese Anweisung ist syntaktisch korrekt. Aber woher weiss man das? Woher weiss man, dass `WHERE` optional ist, `FROM` aber nicht, oder dass man beliebig viele Spalten mit Komma trennen darf?

**EBNF** ist eine formale Notation, um die **Grammatik** (also die erlaubte Syntax) einer Sprache **eindeutig und vollständig** zu beschreiben — egal ob Programmiersprache oder eben SQL. Statt „meistens schreibt man das so ungefähr" gibt es damit eine exakte Regel, die keine Interpretationsspielräume offen lässt. Genau deshalb ist die offizielle SQLite-Dokumentation (sqlite.org) komplett in EBNF-artigen Syntaxdiagrammen aufgebaut — wer diese Notation lesen kann, kann sich jede SQL-Syntax **selbst** aus der Originalquelle erschliessen, ohne auf ein Tutorial angewiesen zu sein.

[WIKI](https://de.wikipedia.org/wiki/Erweiterte_Backus-Naur-Form)

## 1.3. Die wichtigsten Symbole

| **Symbol**                  | **Bedeutung**                                                            | **Beispiel**        | **Heisst**                                                       |
| --------------------------- | ------------------------------------------------------------------------ | ------------------- | ---------------------------------------------------------------- |
| `::=` (oder `=`)            | „ist definiert als"                                                      | `spalte ::= name`   | Eine `spalte` ist ein `name`                                     |
| `\|`                        | Alternative (entweder/oder)                                              | `'ASC' \| 'DESC'`   | ASC **oder** DESC                                                |
| `[ ... ]`                   | optional (0 oder 1×)                                                     | `[WHERE bedingung]` | WHERE-Klausel darf weggelassen werden                            |
| `{ ... }`                   | Wiederholung (0 bis beliebig oft)                                        | `{',' spalte}`      | beliebig viele weitere Spalten, per Komma getrennt               |
| `( ... )`                   | Gruppierung                                                              | `('ASC' \| 'DESC')` | fasst die Alternative zu einer Einheit zusammen                  |
| `'...'` (Anführungszeichen) | **Terminal** = wörtlich genau so zu schreiben                            | `'SELECT'`          | das Wort SELECT muss exakt so im Code stehen                     |
| *kursiv/Kleinbuchstaben*    | **Nichtterminal** = Platzhalter, wird an anderer Stelle weiter definiert | *spaltenname*       | steht stellvertretend für einen beliebigen gültigen Spaltennamen |

**Merkregel:** GROSSSCHRIFT/Anführungszeichen = das musst du **genau so abschreiben**. Kleinschrift = das ist ein **Platzhalter**, den du durch etwas Eigenes ersetzt.

[Syntax - CREATE TABLE](https://www.sqlite.org/lang_createtable.html)

## 1.4. Konkretes Beispiel: SELECT in EBNF

```ebnf
select_stmt  ::= 'SELECT' spaltenliste 'FROM' tabelle [ 'WHERE' bedingung ] [ 'ORDER BY' spalte [ 'ASC' | 'DESC' ] ]

spaltenliste ::= '*' | spalte { ',' spalte }
```

**Wie liest man das laut?**

- `'SELECT'` → muss wörtlich dastehen
- `spaltenliste` → Platzhalter, weiter unten definiert: entweder ein `*` **oder** eine `spalte`, danach beliebig oft (`{...}`) ein Komma gefolgt von einer weiteren `spalte`
- `'FROM' tabelle` → FROM ist Pflicht, danach ein Tabellenname
- `[ 'WHERE' bedingung ]` → die eckigen Klammern sagen: das ganze WHERE kann komplett weggelassen werden
- `[ 'ASC' | 'DESC' ]` → optional, und falls angegeben, genau eine der beiden Alternativen

Damit lässt sich sofort ablesen: `SELECT *, Name FROM Kunde` wäre laut dieser Regel eigentlich nicht korrekt vorgesehen (die Definition erlaubt `*` **oder** eine Spaltenliste, nicht beides gemischt) — genau solche Fragen lassen sich mit EBNF **selbst beantworten**, statt sie einfach auszuprobieren.

## 1.5. Brücke zu den Syntaxdiagrammen

Die „Eisenbahndiagramme" (railroad diagrams), die zu `CREATE TABLE` im Kursmaterial vorkommen, sind **dieselbe Information wie EBNF, nur als Bild statt als Text**: ein Pfad durch das Diagramm entspricht genau einer gültigen EBNF-Ableitung. Wer EBNF lesen kann, kann auch das Diagramm lesen — und umgekehrt. Auf sqlite.org sind beide Darstellungen zu jedem Befehl verfügbar.

---

# 2. Aufgaben

## 2.1. EBNF lesen und anwenden

| **Vorgabe**             | **Beschreibung**                                                                       |
| :---------------------- | :------------------------------------------------------------------------------------- |
| **Lernziele**           | EBNF-Notation lesen und auf konkrete SQL-Anweisungen anwenden                          |
| **Sozialform**          | Einzelarbeit                                                                           |
| **Auftrag**             | siehe unten                                                                            |
| **Hilfsmittel**         |                                                                                        |
| **Erwartete Resultate** | Für jede Anweisung: „gültig"/„ungültig" mit Begründung anhand der EBNF-Regel           |
| **Zeitbedarf**          | 20 Minuten                                                                             |
| **Lösungselemente**     | Beurteilung aller 7 Anweisungen inkl. Begründung, plus eigene EBNF-Regel für Aufgabe 3 |

**Teil A:** Gegeben ist folgende EBNF-Grammatik:

```ebnf
select_stmt  ::= 'SELECT' spaltenliste 'FROM' tabelle [ 'WHERE' bedingung ] [ 'ORDER BY' spalte [ 'ASC' | 'DESC' ] ]

spaltenliste ::= '*' | spalte { ',' spalte }

bedingung    ::= spalte vergleichsop wert { ( 'AND' | 'OR' ) spalte vergleichsop wert }

vergleichsop ::= '=' | '<>' | '<' | '>' | '<=' | '>='
```

Beurteilen Sie für jede der folgenden Anweisungen, ob sie gemäss dieser Grammatik **gültig** oder **ungültig** ist, und begründen Sie Ihre Antwort mit Bezug auf die jeweilige EBNF-Regel:

1. `SELECT Name FROM Kunde;`
2. `SELECT * FROM Kunde WHERE Ort = 'Bern';`
3. `SELECT Name, Vorname FROM Kunde ORDER BY Name DESC;`
4. `SELECT *, Name FROM Kunde;`
5. `FROM Kunde SELECT Name;`
6. `SELECT Name FROM Kunde WHERE Alter > 18 AND Ort = 'Bern' OR Ort = 'Thun';`
7. `SELECT Name FROM Kunde ORDER BY;`

**Teil B (Bonus):** Formulieren Sie selbst eine EBNF-Regel für eine vereinfachte `INSERT`-Anweisung der Form:

```sql
INSERT INTO Kunde (Name, Ort) VALUES ('Meier', 'Bern');
```

Die Spalten- und Wertelisten sollen dabei beliebig viele (mindestens ein) Einträge enthalten können.

---

© 2026 Lukas Müller – Licensed under CC BY-NC-ND 4.0
See [LICENSE](../license.md) file for details.
