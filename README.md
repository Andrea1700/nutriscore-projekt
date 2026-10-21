# Nutri-Score unter der Lupe

**Gut bewertet – aber auch gesund? Wie aussagekräftig ist der Nutri-Score?**
Eine Datenpipeline mit Open Food Facts

| | |
|---|---|
| **Modul** | Data Engineering HS26, ZHAW School of Management and Law |
| **Leistungsnachweis** | Projektarbeit, Teil 1 (Abgabe 27.10.2026, 23:59 Uhr) |
| **Gruppe** | Cindy Sommer (cin614) · Andrea Brunetto (Andrea1700) |
| **Dozierende** | Maria Rothstein, Dr. Václav Pechtor |
| **Datenstand** | Snapshot vom 06.10.2026 |

---

## 1. Problemstellung

Der Nutri-Score bewertet Lebensmittel auf einer Skala von A bis E. Er wird in sieben europäischen Ländern
verwendet, darunter die Schweiz, wo die Kennzeichnung freiwillig ist. Der Score verrechnet ungünstige Nährwerte
(Energie, Zucker, gesättigte Fettsäuren, Salz) mit günstigen (Ballaststoffe, Protein, Obst, Gemüse, Nüsse).
Der **Verarbeitungsgrad** eines Produkts und die meisten **Zusatzstoffe** fliessen nicht ein.
Konsumentinnen und Konsumenten lesen ein A oder B dennoch oft als „gesund“.

## 2. Forschungsfragen und Hypothesen

| | Forschungsfrage |
|---|---|
| FF1 | Wie gut stimmen Nutri-Score und Verarbeitungsgrad (NOVA) überein? |
| FF2 | In welchen Produktkategorien weicht der Nutri-Score am stärksten von Zuckergehalt und Zusatzstoffen ab? |
| FF3 | Trifft die Kritik auch nach dem Algorithmus-Update 2024 noch zu? *(Teil 2)* |

| | Hypothese (α = 0,05) | Frage | Test |
|---|---|---|---|
| H1 | Mehr als 20 % der Produkte mit Nutri-Score A/B sind hochverarbeitet (NOVA 4) | FF1 | einseitiger Binomialtest |
| H2 | Nutri-Score und NOVA hängen nur schwach zusammen (Cramér's V < 0,3) | FF1 | χ²-Test, Cramér's V |
| H3 | Mind. 10 % der Frühstückscerealien mit A/B enthalten > 22,5 g Zucker pro 100 g | FF2 | einseitiger Binomialtest |
| H4 | Anzahl Zusatzstoffe und Nutri-Score-Punkte korrelieren höchstens schwach (\|ρ\| < 0,2) | FF2 | Spearman-Korrelation |

Die Hypothesen wurden vor der Analyse festgelegt.

## 3. Ergebnisse Teil 1 (vorläufig)

| | Kennzahl | n | Entscheid |
|---|---|---|---|
| H1 | 27,7 % NOVA 4 unter A/B, p < 0,001 | 4'936 | **bestätigt** |
| H2 | Cramér's V = 0,363 | 20'454 | verworfen (mittlerer Zusammenhang) |
| H3 | 1,1 % mit > 22,5 g Zucker, p = 1,00 | 281 | verworfen |
| H4 | ρ = 0,358 | 19'870 | verworfen (mehr Zusatzstoffe, schlechterer Score) |

**Kernaussage:** Der Nutri-Score ist besser als sein Ruf – zuckerreiche Frühstückscerealien mit gutem Score sind
selten. Er ist aber blind für den Verarbeitungsgrad: Mehr als jedes vierte Produkt mit A oder B ist hochverarbeitet.

Bei H2 und H4 entscheidet die Effektstärke (V bzw. ρ), nicht der p-Wert: Bei über 20'000 Produkten ist fast
jeder Zusammenhang statistisch signifikant.

---

## 4. Repository-Struktur

```
.
├── README.md                          # diese Datei
├── pyproject.toml / uv.lock           # Python-Umgebung (uv)
├── nutri_score_teil1.ipynb            # kommentierter Code: gesamte Pipeline Teil 1
├── praesentation/
│   ├── nutri_score_teil1.pptx         # Präsentation Kolloquium 1
│   └── nutri_score_teil1.pdf
└── data/
    ├── raw/                           # NICHT auf GitHub (zu gross), siehe Abschnitt 5
    ├── bronze/
    │   └── off_ch_2026-10-06.parquet  # alle Schweizer Produkte, Werte unverändert
    ├── sample/
    │   └── off_ch_silver_sample_1000_2026-10-06.csv   # 1'000 bereinigte Produkte zum Ansehen
    ├── nutriscore.duckdb              # DuckDB-Datenbank mit Silver- und Gold-Tabellen
    └── nova_je_nutriscore.png         # Grafik für die Präsentation
```

## 5. Daten

### Quelle
**Open Food Facts** – offene, gemeinschaftlich gepflegte Lebensmitteldatenbank.
Verwendet wird der nächtliche Parquet-Export:
https://huggingface.co/datasets/openfoodfacts/product-database (Datei `food.parquet`)

*Contains data from Open Food Facts, available under the Open Database License (ODbL 1.0).*
Lizenz: https://opendatacommons.org/licenses/odbl/1.0/

### Vollständige Daten
Der Gesamtexport (`data/raw/off_food_2026-10-06.parquet`, mehrere GB) überschreitet die maximale
Dateigrösse von GitHub und ist deshalb nicht im Repository enthalten.

**Alle Projektdaten (Snapshot 06.10.2026, ZHAW-OneDrive, Zugriff mit ZHAW-Login):**
https://zhaw-my.sharepoint.com/:f:/r/personal/bruneand_students_zhaw_ch/Documents/Nutriscore-Projekt_data?d=w2a7c7987a81f472eace7b8b8632a76f5&csf=1&web=1&e=354ocW

Der Ordner enthält den vollständigen `data`-Ordner: Rohdaten (`raw/`), Bronze-Stufe (`bronze/`),
DuckDB-Datenbank mit Silver- und Gold-Tabellen, Stichprobe (`sample/`) und Grafik.
Zur Reproduktion den Ordner als `data/` in das Repository kopieren; das Notebook überspringt dann den Download.
Die Datengrundlage der Analyse (alle Schweizer Produkte, Werte unverändert) liegt zusätzlich direkt im
Repository unter `data/bronze/off_ch_2026-10-06.parquet`.


### Im Repository enthaltene Daten

| Datei | Inhalt | Stufe |
|---|---|---|
| `data/bronze/off_ch_2026-10-06.parquet` | 105'693 Schweizer Produkte, 15 Spalten, Werte unverändert | Bronze |
| `data/nutriscore.duckdb` | Tabellen `silver_flagged`, `silver_products`, `cleaning_removed`, `gold_h1_…` bis `gold_h4_…` | Silver, Gold |
| `data/sample/…sample_1000….csv` | zufällige Stichprobe von 1'000 bereinigten Produkten (Seed 42) | Silver |

### Wichtigste Spalten

| Spalte | Bedeutung |
|---|---|
| `code` | Barcode (Produkt-ID) |
| `nutriscore_grade` / `nutriscore_score` | Nutri-Score-Klasse (a–e) und Punktwert |
| `nova_group` | Verarbeitungsgrad 1 (unverarbeitet) bis 4 (hochverarbeitet) |
| `additives_n` | Anzahl Zusatzstoffe |
| `energy_kcal`, `fat`, `saturated_fat`, `carbohydrates`, `sugars`, `fiber`, `proteins`, `salt` | Nährwerte pro 100 g |
| `categories_tags` | Produktkategorien |
| `ist_getraenk` | Getränk ja/nein (separat ausgewertet) |

---

## 6. Data-Engineering-Lebenszyklus

| Phase | Umsetzung |
|---|---|
| **Generation** | Hersteller und Freiwillige erfassen Produkte bei Open Food Facts |
| **Ingestion** | Batch-Download des nächtlichen Parquet-Exports (die API ist für Massenabfragen nicht vorgesehen) |
| **Storage** | DuckDB + Parquet in drei Stufen: Bronze (roh) → Silver (bereinigt) → Gold (pro Hypothese) |
| **Transformation** | Sechs Bereinigungsregeln mit Protokoll, explorative Analyse, Hypothesentests |
| **Serving** | Teil 2: Klassifikationsmodelle (Option B) |

**Speicherwahl:** PostgreSQL, MongoDB und DuckDB + Parquet wurden verglichen. DuckDB wurde gewählt, weil es ohne
Server auskommt, verschachtelte Felder unterstützt und spaltenbasiert nur die benötigten Daten liest.
Nachteil: kein Mehrbenutzerbetrieb – für die Analyse durch zwei Personen nicht relevant.

### Bereinigung (Bronze → Silver)

| Regel | Betroffene Zeilen |
|---|---|
| Duplikate (gleicher Barcode) | 1 |
| Unmögliche Werte (< 0 oder > 100 g/100 g, Zucker > Kohlenhydrate, gesättigte Fette > Fett) | 267 |
| Summe Fett + Kohlenhydrate + Protein + Salz > 101 g | 218 |
| Energie weicht > 20 % und > 15 kcal vom berechneten Wert ab | 3'933 |
| Nutri-Score oder NOVA-Gruppe fehlt | 82'186 |
| Getränke (nicht entfernt, separat ausgewertet) | 8'225 |

Ergebnis: 105'693 → **22'949** Produkte (davon 20'454 Lebensmittel ohne Getränke). Mehrfachnennung möglich.
Jede entfernte Zeile ist mit Grund in der Tabelle `cleaning_removed` gespeichert.

---

## 7. Reproduktion

Voraussetzung: [uv](https://docs.astral.sh/uv/) ist installiert.

```bash
git clone [URL DIESES REPOSITORYS]
cd [REPOSITORY-NAME]
git checkout abgabe-teil1
uv sync                       # installiert Python und alle Pakete gemäss uv.lock
uv run jupyter lab            # öffnet Jupyter im Browser
```

Danach `nutri_score_teil1.ipynb` öffnen und *Run → Run All Cells* ausführen.

- Mit den Rohdaten aus Abschnitt 5 in `data/raw/`: Laufzeit wenige Minuten.
- Ohne Rohdaten: Das Notebook lädt den aktuellen Export (mehrere GB, 10–30 Minuten).
- Nur die Silver- und Gold-Tabellen ansehen, ohne Neuberechnung: `data/nutriscore.duckdb` direkt mit DuckDB öffnen.

Verwendete Pakete: duckdb, pandas, scipy, matplotlib, requests, pyarrow, jupyterlab.

---

## 8. Einschränkungen

- **Fehlende Werte:** Bei 78 % der Schweizer Produkte fehlt der Nutri-Score oder die NOVA-Gruppe. Da nicht
  zufällig ist, welche Produkte vollständig erfasst sind, ist eine Verzerrung möglich (Prüfung in Teil 2).
- **Datenqualität:** Die Daten werden von Freiwilligen erfasst; die Bereinigungsregeln fangen nur offensichtliche Fehler ab.
- **Nutri-Score-Version:** Der Export enthält kein Feld, das angibt, nach welcher Version (vor oder nach 2024) der
  Score berechnet wurde. FF3 kann deshalb nicht direkt aus den Daten beantwortet werden.
- **Schwellenwerte:** Die Grenzen 20 % (H1) und 10 % (H3) wurden von der Gruppe festgelegt.

## 9. Ausblick Teil 2

- FF3: Version der Scores klären und alte mit neuer Berechnung vergleichen
- Verzerrung durch fehlende Werte prüfen
- Serving Option B: NOVA 4 aus den Nährwerten vorhersagen, Vergleich von mindestens zwei Klassifikationsmodellen

---

## 10. Quellen

- Department of Health & Food Standards Agency (2016). *Guide to creating a front of pack (FoP) nutrition label for pre-packed products sold through retail outlets*. London.
- Merz, B. et al. (2024). Nutri-Score 2023 update. *Nature Food, 5*(2), 102–110.
- Monteiro, C. A. et al. (2019). Ultra-processed foods: what they are and how to identify them. *Public Health Nutrition, 22*(5), 936–941.
- Open Food Facts (2026). *Data, API and SDKs*. https://world.openfoodfacts.org/data
- Reis, J. & Housley, M. (2022). *Fundamentals of Data Engineering*. Sebastopol: O'Reilly Media.

Quellen für Code und Methoden (DuckDB, SciPy, Cramér's V, EU-Umrechnungsfaktoren) sind direkt im Notebook
kommentiert und im Quellenverzeichnis am Ende des Notebooks aufgeführt.

## 11. Einsatz von KI-Werkzeugen

[Gemäss ZHAW-Vorgaben angeben, ob und wofür KI-Werkzeuge verwendet wurden, z. B. „Bei der Strukturierung der
Präsentation, der Erstellung des Code-Gerüsts und der README wurde Claude (Anthropic) eingesetzt. Alle Inhalte
wurden von der Gruppe geprüft, ausgeführt und verantwortet.“]
