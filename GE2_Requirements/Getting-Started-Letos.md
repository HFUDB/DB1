|                                             |                          |                               |
| ------------------------------------------- | ------------------------ | ----------------------------- |
| **Informatik\*in / Systemtechniker\*in HF** | **Datenbankentwicklung** | ![logo](../x_gitres/logo.png) |

# Getting Started mit Letos

*Letos ist die Datenbankanwendung, mit der wir in diesem Kurs praktisch arbeiten.*

> **Hinweis:** Letos hiess bis vor Kurzem **SQLiteStudio**. Es handelt sich um dieselbe Anwendung, vom selben Entwickler, mit derselben Funktionalität – nur der Name hat sich geändert (Grund: Abgrenzung zum offiziellen SQLite-Projekt). Solltet ihr in älteren Tutorials, Foreneinträgen oder Videos den Namen „SQLiteStudio" sehen, ist damit dieselbe Software gemeint. Screenshots und Anleitungen zu SQLiteStudio lassen sich 1:1 auf Letos übertragen.

## Lernziele

Nach diesem Kapitel können Sie:

- [ ] Letos herunterladen und starten
- [ ] eine neue SQLite-Datenbank erstellen bzw. eine bestehende öffnen
- [ ] sich in der Benutzeroberfläche orientieren (Datenbank-Panel, Editor, Toolbar)
- [ ] eine Tabelle über die grafische Oberfläche erstellen und deren Daten ansehen/bearbeiten
- [ ] SQL-Abfragen im SQL-Editor schreiben und ausführen
- [ ] Daten als CSV importieren und exportieren

## 1. Was ist Letos?

Letos ist ein kostenloses, quelloffenes Verwaltungsprogramm (GUI) für SQLite-Datenbanken. Es läuft unter Windows, macOS und Linux und bietet unter anderem:

- eine grafische Übersicht über Tabellen, Views, Indizes und Trigger
- einen SQL-Editor mit Syntaxhervorhebung und Fehlerprüfung
- Werkzeuge zum Importieren/Exportieren von Daten (CSV, SQL, JSON, XML, HTML, PDF)
- seit Version 4.0 einen integrierten **ERD-Editor**, mit dem sich Tabellenbeziehungen auch grafisch darstellen lassen – nützlich als Ergänzung zu den ERM-Diagrammen aus Kapitel 3/4

Letos enthält keine Werbung, keine Telemetrie und es gibt keine kostenpflichtige Version.

## 2. Installation

1. Offizielle Download-Seite: **[https://letos.org](https://letos.org)**, alternativ direkt über die Releases auf GitHub: `https://github.com/pawelsalawa/letos/releases`
2. Passendes Paket für das eigene Betriebssystem wählen:
   - **Windows:** Installer-Paket (empfohlen, legt automatisch einen Startmenü-Eintrag und die Dateizuordnung für `.sqlite`/`.db` an) oder portable ZIP-Version ohne Installation
   - **macOS:** `.dmg`-Datei herunterladen, Anwendung in den Programme-Ordner ziehen
   - **Linux:** Paket über die Paketverwaltung der Distribution installieren, falls vorhanden, sonst das bereitgestellte Archiv verwenden
3. Letos starten. Beim allerersten Start ist noch keine Datenbank verbunden – das erledigen wir im nächsten Schritt.

## 3. Datenbank erstellen oder öffnen

Über das Menü **Database** (bzw. das entsprechende Symbol in der Toolbar) stehen zwei Optionen zur Verfügung:

| **Aktion**                      | **Vorgehen**                                                                                                                                                  |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Neue Datenbank anlegen**      | *Database → New database* – Speicherort und Dateiname wählen (z.B. `werkstatt.db`); Letos legt die leere SQLite-Datei an und verbindet sich automatisch damit |
| **Bestehende Datenbank öffnen** | *Database → Add a database* – über den Ordner-Button die vorhandene `.db`/`.sqlite`-Datei auswählen                                                           |

Alle verbundenen Datenbanken erscheinen anschliessend im **Databases-Panel** auf der linken Seite, aufgeklappt in ihre Bestandteile (Tables, Views, Indexes, Triggers).

> **Wichtig:** Eine Datenbank aus der Liste zu entfernen, löscht **nicht** die zugrundeliegende Datei – es trennt lediglich die Verbindung in Letos. Um eine Datenbankdatei tatsächlich zu löschen, muss das im Dateisystem (Explorer/Finder) gemacht werden.

## 4. Die Benutzeroberfläche im Überblick

Letos gliedert sich in drei Bereiche:

- **Links – Databases-Panel:** Baumansicht aller verbundenen Datenbanken mit ihren Tabellen, Views, Indizes und Triggern. Rechtsklick auf ein Element öffnet ein Kontextmenü (z.B. *Edit table*, *Delete table*, *New table*).
- **Mitte – Arbeitsbereich:** Hier öffnen sich Tabs für Tabellenstruktur, Dateninhalt oder SQL-Editor-Fenster, je nachdem, was gerade angeklickt wurde.
- **Oben – Toolbar:** Schnellzugriff auf die häufigsten Aktionen: Verbinden/Trennen einer Datenbank, neue Tabelle, SQL-Editor öffnen, Commit/Rollback offener Änderungen.

Zusätzliche Werkzeuge (Import/Export, DDL-Verlauf, Funktions- und Kollationseditor) befinden sich im Menü **Tools**.

## 5. Eine Tabelle erstellen

Es gibt zwei gleichwertige Wege – beide führen zum selben Ergebnis:

**a) Über die grafische Oberfläche:**

1. Rechtsklick auf *Tables* im Databases-Panel → *New table*
2. Tabellennamen vergeben
3. Spalten einzeln hinzufügen: Name, Datentyp, `NOT NULL`, `PRIMARY KEY` etc. per Checkbox setzen
4. Mit dem Speichern-Symbol übernehmen – Letos generiert daraus automatisch das passende `CREATE TABLE`-Statement

**b) Über den SQL-Editor** (siehe Schritt 6): Das `CREATE TABLE`-Statement direkt eintippen und ausführen. Für den weiteren Kursverlauf ist dieser Weg meist der schnellere, da wir DDL-Statements ohnehin schriftlich festhalten wollen.

## 6. SQL-Editor: Abfragen schreiben und ausführen

1. SQL-Editor öffnen: Toolbar-Symbol, Menü *Tools → SQL Editor*, oder Tastenkombination **Alt+E**
2. Sicherstellen, dass oben im Editorfenster die richtige Datenbank ausgewählt ist (Letos kann mehrere Datenbanken gleichzeitig verbunden haben)
3. SQL-Statement eintippen
4. Ausführen:

| **Aktion**                                                                                    | **Standard-Tastenkombination** |
| --------------------------------------------------------------------------------------------- | ------------------------------ |
| Aktuelle Abfrage ausführen                                                                    | **F9**                         |
| Nur die Abfrage unter dem Cursor ausführen (wenn mehrere durch `;` getrennt im Editor stehen) | **Ctrl+F9** (macOS: Cmd+F9)    |
| Alle Abfragen im Editor ausführen                                                             | **Shift+F9**                   |

Das Ergebnis erscheint in einem Grid unterhalb (oder in einem separaten Tab, je nach Einstellung). Bei Syntaxfehlern markiert Letos die betroffene Stelle bereits während der Eingabe mit einer wellenförmigen Unterstreichung.

> Alle Tastenkombinationen lassen sich in den Einstellungen frei anpassen – die obigen sind die Standardbelegung.

### Beispiel: INSERT und SELECT im SQL-Editor

Am einfachsten lässt sich der Ablauf an einem kleinen Beispiel nachvollziehen. Folgende drei Statements nacheinander in den SQL-Editor eintippen und jeweils mit **F9** ausführen:

```sql
-- 1. Tabelle anlegen
CREATE TABLE Person (
    PersonId INTEGER PRIMARY KEY AUTOINCREMENT,
    Name     TEXT NOT NULL,
    Ort      TEXT
);
```

```sql
-- 2. Einen Datensatz einfügen
INSERT INTO Person (Name, Ort)
VALUES ('Meier', 'Zürich');
```

```sql
-- 3. Den eingefügten Datensatz abfragen
SELECT * FROM Person;
```

Nach dem `SELECT`-Befehl erscheint im Ergebnisbereich unterhalb des Editors eine Tabelle mit genau einer Zeile – der soeben eingefügten Person, inklusive der automatisch vergebenen `PersonId`. Wird das `INSERT`-Statement mehrfach ausgeführt (z.B. mit anderen Namen), zeigt das erneute Ausführen von `SELECT * FROM Person;` jeweils alle bisher vorhandenen Zeilen.

> **Tipp:** Stehen mehrere Statements – durch `;` getrennt – im selben Editorfenster, lässt sich mit dem Cursor in einer bestimmten Zeile und **Ctrl+F9** gezielt nur dieses eine Statement ausführen, ohne die anderen erneut auszuführen.

## 7. Daten ansehen und bearbeiten

Doppelklick auf eine Tabelle im Databases-Panel öffnet sie im Arbeitsbereich mit zwei Reitern:

- **Structure:** Spaltendefinitionen, Primär-/Fremdschlüssel, Indizes
- **Data:** tabellarische Ansicht aller Datensätze

Im *Data*-Reiter lassen sich Zellen direkt per Doppelklick bearbeiten. Änderungen werden erst durch **Commit** (grüner Haken in der Toolbar) dauerhaft in die Datenbank geschrieben; **Rollback** verwirft sie wieder. Dieses Verhalten entspricht dem Transaktionsprinzip aus Kapitel 10.6 (`COMMIT`/`ROLLBACK`).

## 8. Import und Export

Über das Menü **Tools**:

- **Import:** Daten aus CSV oder anderen Textformaten in eine (neue oder bestehende) Tabelle einlesen
- **Export:** Datenbank oder einzelne Tabellen als SQL-Skript, CSV, JSON, XML, HTML oder PDF ausgeben

Das ist z.B. praktisch, um Übungsresultate als CSV abzugeben oder eine Musterdatenbank für alle Studierenden als fertiges SQL-Skript bereitzustellen.

## 9. ERD-Editor (Kurzhinweis)

Letos 4.0 bringt einen eingebauten **ERD-Editor**, der bestehende Fremdschlüsselbeziehungen einer Datenbank automatisch als Diagramm darstellt und auch das Neuanlegen von Tabellen per Diagramm erlaubt. Zu finden über *Tools → ERD Editor* (Bezeichnung kann je nach Version leicht abweichen). Dies ersetzt nicht die manuelle ERM-Modellierung aus Kapitel 3/4, eignet sich aber gut, um am Ende zu kontrollieren, ob das tatsächlich erstellte Schema der eigenen Planung entspricht.

## Kurzübung: Erste Schritte selbst ausprobieren

Die Tabelle `Person` aus dem Beispiel in Abschnitt 6 ist bereits angelegt und enthält einen Datensatz. Darauf aufbauend:

1. Über den *Data*-Reiter der Tabelle **von Hand** (also über die GUI, nicht per `INSERT`) zwei weitere Personen mit unterschiedlichen Orten eintragen und committen.
2. Im SQL-Editor eine `SELECT`-Abfrage mit `WHERE` schreiben, die nur die Personen aus einem bestimmten Ort anzeigt.
3. Eine weitere Person diesmal per `INSERT INTO Person (...) VALUES (...);` im SQL-Editor hinzufügen.
4. Mit `SELECT COUNT(*) FROM Person;` prüfen, wie viele Personen insgesamt in der Tabelle stehen.
5. Die Tabelle als CSV exportieren.

Wer diese fünf Schritte ohne Hilfe durchführen kann, beherrscht die Grundfunktionen von Letos und ist bereit für die weiteren Kursinhalte.
