# Projekt.md — Lotto-Analyse-Tools (Zentrale Übersicht)

> **Für jeden neuen Chat zuerst diese Datei lesen.** Sie beschreibt beide
> Projekte, den aktuellen Stand und die dauerhaft geltenden Regeln. Technische
> Details zum Datenformat stehen in **`LottoNumberFormat_Info.md`**, Planungs-
> notizen zum EuroJackpot in **`eurolotto.md`** (beide im selben Projektordner).

> **AKTUELLER STAND — Entwicklung vorerst abgeschlossen (Stand: 26. Juni 2026):**
> Zwei eigenständige Schwesterprojekte, jeweils Manager (Datenpflege) + Analyzer
> (Auswertung). **Alle vier Tools sind fertig und im Praxiseinsatz bestätigt.**
> Stabiler Stand — keine offenen Baustellen.
>
> | Projekt | Manager | Analyzer | Stand |
> |---------|---------|----------|-------|
> | **6aus49** | `Lotto_Manager_Pro_2-0-0.html` | `Lotto_2-1-0.html` | fertig (v2.1.0) |
> | **EuroJackpot** | `EuroJackpot_Manager_0-1-8.html` | `Eurolotto_1-0-1.html` | fertig (v1.0.1) |
>
> Alle Tools: dunkles Konsolen-Design, Overlay-/Kachel-Bedienung, Tidy-JSON,
> vollständig selbst-enthalten (Bedienung auch vom Handy/unterwegs).
> Web-Recherche bleibt standardmäßig AUS (nur gezielt bei Faktenfragen, mit Ansage).

---

## 1. Überblick — zwei Schwesterprojekte

Beide Spiele werden mit demselben Werkzeug-Konzept abgedeckt, aber als **getrennte
Tools** (eigene Daten, eigenes Design, eigener Update-Ablauf):

| Tool | Rolle | Kurzbeschreibung |
|------|-------|------------------|
| **Manager** (je Spiel) | Datenverwaltung | Ziehungen laden, hinzufügen, suchen, löschen, als JSON speichern |
| **Analyzer** (je Spiel) | Analyse / Prognose | Häufigkeiten, Hot/Cold, Chi², Entropie, Übergänge, Dieter- & KI-Prognose, Forschungslabor, 12 Spielfelder + Wheeling, Drucken |

- **6aus49:** 6 Zahlen (1–49) + Superzahl (0–9). Datenbestand ab 1955.
- **EuroJackpot:** 5 Hauptzahlen (1–50) + 2 Eurozahlen (1–12). Datenbestand ab 2012.

Optische Trennung, damit man die Tools nie verwechselt:
- **6aus49:** Amber/Bernstein-Akzent + Cyan.
- **EuroJackpot:** **Violett** (`#b388ff`) + **Gold** (`#f4c542`) für die Eurozahlen + **Cyan** (`#5cc8ff`) für die KI-Prognose.

---

## 2. Grundprinzipien (gelten für alle Tools)

- **Selbst-enthalten:** Sämtlicher Code (Konverter, Worker, Styles, Chart.js,
  ZIP-Entpackung) steckt in der jeweiligen HTML. Kein Laden aus lokalen Pfaden,
  keine externen Bibliotheken zur Laufzeit — läuft sofort, auch vom Handy und auf
  GitHub Pages.
- **Mathematische Ehrlichkeit (nicht verhandelbar):** Ziehungen sind unabhängig.
  Keine Methode schlägt systematisch den Erwartungswert (Details in Abschnitt 3).
  Die Tools analysieren Muster und unterstützen die Spielauswahl — sie sagen
  nichts vorher. Das wird in der Oberfläche offen kommuniziert.
- **Versionierung:** Versionsnummer als Kommentar am Dateianfang (mit Stand +
  Changelog) **und** sichtbar in der Oberfläche. Semantisch hochzählen.
- **Arbeitsweise:** Erst gründlich in Worten planen, dann komplette getestete
  Versionen bauen (keine halbfertigen Stände). Rigoros mit **Node.js** testen
  (`node --check` + Integrationstests gegen die echte Historie) zwischen den Etappen.
- **Web-Recherche standardmäßig AUS**, nur gezielt für einzelne Faktenfragen,
  immer mit kurzer Ansage.

---

## 3. Mathematische Grundlage (das Herz des Projekts)

Dieter beobachtet: Nach einer Prognose getippt, sind selten mehr als 2 Richtige
dabei. Das ist **kein Fehler der Methode**, sondern exakt die Mathematik des Spiels.
Die echten Wahrscheinlichkeiten pro Tipp (egal welche Methode), Beispiel 6 aus 49:

| Richtige | Wahrscheinlichkeit | grob |
|----------|--------------------|------|
| 0 | 43,6 % | fast jede 2. |
| 1 | 41,3 % | fast jede 2. |
| 2 | 13,2 % | jede ~8. |
| 3 | 1,77 % | jede ~57. |
| 4 | 0,097 % | jede ~1.000. |
| 5 | 0,0018 % | 1 zu ~54.000 |
| 6 | 0,0000072 % | 1 zu ~14 Mio |

Im Schnitt hat **jeder** Tipp ~0,73 richtige Zahlen — deshalb landet der Backtest
immer um diesen Wert. Grund: Die Kugeln haben kein Gedächtnis. Keine Gewichtung
kann die *durchschnittliche* Trefferquote systematisch über den Zufallswert heben.

**Zur Defizit-/„überfällig"-Idee:** Dass sich Häufigkeiten mit mehr Ziehungen
angleichen (Gesetz der großen Zahlen), passiert durch *Verdünnung* alter
Abweichungen, **nicht** durch „Nachholen". Das Angleichen führt sogar *weg* von
Vorhersagbarkeit. Die Tools nutzen Defizit-Logik daher als **Auswahlhilfe**
(welche Zahlen spielen?), nicht als Vorhersage.

### Empirisch bestätigt durch das Forschungslabor
Das Forschungslabor prüft mit echten Statistik-Tests, ob aufeinanderfolgende
Ziehungen zusammenhängen:

- **6aus49:** dreifach „grün" (p = 0,865 / 0,760 / 0,385) → kein nutzbarer
  Zusammenhang. Unabhängigkeit bestätigt.
- **EuroJackpot:** ebenfalls durchweg „grün" — auch die Eurozahlen sind unabhängig
  (sobald man die Epochen korrekt berücksichtigt, siehe 5b). Wichtige Erkenntnis
  beim Bau: Eine naive Wiederholungsanalyse ohne Epochen wirkte zunächst auffällig
  (p = 0,012) — rein wegen der frühen „2 aus 8"-Phase. Epochen-korrekt gerechnet
  ist sie grün. Das ist die Epochen-Logik in Reinform.
- **Aktualitäts-Regler:** Dieters eigene Tests zeigten, dass stärkere Gewichtung
  neuerer Ziehungen das Ergebnis *verschlechtert* (ganz rechts am schlechtesten).
  Das stützt die Mathematik (stärkere Gewichtung = weniger effektive Daten = mehr
  Rauschen). Der Regler bleibt als mildes Feinwerkzeug erhalten, nicht als Brechstange.

### Der eine Hebel, der real wirkt: Wheeling
Das **Abdeck-/Wheeling-System** ist der einzige Weg, der tatsächlich auf die
Häufigkeit von 3ern/4ern wirkt — nicht indem es den Erwartungswert ändert, sondern
indem es die Treffer über die Felder **bündelt** statt sie zu verstreuen. Gute
Systeme liefern etwa doppelt so oft ein Feld mit 3–4 Richtigen wie 12 beliebige
Tipps. Die Garantie wird im Tool **vollständig durchgerechnet und bewiesen** —
angezeigt wird nur, was wirklich gilt.

> Dieters physikalische Hypothesen (Maschinen-Abnutzung über die Jahre,
> mechanische Kopplung der Kugeln innerhalb einer Ziehung) wurden ernst genommen
> und als echte Tests umgesetzt — das Forschungslabor ist das Ergebnis. Keine
> Hypothese ergab einen nutzbaren Vorhersage-Vorteil; das ist selbst ein wertvolles,
> datenbasiertes Ergebnis.

---

## 4. Projekt 6aus49

### Tools & Stand
- **Analyzer `Lotto_2-1-0.html` (v2.1.0)** — fertig. Dient zugleich als
  Transformations-Vorlage für den EuroJackpot-Analyzer.
- **Manager `Lotto_Manager_Pro_2-0-0.html`** — vorerst abgeschlossen.

### Datenquelle (komfortabel: automatisch aktuell)
Es gibt ein gepflegtes Archiv (JohannesFriedrich / LottoNumberArchive) im
Tidy-Format, das **nach jeder Ziehung aktualisiert** wird und ohne Commit-Hash
geladen wird → die Daten veralten nicht von selbst. Beide Tools laden direkt von dort.

### Funktionsumfang Analyzer (Endstand v2.1.0)
- Auswertungen: Häufigkeiten, Hot/Cold, Chi², Entropie, Übergangs-Heatmap,
  Wochentag (Mi/Sa), Gerade/Ungerade, Hoch/Niedrig, Summe.
- **Prognosen:** Dieter- und KI-Methode (je Top-6) + passende **Superzahl** per
  Defizit-Logik (ab Einführung 1991), als 7. Kugel dargestellt.
- **Backtest & Optimierung:** gegen die letzten N Ziehungen, Superzahl getrennt
  bewertet, Gewichte-Optimierung im Hintergrund-Worker.
- **Forschungslabor:** Modul A (Wiederholungen), Modul B (Summen-Folge), Modul C
  (Autokorrelation/Ljung-Box).
- **12 Spielfelder** (3 Methoden + 9 Variationen), gemeinsame Superzahl, voll
  bearbeitbar, würfelbar; **Abdeck-/Wheeling-System** (Pool 8–12, Garantie bewiesen);
  responsiv (Laptop = Lottoschein-Raster, Handy = Kugelreihe); **Drucken / als PDF**.

---

## 5. Projekt EuroJackpot

### Tools & Stand
- **Analyzer `Eurolotto_1-0-1.html` (v1.0.1)** — **fertig**, alle Module im
  Praxiseinsatz bestätigt. Vollständige Übertragung des 6aus49-Analyzers auf
  5 aus 50 + 2 aus 12, mit den Eurozahlen als **vollwertiger zweiter Zahlengruppe**.
  (v1.0.1 korrigierte vier Anzeige-Texte aus der Portierung — siehe 5g.)
- **Manager `EuroJackpot_Manager_0-1-8.html` (v0.1.8)** — fertig, funktional
  komplett. Lädt jetzt **ZIP oder CSV** direkt (siehe 5c).

### 5a. Datenlage (wichtig!)
- **Kein automatisch gepflegtes Archiv** wie bei 6aus49. Die Daten werden **selbst
  gepflegt** über den Manager.
- **Quelle = offizielle WestLotto-Downloads** (autoritativ, geprüft): feste ZIP für
  2012–2021 (ändert sich nie) + laufend aktualisierte ZIP/CSV ab 2022.
- Die vollständige, verifizierte **`eurojackpot_history.json`** (Tidy-Format,
  **966 Ziehungen**, lückenlos, 0 Fehler) liegt auf der eigenen GitHub-Seite und ist
  die **Datenquelle für beide Tools** (Manager und Analyzer laden sie beim Start).

### 5b. Epochen-Regeln (kritisch für die Datenintegrität)
Der Eurozahlen-Pool ist über die Jahre gewachsen. **Alle Euro-Statistiken müssen
epochengewichtet sein**, sonst erscheinen die Zahlen 9–12 fälschlich als „kalt" oder
„überfällig":

| Zeitraum | Eurozahlen | Ziehungstage |
|----------|-----------|--------------|
| bis 09.10.2014 | 1–8 | nur Freitag |
| bis 24.03.2022 | 1–10 | nur Freitag |
| ab 25.03.2022 | **1–12** | Freitag **+ Dienstag** (Di ab 29.03.2022) |

Jackpot-Chance: 1 : 139.838.160. Der Analyzer setzt die Epochen überall um — bei
Hot/Cold, Defizit, beiden Prognosen und sogar in der Wiederholungsanalyse des
Forschungslabors (erst dadurch wird sie korrekt „grün"). **Wichtig:** Epochen
werden *gewichtet*, nicht *gefiltert* — alle 966 Ziehungen zählen, nur die
*Erwartung* je Zahl richtet sich nach der Epoche. (Reines Wegfiltern der Jahre
vor 2022 würde ~85 % der Daten verschenken und wäre falsch.)

### 5c. ⭐ Daten aktuell halten — genaues Vorgehen (halbautomatisch)

> **Goldene Regel: IMMER zuerst die vollständige Historie laden, DANN ergänzen.**
> Die WestLotto-Datei enthält **nur Ziehungen ab 2022** — niemals in einen leeren
> Manager importieren, sonst fehlen die Jahre 2012–2021!

**Standardweg (für eine ODER mehrere verpasste Ziehungen — immer sicher):**

1. **Manager öffnen.** Er lädt `eurojackpot_history.json` automatisch von GitHub.
   Zur Sicherheit oben **„Historie laden"** klicken → Liste muss gefüllt sein (~966+).
2. **Aktuelle Datei bei WestLotto holen:**
   `https://www.westlotto.de/service/downloads/downloads.html#akkordeon_entry_gewinnzahlendownload`
   → **EUROJACKPOT** → Format **CSV** → **„weiter"** → es lädt **`EJ_ab_2022.csv.zip`**.
   **Entpacken ist NICHT mehr nötig.**
3. Im Manager **„CSV oder ZIP wählen"** → die heruntergeladene **ZIP direkt** wählen
   (alternativ eine entpackte CSV — beides geht). Häkchen **„Zusammenführen" bleibt AN**.
   Der Manager entpackt die ZIP automatisch und importiert sie wie eine CSV.
4. Manager meldet *„X neue Ziehung(en) ergänzt, Y bereits vorhanden"* → **alle Lücken
   gefüllt**, egal wie viele gefehlt haben.
5. **„Speichern"** → Dateiname **`eurojackpot_history`** (ohne Datum!) → lädt die
   aktualisierte JSON herunter.
6. Diese Datei auf GitHub (`dietertepe.github.io/Eurolotto/`) **austauschen** (alte
   ersetzen, gleicher Name). → **Daten wieder vollständig & online.**

**Schnellweg (nur wenn die EINE neueste Ziehung fehlt):**
Statt Datei-Upload oben **„Aktuelle Ziehung holen"** → setzt die neueste Ziehung in
die Eingabemaske → **„Ziehung hinzufügen"** → **„Speichern"** → hochladen. Holt immer
nur **eine** Ziehung; bei mehreren Lücken den Standardweg (ZIP) nehmen.

> **Datensicherheit:** Solange „Zusammenführen" AN ist und die Historie geladen war,
> kann beim Import nichts verloren gehen — es kommen nur fehlende Ziehungen dazu,
> Duplikate werden per Datum übersprungen, manuelle Nachträge bleiben erhalten.

> **ZIP-Sicherheitsnetz:** Sollte eine ZIP einmal nicht lesbar sein, kommt eine
> klare Meldung mit dem Hinweis, sie von Hand zu entpacken und die CSV direkt zu
> wählen. Man kann nie in eine Sackgasse geraten.

> **Aktualitätsprüfung im Analyzer:** Beim Laden erkennt der Analyzer fehlende
> Di/Fr-Ziehungen automatisch und zeigt einen wegklickbaren Hinweis mit Link und
> genau dieser Anleitung.

### 5d. Warum nicht vollautomatisch?
WestLotto bietet für die aktuellen Daten **keinen festen Download-Link** (die Datei
wird erst nach dem Formular-Klick erzeugt, die Seite ist zudem robots-/CORS-
geschützt). Ein GitHub-Action müsste die Seite mit einem ferngesteuerten Browser
„bedienen" — fragil und wartungsintensiv. **Entscheidung: bei der stabilen
Halbautomatik (5c) bleiben** — robust und in wenigen Minuten erledigt. Das direkte
ZIP-Laden (ab Manager v0.1.8) spart darin den manuellen Entpack-Schritt.

### 5e. Funktionsumfang Analyzer (v1.0.1) — die Unterschiede zu 6aus49
- **Alles dual (Haupt + Euro):** Häufigkeiten, Hot/Cold, Chi², Entropie, Übergänge,
  Wochentag (**Di/Fr**), Gerade/Ungerade, Hoch/Niedrig, Summe — die Eurozahlen
  bekommen eine eigene, **epochengewichtete** Auswertung.
- **Prognosen:** Dieter + KI je **Top-5** für die Hauptzahlen und **echte Top-2**
  für die Eurozahlen (kein bloßes Superzahl-Anhängsel). Zeitfenster 800 / 400 / 300.
- **Backtest** gegen die letzten 50: Hauptzahl-Trefferverteilung 0–5 **und**
  Eurozahlen-Treffer 0–2 getrennt (Zufallsvergleich ≈ 0,33).
- **Forschungslabor:** die drei Module für die Hauptzahlen + epochen-korrekte
  Euro-Wiederholungsanalyse.
- **12 Spielfelder — jedes mit eigenem Eurozahlen-Paar** (2 aus 12), passend zur
  jeweiligen Methode; getrennte Raster für Hauptzahlen (1–50) und Eurozahlen (1–12).
- Wheeling mit bewiesener Garantie (auf 5er-Felder); Drucken / als PDF mit 5+2-Layout.
- Korrekte Kennzahlen für 5 aus 50: Summe 15–240 (Mitte ~127), Hoch/Niedrig 1–25 /
  26–50, Gerade/Ungerade um 2,5 (3:2 und 2:3 am häufigsten), Chi² df=49 / krit. 66,34.

### 5f. Funktionsumfang Manager (v0.1.8)
- Historie laden (GitHub, Tidy-JSON, mehrere Proxy-Fallbacks).
- **Import per ZIP ODER CSV** — die WestLotto-ZIP wird automatisch entpackt
  (native `DecompressionStream`, keine Bibliothek), dann läuft alles durch denselben
  Import. Erkennt zwei CSV-Formate automatisch (WestLotto-Semikolon mit Quoten +
  eonurk-Komma), Quoten-Spalten werden ignoriert.
- **Merge-Schutz** („Zusammenführen", Standard an): nur fehlende Ziehungen ergänzen,
  Duplikate per Datum übersprungen.
- Eingabe über zwei Kugelraster mit Epochen- und Wochentag-bewusster Validierung
  (Warnung statt hartem Block), Wochentag-Badges (Fr/Di), „Aktuelle Ziehung holen",
  WestLotto-Download-Knopf, Suche/Löschen, Speichern als reines Tidy-Array.

### 5g. Status EuroJackpot (abgeschlossen)
| Aufgabe | Status |
|---------|--------|
| Manager: Raster 5 aus 50 + 2 aus 12, Epochen-Validierung, CSV-Import (WestLotto + eonurk), Wochentag-Badges, Merge-Schutz, **ZIP-Import** | ✅ v0.1.8 |
| Vollständige Historie aus offiziellen Quellen (966 Ziehungen, 0 Fehler) | ✅ erledigt |
| **Analyzer `Eurolotto_1-0-1.html`** (vollständig, alle Module praxisbestätigt) | ✅ **v1.0.1 fertig** |
| Vollautomatik (GitHub-Action) | ⬜ verworfen — WestLotto ohne festen Link |

---

## 6. Datenformat (Tidy) — für beide Projekte

Gespeichert und geladen wird als **reines Tidy-Array** (flach, ohne Wrapper) —
austauschbar mit dem jeweiligen Original, jederzeit wieder ladbar, leicht erweiterbar.

**6aus49:**
```json
[ {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":3}, …,
  {"id":1,"date":"09.10.1955","variable":"Superzahl","value":2} ]
```
intern (breit): `{id, date, Lottozahl:[6], Superzahl:0–9}` — Superzahl optional (vor 1991 keine).

**EuroJackpot:**
```json
[ {"id":1,"date":"23.03.2012","variable":"Hauptzahl","value":5}, …,
  {"id":1,"date":"23.03.2012","variable":"Eurozahl","value":6} ]
```
intern (breit): `{id, date, Hauptzahlen:[5], Eurozahlen:[2]}`.

In jeder HTML stecken die Konverter **`tidyToWide()`** (nach dem Laden) und
**`wideToTidy()`** (beim Speichern), sodass der restliche Code unverändert bleibt.
Der EuroJackpot-Manager liest zusätzlich **ZIP** (entpackt die enthaltene CSV per
nativer `DecompressionStream`) und erkennt zwei CSV-Formate automatisch.

> Validierungsmuster (bewährt): Script extrahieren, `node --check` für exakte
> Fehlerzeilen, dann Integrationstest gegen die echte Historie (bzw. gegen eine
> echte Test-ZIP). „Failed to fetch" in der Chat-Vorschau ist immer ein
> Sandbox-Artefakt, kein echter Fehler.

---

## 7. Wichtige Referenzen

**6aus49 (Daten automatisch aktuell):**
- Daten (Tidy): `https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_tidy_complete.json`
- Projektseite: `https://johannesfriedrich.github.io/LottoNumberArchive/`
- Repository: `https://github.com/JohannesFriedrich/LottoNumberArchive`

**EuroJackpot (Daten selbst gepflegt):**
- Eigene Datendatei (Quelle für beide Tools): `https://dietertepe.github.io/Eurolotto/eurojackpot_history.json`
- Download (ab 2022, ZIP): `https://www.westlotto.de/service/downloads/downloads.html#akkordeon_entry_gewinnzahlendownload` → EUROJACKPOT + CSV → „weiter" → `EJ_ab_2022.csv.zip` (direkt im Manager ladbar, kein Entpacken nötig)
- ZIP 2012–2021 (statisch, nur einmal nötig): `https://www.westlotto.de/westlotto-medien/zahlen-fakten/gewinnzahlendownload/eurojackpot.zip`
- Aktuelle Ziehung zur Kontrolle: `https://www.westlotto.de/eurojackpot/gewinnzahlen/gewinnzahlen.html`
- Hosting beider EuroJackpot-Tools: `https://dietertepe.github.io/Eurolotto/`

**Doku im Projektordner:** `LottoNumberFormat_Info.md` (Format-Details),
`eurolotto.md` (EuroJackpot-Planungsnotizen).

---

## 8. Backlog / Ideen (nicht vorrangig, Entwicklung ruht)

- **„Zufalls-Vergleich" im Backtest:** Methode direkt gegen reinen Zufall stellen,
  um jederzeit sichtbar zu machen, was wirklich Wirkung hat.
- Reine Einzelzahl-Signifikanz und eine echte Paar-Matrix (Kopplung innerhalb einer
  Ziehung) — durch das Forschungslabor sinngemäß abgedeckt, bei Bedarf vertiefbar.
- Paar-/Nachbarschaftsanalyse, Spiel77/Super6 (falls im Format verfügbar).
- Mögliche gezielte Faktenfrage für spätere Web-Recherche: Werden für die
  verschiedenen Ziehungstage dieselbe Maschine/derselbe Kugelsatz verwendet?

---

## 9. Historie (kompakt, für den Werdegang)

- **Phase 1 (6aus49):** Umstellung beider Dateien auf das immer aktuelle Tidy-Format
  (Konverter `tidyToWide`/`wideToTidy`), Funktionsgleichheit gewahrt → v1.0.x.
- **Phase 2 (6aus49):** Kompletter responsiver Neubau mit Overlay-/Kachel-Bedienung,
  saubere Trennung Datenschicht / Analyse / UI; vier neue Auswertungen (Wochentag,
  Gerade/Ungerade, Hoch/Niedrig, Summe); Superzahl-Logik; 12 Spielfelder → v2.0.x.
- **Phase 3 (6aus49):** Prognose-Vertiefung + Forschungslabor (Unabhängigkeit
  empirisch bestätigt), Aktualitäts-Regler, Abdeck-/Wheeling-System, Drucken → v2.1.0.
- **EuroJackpot:** Eigene Tool-Suite aufgebaut — Manager mit WestLotto-CSV-Import,
  verifizierter 966-Ziehungen-Historie, Wochentag-Badges und Merge-Schutz (→ v0.1.7),
  dann der vollständige Analyzer durch systematische Übertragung der 6aus49-Vorlage
  auf 5 aus 50 + 2 aus 12 (→ v1.0.0). Abschluss-Feinschliff: vier Anzeige-Texte aus
  der Portierung korrigiert (Analyzer → v1.0.1) und direktes ZIP-Laden im Manager
  ergänzt (native Entpackung, kein Hand-Entpacken mehr; Manager → v0.1.8).

> **Diese Datei in jedem Chat aktuell halten, wenn sich der Stand ändert** — vor
> allem die Tabellen in Abschnitt 4/5 und den Kopf-Stand. Aktueller Stand:
> Entwicklung vorerst abgeschlossen, alle vier Tools fertig und bestätigt.
