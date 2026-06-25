# Projekt.md — Lotto-Tools (Zentrale Projektübersicht)

> **WICHTIG für jeden neuen Chat:** Diese Datei zuerst lesen. Sie beschreibt das
> Vorhaben, den aktuellen Stand und den Plan. Ergänzende technische Details zum
> JSON-Format stehen in **`LottoNumberFormat_Info.md`** (im selben Projektordner)
> — diese Datei immer mitlesen, wenn es um die Datenstruktur geht.

> **AKTUELLER STAND (Kurzfassung, Details in Abschnitt 9 & 11):**
> Die Tools sind längst über die ursprüngliche Phase-1-Planung (Abschnitte 2–7,
> historisch) hinaus. Aktuelle Dateien:
> - **Analyzer:** `Lotto_2-1-0.html` — wird aktiv weiterentwickelt.
> - **Manager:** `Lotto_Manager_Pro_2-0-0.html` — vorerst abgeschlossen.
>
> Beide im einheitlichen dunklen Konsolen-Design mit Overlay-/Kachel-Bedienung,
> Tidy-JSON-Format, vollständig selbst-enthalten. Nächster Arbeitsbereich:
> **Abschnitt 11 (Phase 3 — Prognose verbessern & Forschungslabor).**
> Web-Recherche bleibt standardmäßig AUS (nur gezielt bei Faktenfragen).
>
> **NEU – Schwesterprojekt EuroJackpot:** siehe **Abschnitt 12** (eigene Tools,
> eigene Daten, eigener Aktuell-Halten-Ablauf). Planungsdetails in `eurolotto.md`.

---

## 1. Worum geht es?

Es existieren zwei eigenständige HTML-Dateien (jeweils komplett selbst-enthalten,
ohne externe lokale Dateien, da Bedienung u. a. vom Handy aus erfolgt):

| Datei | Rolle | Kurzbeschreibung |
|-------|-------|------------------|
| `Lotto_Manager_Pro.html` | **Datenverwaltung** | Ziehungen laden, hinzufügen, suchen, löschen, als JSON speichern |
| `Lotto.html` | **Analyse / Prognose** | Häufigkeiten, Hot/Cold, Chi², Entropie, Übergangs-Heatmap, KI-Vorhersage, Dieter-Prognose + Auto-Optimierung |

Beide nutzen dieselben Lottodaten (Lotto 6aus49, Datenbestand ab 1955).

---

## 2. Das aktuelle Hauptziel (Phase 1)

**Umstellung beider Dateien auf das neue, immer aktuelle JSON-Format (Tidy).**

### Warum?
Bisher laden beide Dateien eine **Breitformat-Datei über eine GitHub-URL mit
festem Commit-Hash** → die Daten sind dadurch eingefroren und veralten.

Neuer Link (immer aktuell, ohne Commit-Hash, meist direkt nach jeder Ziehung
aktualisiert):

```
https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_tidy_complete.json
```

### Der Haken
Das neue Format ist das **Tidy-/Langformat** — strukturell anders als der
bestehende Code erwartet. Details siehe `LottoNumberFormat_Info.md`. Kurzfassung:

**Tidy-Format (NEU, zum Laden):** flaches Array, eine Zeile pro Zahl, kein Wrapper.
```json
[
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":3},
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":12},
  ...
  {"id":<n>,"date":"...","variable":"Superzahl","value":<0-9>}
]
```

**Breitformat (ALT, was der Code intern erwartet):**
```json
{ "data": [ { "id":1, "date":"09.10.1955", "Lottozahl":[3,12,13,16,23,41], "Superzahl":2 } ] }
```

### Lösungsansatz (sicher & minimal-invasiv)
Zwei kleine Konverter-Funktionen in **jeder** HTML (vollständig enthalten):

- **`tidyToWide(tidyArray)`** — wandelt direkt nach dem Laden das Tidy-Array in
  das alte Breitformat um. Dadurch bleibt **der gesamte restliche Code unverändert**
  (Analyse, Backtest, Optimierung, Manager-CRUD laufen weiter wie bisher).
- **`wideToTidy(wideData)`** — wandelt beim Speichern wieder zurück ins Tidy-Format.

**Robustheit:** Superzahl kann fehlen (frühe Ziehungen vor 1991 haben keine). Der
Konverter muss das sauber abfangen (Ziehung gilt als gültig, wenn genau 6
Lottozahlen vorhanden sind; Superzahl optional).

### Speicherformat (Entscheidung)
Gespeichert wird als **reines Tidy-Array** (flach, **ohne** `meta`-Wrapper) —
exakt identisch zum GitHub-Original. Vorteile:
- Gespeicherte Datei ist austauschbar mit dem Original, jederzeit wieder ladbar.
- **Erweiterbar:** Neue Datenarten werden einfach als neue `variable`-Werte
  ergänzt (z. B. `"Wochentag"`, `"Spiel77"`), ohne die Grundstruktur zu ändern.

---

## 3. Was muss aus dem Tidy-Format herausgefiltert werden?

Der bestehende Code benötigt pro Ziehung:
- **`id`** (Integer, fortlaufend)
- **`date`** (String `"TT.MM.JJJJ"`)
- **`Lottozahl`** — Array aus 6 Integers (alle Tidy-Zeilen mit `variable=="Lottozahl"`, gruppiert nach `id`)
- **`Superzahl`** — Integer 0–9 (Tidy-Zeile mit `variable=="Superzahl"`, falls vorhanden)

Gruppierungs-Logik: nach `id` zusammenfassen, `variable`/`value` in die
Zielfelder einsortieren, nach 6 Lottozahlen sortieren.

---

## 4. Rahmenbedingungen (gelten immer)

- **Vollständigkeit:** Sämtlicher Code (inkl. Konverter, Worker, Styles) muss in
  der jeweiligen HTML **komplett enthalten** sein. Kein Laden aus lokalen Pfaden.
- **Versionierung:** Beide angepassten Dateien starten **bei Version 1.0.0**.
  - Versionsnummer als **Kommentar am Anfang** der HTML (mit Stand/Datum + Changelog-Zeile).
  - Versionsnummer **sichtbar in der Oberfläche** anzeigen.
  - Bei Weiterentwicklung hochzählen (1.0.1, 1.1.0, …).
- **Qualität:** Code sorgfältig prüfen, ob alles korrekt zusammenarbeitet. In
  jeder Phase ausgiebig testen, Bugs ausschließen.
- **Funktionsgleichheit:** Nach Phase 1 müssen beide Dateien **exakt** so
  funktionieren wie bisher — nur mit dem neuen Format.

---

## 5. Reihenfolge der Umsetzung (Phase 1)

1. **`Lotto_Manager_Pro.html` zuerst** (erzeugt/verwaltet die Daten; hier müssen
   Laden *und* Speichern im neuen Format getestet werden).
2. **`Lotto.html` danach** (Analyse; nutzt dieselbe `tidyToWide`-Logik).

Pro Datei: Konverter einbauen → Lade-Funktion auf neuen Link umstellen →
Speichern auf Tidy umstellen → testen → Version auf 1.0.0 + sichtbar machen.

---

## 6. Nebenwunsch (nicht vorrangig)

Im `Lotto_Manager_Pro.html` lädt „Aktuelle Ziehung" über den WestLotto-RSS-Feed.
Das funktioniert **in Deutschland**, aber **nicht in Portugal** (vermutlich
Geo-/CORS-Blockade). Schön wäre eine Lösung, die auch aus Portugal funktioniert
(z. B. robusterer Proxy-Fallback oder alternative Quelle). **Niedrige Priorität** —
zuerst läuft die Format-Umstellung.

> **Hinweis:** Sobald der neue Tidy-Link genutzt wird, ist die „aktuelle Ziehung"
> ohnehin meist schon in den Daten enthalten (Datei ist nach jeder Ziehung aktuell).
> Der RSS-Abruf wird dadurch teils überflüssig — das beim Neubau bedenken.

---

## 7. Zukunftsplanung (Phase 2+) — schon jetzt mitdenken

### 7a. Kompletter Neubau beider Dateien
- Von Grund auf neu, nach besten aktuellen Methoden, **leicht erweiterbar**.
- **Optimal & schnell** auf Handy, Laptop, PC (responsives Design).
- Beste Darstellung (CSS oder bessere Methoden — Entscheidung beim Neubau).
- **Overlay-/Modal-Fenster** für bessere Handy-Darstellung in Betracht ziehen
  (Funktionsbereiche per Klick öffnen statt alles auf einer langen Seite).
- Saubere Trennung: Datenschicht (Laden/Konvertieren/Speichern) ↔ Analyse ↔ UI,
  damit Erweiterungen einfach einzuhängen sind.

### 7b. Zusammenführung zu einer Gesamt-HTML
- `Lotto.html` als Basis; den Manager als **Overlay** integrieren.
- Idee: Ein Overlay öffnet automatisch **12 Spielfelder** (optisch wie das
  Klick-Raster im Manager) und füllt sie mit den besten berechneten Tipps.
- Berechnete Zahlen per **Button** in die Spielfelder übernehmen (oder ähnliche
  Mechanik).
- Diese Integration **bereits in der Architektur des Neubaus berücksichtigen**
  (z. B. wiederverwendbare „Spielfeld-Komponente", gemeinsame Datenschicht).

---

## 8. Vorschläge für nützliche Erweiterungen (Backlog, später)

Kandidaten für zusätzliche Auswertungen/Anzeigen (nach und nach):
- **Wochentag** der Ziehung (Mi/Sa) aus dem Datum → getrennte Statistiken.
- **Letzte Ziehung sichtbar anzeigen** (Datum + Zahlen) als Kontrolle, dass
  aktuelle Daten geladen wurden.
- **Gerade/Ungerade-** und **Hoch/Niedrig-Verteilung** als zusätzliche Filter.
- **Zahlensumme** pro Tipp (typische Gewinnsummen liegen in einem Bereich).
- **Paar-/Nachbarschaftsanalyse** (welche Zahlen treten oft gemeinsam auf).
- **Spiel77/Super6** falls im Datenformat verfügbar (prüfen).

> Grundhaltung des Projekts: Lottozahlen sind nicht real vorhersagbar. Ziel ist,
> aus vielen Daten **Wahrscheinlichkeiten** einzugrenzen — Zufall mit
> Wiederholungen. Prognose-Qualität soll schrittweise verbessert werden. Erst
> muss aber alles wie bisher laufen (nur mit neuem Format).

---

## 9. Statusübersicht

| Aufgabe | Status |
|---------|--------|
| Projekt.md angelegt | ✅ erledigt |
| JSON-Format dokumentiert (`LottoNumberFormat_Info.md`) | ✅ vorhanden |
| Konverter-Konzept festgelegt (`tidyToWide` / `wideToTidy`) | ✅ geplant |
| `Lotto_Manager_Pro.html` auf Tidy umstellen (v1.0.0) | ✅ erledigt |
| `Lotto.html` auf Tidy umstellen (v1.0.0) | ✅ erledigt |
| **v1.0.1 beide:** Selbstschliessende Toasts | ✅ erledigt |
| **v1.0.1 Analyzer:** Auto-Analyse nach Laden | ✅ erledigt |
| **v1.0.1 Analyzer:** Optimierung 16–56× schneller (identisches Ergebnis) | ✅ erledigt |
| **v1.0.1 Analyzer:** Letzte Ziehung sichtbar | ✅ erledigt |
| **v1.0.1 Manager:** Auto-Laden beim Start + zeitversetzt aktuelle Ziehung | ✅ erledigt |
| **v1.0.1 Manager:** Sprungleiste (Erster/Letzter/zu ID) + lesbare Liste | ✅ erledigt |
| Portugal-Tauglichkeit „Aktuelle Ziehung" (mehrere Proxy-Fallbacks) | ✅ erledigt (v1.0.1, Test im Ausland steht aus) |
| Neubau Analyzer responsiv + Overlays (Phase 2) → **Lotto_2-0-0.html** | ✅ erledigt |
| **v2.0.0:** 4 neue Auswertungen (Wochentag, Gerade/Ungerade, Hoch/Niedrig, Summe) | ✅ erledigt |
| **v2.0.0:** Engine geprüft — identische Ergebnisse wie v1 (Dieter/KI/Chi²/Optimizer) | ✅ erledigt |
| Neubau Manager responsiv + Overlays (Phase 2) → **Lotto_Manager_Pro_2-0-0.html** | ✅ erledigt |
| **Manager v2.0.0:** CRUD geprüft — identisches Verhalten wie 1.0.1 | ✅ erledigt |
| **Analyzer v2.0.1:** Superzahl per Defizit-Logik für Dieter & KI (ab 1991), 7. Kugel | ✅ erledigt |
| **Analyzer v2.0.1:** Coupon-Vorschau führt Superzahl separat + Ring bei Überschneidung | ✅ erledigt |
| **Analyzer v2.0.2:** Backtest bewertet Superzahl getrennt (Trefferquote, Zufall ~10 %) | ✅ erledigt |
| **Analyzer v2.0.3:** 12 Spielfelder (3 Methoden + 9 Variationen), gemeinsame Superzahl, bearbeitbar + würfeln | ✅ erledigt |
| Entscheidung: Manager bleibt eigenständig, nur Spielfelder wandern in den Analyzer | ✅ entschieden |
| **Analyzer v2.0.4:** Optimierungszeitraum einstellbar (letzte N statt fest 50) | ✅ erledigt |
| **Analyzer v2.0.4:** Übernehmen-Buttons (Dieter/KI → wählbares Feld 1–12 + Superzahl) | ✅ erledigt |

> Diese Tabelle in jedem Chat aktuell halten, wenn sich der Stand ändert.

---

## 11. Phase 3 — Prognose verbessern & „Forschungslabor" (in Planung)

> **Aktueller Stand:** Manager (`Lotto_Manager_Pro_2-0-0.html`) ist vorerst
> abgeschlossen. Aktiv weiterentwickelt wird **nur der Analyzer**, aktuell
> `Lotto_2-0-4.html`. Vorgehen: erst ausführlich in Worten planen und gemeinsam
> besprechen, dann bauen. Web-Recherche bleibt standardmäßig AUS und wird nur
> gezielt für vereinzelte Faktenfragen kurz aktiviert (Claude sagt Bescheid).

### Ausgangslage & ehrliche Grundlage
Dieter beobachtet: Nach der Dieter-Prognose getippt, sind selten mehr als 2
Richtige dabei. Das ist **kein Fehler der Methode**, sondern exakt die Mathematik
von 6 aus 49. Die echten Wahrscheinlichkeiten pro Tipp (egal welche Methode):

| Richtige | Wahrscheinlichkeit | grob |
|----------|--------------------|------|
| 0 | 43,6 % | fast jede 2. |
| 1 | 41,3 % | fast jede 2. |
| 2 | 13,2 % | jede ~8. |
| 3 | 1,77 % | jede ~57. |
| 4 | 0,097 % | jede ~1.000. |
| 5 | 0,0018 % | 1 zu ~54.000 |
| 6 | 0,0000072 % | 1 zu ~14 Mio |

Im Schnitt hat **jeder** Tipp 0,73 richtige Zahlen — deshalb landet der Backtest
immer um ~0,7. Grund: Ziehungen sind **unabhängig** (die Kugeln haben kein
Gedächtnis). Daher kann **keine** Gewichtung die *durchschnittliche* Trefferquote
systematisch über den Zufallswert heben. Das Auto-Optimieren findet Gewichte, die
auf den letzten N Ziehungen *zufällig* etwas besser waren — dieser Vorteil trägt
nicht zuverlässig in die Zukunft (Überanpassung).

**Wichtige Klarstellung zum „Angleichen":** Dass sich Häufigkeiten mit mehr
Ziehungen angleichen (Gesetz der großen Zahlen), passiert durch *Verdünnung*
alter Abweichungen, **nicht** durch „Nachholen" überfälliger Zahlen. Die
Defizit-Idee („überfällige Zahlen kommen jetzt") trägt deshalb mathematisch
nicht. Das Angleichen führt sogar *weg* von Vorhersagbarkeit (weniger Unterschied
zwischen den Zahlen).

### Dieters physikalische Hypothesen (ernst genommen, testbar gemacht)
Dieter bringt überlegenswerte Gedanken ein, die wir als echte Tests umsetzen:
- **Maschine ändert sich über die Zeit:** Ältere Geräte/Kugeln evtl. minimal
  unrund, leicht unterschiedliches Gewicht, Abnutzung → frühere Jahre evtl. leicht
  verzerrt, neuere sauberer. Folge: neuere Ziehungen könnten aussagekräftiger sein
  als alte (nicht wegen „überfällig", sondern weil die *Maschine eine andere ist*).
  Historisch gab es real nachgewiesene verzerrte Lottomaschinen — die Intuition ist
  nicht abwegig.
- **Mechanische Kopplung innerhalb einer Ziehung:** Wird eine Kugel gezogen, ändert
  sich das Verhalten der übrigen — aber **nur innerhalb derselben Ziehung**, nicht
  von Ziehung zu Ziehung (danach neu gemischt). Könnte sich als *Paar-Muster
  innerhalb einer Ziehung* zeigen.
- **Mi/Sa:** Dieter möchte **alle Ziehungen gemeinsam** betrachten (Mi+Sa
  zusammen), nicht trennen.

### Der gemeinsame Plan — Bausteine (Reihenfolge noch offen)
1. **Aktualitäts-Regler (Baustein 1):** Schieberegler, der neueren Ziehungen mehr
   Gewicht gibt (alle Daten bleiben drin, nur unterschiedlich stark gewichtet).
   Ganz links = „alle gleich" (wie bisher), ganz rechts = „neuere zählen stärker".
   Direkt Dieters Idee, im Backtest überprüfbar. Ggf. weitere Regler (z. B.
   Gewicht der Paar-Beziehungen). **Empfohlener Startpunkt.**
   > **Testergebnis (v2.0.5/2.0.6):** In Dieters Tests wurde das Backtest-Ergebnis
   > mit stärkerer Aktualität *schlechter*, ganz rechts am schlechtesten. Das stützt
   > die Mathematik (stärkere Gewichtung = weniger effektive Daten = mehr Rauschen)
   > und spricht dafür, dass „alte vs. neue Maschine" keinen nutzbaren Vorhersage-
   > Vorteil bringt. Regler in v2.0.6 daher bewusst sehr mild gemacht (Halbwertszeit
   > ~20000 bis ~2500) — als Feinwerkzeug, nicht als Brechstange.
2. **Signifikanz-Detektor (Baustein 2):** Prüft jede Einzelzahl über die gesamte
   Historie, ob ihre Häufigkeit so stark vom Erwartungswert abweicht, dass reiner
   Zufall unwahrscheinlich wird. Ehrliche Prüfung der „verborgene Struktur"-These.
3. **Paar-Analyse (Baustein 3):** Sucht Zahlenpaare, die auffällig oft *gemeinsam
   in einer Ziehung* erscheinen (Dieters Kopplungs-These; stand ohnehin im Backlog).
4. **Muster-Suche zwischen Ziehungen (Baustein 4):** Prüft, ob auf bestimmte
   Ziehungs-Eigenschaften (z. B. hohe Summe) systematisch bestimmte andere folgen.
   Muster-Suche, **kein** Vorhersage-Versuch.
5. **Abdeck-/Wheeling-System (Baustein 5):** Verteilt die 12 Felder so, dass
   3er/4er häufiger *gebündelt* in einzelnen Feldern auftreten. Dieter-Prognose
   bestimmt dabei den Favoriten-Pool (~10–12 Zahlen). **Der eine Weg, der real auf
   die Häufigkeit von 3ern/4ern wirkt** — verändert die *Verteilung* der Ergebnisse
   über die Felder, nicht den Erwartungswert. Macht Lotto nicht profitabel, bündelt
   aber die „schönen" Treffer öfter (gute Systeme ~doppelt so oft ein Feld mit 3–4
   Richtigen wie 12 beliebige Tipps).

### Grundhaltung für Phase 3
Tests sollen ehrlich sein: Findet ein Test etwas wirklich Signifikantes → echter,
datenbasierter Hinweis, dem wir nachgehen. Findet er nichts → Gewissheit, und der
Fokus liegt sinnvoll auf Baustein 5 (Abdeck-System). So oder so gewinnt Dieter an
Klarheit. Ein optionaler **„Zufalls-Vergleich" im Backtest** (Methode direkt gegen
reinen Zufall stellen) macht jederzeit sichtbar, was wirklich Wirkung hat.

> Mögliche Faktenfrage für eine *gezielte* Web-Recherche später: Werden für Mi und
> Sa dieselbe Maschine und derselbe Kugelsatz verwendet? (relevant für die Frage,
> ob getrennte Betrachtung je sinnvoll wäre — aktuell aber bewusst gemeinsam).

> **Stand Phase 3 (v2.0.8):** Das Forschungslabor (v2.0.7) hat auf der echten
> Historie dreimal „grün" geliefert — kein nutzbarer Zusammenhang zwischen
> aufeinanderfolgenden Ziehungen. Damit ist die Unabhängigkeit der Ziehungen
> empirisch bestätigt, und der Fokus liegt zu Recht auf dem Abdeck-System
> (Baustein 5, v2.0.8) — dem einzigen Weg, der real auf die Häufigkeit von
> 3ern/4ern wirkt (Bündelung der Treffer, ohne den Erwartungswert zu ändern).
> Die fünf geplanten Bausteine sind damit umgesetzt; Baustein 2/3 (reine
> Einzelzahl-Signifikanz, echte Paar-Matrix) könnten bei Bedarf noch vertieft
> werden, sind aber durch das Forschungslabor sinngemäß abgedeckt.

### Status Phase 3
| Baustein | Status |
|----------|--------|
| 1 — Aktualitäts-Regler | ✅ erledigt (v2.0.5, Feinjustierung v2.0.6; Regler „aus" = bitgenau wie 2.0.4) |
| 2 — Signifikanz-Detektor (Einzelzahlen) | 🟡 teilw. über Forschungslabor v2.0.7 (Sequenz-Tester); reine Einzelzahl-Signifikanz noch offen |
| 3 — Paar-Analyse (Kopplung in Ziehung) | 🟡 verwandt abgedeckt: Modul A (Wiederholungen) in v2.0.7; echte Paar-Matrix noch offen |
| 4 — Muster-Suche zwischen Ziehungen | ✅ erledigt (v2.0.7 Forschungslabor: Modul B Summen-Folge, Modul C Autokorrelation) |
| 5 — Abdeck-/Wheeling-System (12 Felder) | ✅ erledigt (v2.0.8; Pool 8–12 Dieter/KI/kombiniert, Garantie durchgerechnet & bewiesen) |
| UI v2.0.9: Spielfelder responsiv — Laptop zeigt Lottoschein-Raster, Handy kompakte Kugelreihe | ✅ erledigt |
| v2.1.0: Spielfelder drucken / als PDF speichern (native Druckfunktion, weißes Layout, ohne externe Bibliothek) | ✅ erledigt |
| optional — „Zufalls-Vergleich" im Backtest | ⬜ Idee |

---

## 10. Wichtige Referenzen

- **Neuer Daten-Link (Tidy, immer aktuell):**
  `https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_tidy_complete.json`
- **Projektseite mit allen Infos:**
  `https://johannesfriedrich.github.io/LottoNumberArchive/`
- **Repository:**
  `https://github.com/JohannesFriedrich/LottoNumberArchive`
- **Format-Detaildoku:** `LottoNumberFormat_Info.md` (im Projektordner)

---

## 12. EuroJackpot — Schwesterprojekt (eigene Tools, eigene Daten)

> **Eigenes Teilprojekt** (5 aus 50 + 2 aus 12). Technische Planungsdetails in
> **`eurolotto.md`**. Dieser Abschnitt hält den Stand fest — und vor allem die
> **Schritt-für-Schritt-Anleitung zum Aktuell-Halten der Daten** (Abschnitt 12c).

### 12a. Worum geht es?
Zwei eigenständige HTML-Tools (analog zu 6aus49), komplett selbst-enthalten:

| Datei | Rolle | Stand |
|-------|-------|-------|
| `EuroJackpot_Manager_0-1-6.html` | **Datenverwaltung** | aktiv, funktional komplett |
| `Eurolotto_1-0-0.html` (geplant) | **Analyse / Prognose** | in Arbeit (Pendant zu `Lotto_2-1-0.html`) |

Eigenes Design: **Violett** als Akzent + **Gold** für die Eurozahl-Kugeln — klar
abgegrenzt vom 6aus49-Amber/Cyan, damit man die Tools nie verwechselt.

### 12b. Datenlage (wichtig!)
- **Kein automatisch gepflegtes Archiv** wie bei 6aus49 (es gibt kein
  EuroJackpot-Pendant zu JohannesFriedrich). Die Datei wird **selbst gepflegt**.
- **Datenquelle = offizielle WestLotto-Downloads** (autoritativ, geprüft):
  - **2012–2021:** feste ZIP-URL (ändert sich nie).
  - **ab 2022:** laufend aktualisierte CSV (semikolon-getrennt, inkl. Quoten).
- Die vollständige, geprüfte Datei **`eurojackpot_history.json`** (Tidy-Format,
  **966 Ziehungen**, Stand 23.06.2026, lückenlos verifiziert) liegt auf der
  eigenen GitHub-Seite und ist die **Datenquelle für beide Tools** — Manager und
  Analyzer laden sie beim Start (wie 6aus49 das LottoNumberArchive lädt).
- **Epochen-Regeln (kritisch für die Datenintegrität):** Eurozahlen 1–8 bis
  09.10.2014, 1–10 bis 24.03.2022, **1–12 ab 25.03.2022**. Erste
  **Dienstagsziehung: 29.03.2022** (davor nur Freitag).

### 12c. ⭐ Daten aktuell halten — genaues Vorgehen (halbautomatisch)

> **Goldene Regel: IMMER zuerst die vollständige Historie laden, DANN ergänzen.**
> Die WestLotto-CSV enthält **nur Ziehungen ab 2022** — niemals in einen leeren
> Manager importieren, sonst fehlen die Jahre 2012–2021!

**Standardweg (für eine ODER mehrere verpasste Ziehungen — immer sicher):**

1. **Manager öffnen** (`EuroJackpot_Manager_*.html`). Er lädt die eigene
   `eurojackpot_history.json` automatisch von GitHub. Zur Sicherheit oben
   **„Historie laden"** klicken → die Datenliste muss gefüllt sein (~966+).
2. **Aktuelle CSV bei WestLotto holen:**
   `https://www.westlotto.de/service/downloads/downloads.html#akkordeon_entry_gewinnzahlendownload`
   → Abschnitt **„Gewinnzahlen Download für das aktuelle Jahr"** →
   **EUROJACKPOT** wählen → Dateiformat **CSV** → Button **„weiter"**.
   Es lädt eine **`EJ_ab_2022.csv.zip`** herunter → **entpacken** →
   ergibt **`EJ_ab_2022.csv`**.
3. Im Manager **„CSV importieren"** → **„CSV-Datei wählen"** → die
   `EJ_ab_2022.csv` auswählen. Häkchen **„Zusammenführen" bleibt AN**.
4. Der Manager meldet: *„X neue Ziehung(en) ergänzt, Y bereits vorhanden"*.
   → **Alle Lücken sind gefüllt**, egal wie viele Ziehungen gefehlt haben.
5. **„Speichern"** → Dateiname **`eurojackpot_history`** (ohne Datum!) →
   lädt `eurojackpot_history.json` herunter.
6. Diese Datei auf GitHub (`dietertepe.github.io/Eurolotto/`) **austauschen**
   (alte ersetzen, gleicher Name). → **Daten wieder vollständig & online.**

**Schnellweg (nur wenn die EINE neueste Ziehung fehlt):**
- Statt CSV-Import oben **„Aktuelle Ziehung holen"** → setzt die neueste Ziehung
  in die Eingabemaske → **„Ziehung hinzufügen"** → **„Speichern"** → hochladen.
- Holt immer nur **eine** Ziehung. Bei mehreren Lücken → den Standardweg (CSV) nehmen.

> **Datensicherheit:** Solange „Zusammenführen" AN ist und die Historie geladen
> war, kann beim Import **nichts verloren gehen** — es kommen nur fehlende
> Ziehungen dazu, Duplikate werden per Datum übersprungen. Auch eigene manuelle
> Nachträge bleiben erhalten.

### 12d. Warum nicht vollautomatisch?
WestLotto bietet für die aktuellen Daten **keinen festen Download-Link** (die
Datei wird erst nach dem Formular-Klick erzeugt). Ein GitHub-Action müsste die
Seite mit einem ferngesteuerten Browser „bedienen" — fragil und wartungsintensiv,
bricht bei jeder Seitenänderung. **Entscheidung: bei der stabilen Halbautomatik
(12c) bleiben** — robust und in wenigen Minuten erledigt.

### 12e. Internes Datenformat (EuroJackpot)
Analog zu 6aus49, aber mit zwei Zahlengruppen:
- **Tidy (Speichern/Laden):** flaches Array, eine Zeile pro Zahl,
  `variable: "Hauptzahl"` (5×) bzw. `"Eurozahl"` (2×), je
  `{id, date, variable, value}`. Datum `"TT.MM.JJJJ"`.
- **Breit (intern):** `{id, date, Hauptzahlen:[5], Eurozahlen:[2]}`.
- Konverter `tidyToWide()` / `wideToTidy()` wie bei 6aus49.
- **CSV-Import erkennt zwei Formate automatisch:** offizielle WestLotto-CSV
  (Semikolon, Zahlen in Einzelspalten + Quoten) und eonurk-CSV (Komma, Zahlen
  gebündelt). Quoten-Spalten werden ignoriert.

### 12f. Wichtige Referenzen (EuroJackpot)
- **Eigene Datendatei (Tidy, selbst gepflegt) — Quelle für beide Tools:**
  `https://dietertepe.github.io/Eurolotto/eurojackpot_history.json`
- **CSV-Download (aktuelles Jahr, ab 2022):**
  `https://www.westlotto.de/service/downloads/downloads.html#akkordeon_entry_gewinnzahlendownload`
  → EUROJACKPOT + CSV → „weiter" → **`EJ_ab_2022.csv.zip`** → entpacken →
  **`EJ_ab_2022.csv`**
- **ZIP 2012–2021 (statisch, nur einmal nötig):**
  `https://www.westlotto.de/westlotto-medien/zahlen-fakten/gewinnzahlendownload/eurojackpot.zip`
- **Aktuelle Ziehung ansehen (zur Kontrolle):**
  `https://www.westlotto.de/eurojackpot/gewinnzahlen/gewinnzahlen.html`
- **Planungsdetails:** `eurolotto.md` (im Projektordner)

### 12g. Status EuroJackpot
| Aufgabe | Status |
|---------|--------|
| Manager-Grundgerüst (Raster 5 aus 50 + 2 aus 12, Epochen-Validierung) | ✅ v0.1.0 |
| CSV-Import (eonurk) + URL-Laden + Proxy-Fallbacks | ✅ v0.1.1 |
| Speichern mit festem Dateinamen (kein Datums-Suffix) | ✅ v0.1.2 |
| Vorbefüllung letzte Ziehung + optionaler Live-Abruf | ✅ v0.1.3 |
| Zusammenführen-Modus (Merge, schützt vor Datenverlust) | ✅ v0.1.4 |
| Wochentag-Anzeige (Fr/Di) in Liste, Suche, Eingabe | ✅ v0.1.5 |
| WestLotto-CSV-Format im Import erkannt | ✅ v0.1.6 |
| Vollständige Historie aus offiziellen Quellen (966 Ziehungen, 0 Fehler) | ✅ erledigt |
| Analyzer `Eurolotto_1-0-0.html` (Pendant zu Lotto_2-1-0) | ⬜ in Arbeit |
| Vollautomatik (GitHub-Action) | ⬜ verworfen — WestLotto ohne festen Link |

> Diese Tabelle in jedem Chat aktuell halten, wenn sich der Stand ändert.
