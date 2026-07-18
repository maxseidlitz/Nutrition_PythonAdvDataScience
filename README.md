# PyTUMs: Open Food Facts — NOVA & Nutri-Score

Abschlussprojekt für den TUM-Kurs **Python and Advanced Data Science** (Dozent: Batuhan Can).

**Team:** Maximilian Seidlitz, Niklas Matusik, Bilal Mert, Bui Ngoc Minh Quach  
**Abgabe:** 21.07.2026

## Ziel

Vorhersage der **NOVA-Gruppe** (Verarbeitungsgrad 1–4) anhand von Nährwerten, Zusatzstoff-/Zutaten-Anzahl und Produktkategorie — und Vergleich mit der Vorhersage des **Nutri-Score** (A–E).

NOVA und Nutri-Score messen unterschiedliche Dimensionen: Verarbeitungsgrad vs. Nährwertprofil. Ein Produkt kann z. B. Nutri-Score A haben und trotzdem ultra-verarbeitet sein (NOVA 4).

## Hauptergebnisse

| Aufgabe | Bestes Modell | F1 (gewichtet) |
|---|---|---|
| **NOVA** (Hauptproblem) | Random Forest | **0,857** |
| **Nutri-Score** (Vergleich) | Random Forest | **0,827** |

Wichtigste Features für NOVA: `ingredients_n` (22,1 %) und `additives_n` (19,4 %). Ohne Zusatzstoffe/Kategorie fällt NOVA-F1 auf 0,744 (Leakage-Check).

## Datensatz

| Eigenschaft | Beschreibung |
|---|---|
| **Quelle** | [Open Food Facts Product Database](https://huggingface.co/datasets/openfoodfacts/product-database) auf Hugging Face |
| **Laden** | Streaming über `datasets` (~4,6 Mio. Produkte) |
| **Stichprobe** | **100.000** Produkte per **Reservoir Sampling** (Algorithm R, Seed 42) |
| **Rohdaten** | `openfoodfacts_raw_sample.parquet` (+ Meta in `openfoodfacts_raw_sample_meta.json`) |
| **Modelldaten** | `openfoodfacts_model_data_nova.csv` (23.406), `openfoodfacts_model_data_nutriscore.csv` (28.924) |
| **Zielvariablen** | `nova_group` (1–4, Hauptproblem), `nutriscore_grade` (A–E, Vergleich) |
| **Features** | 9 Nährwerte/100 g + `additives_n` + `ingredients_n` + Kategorie |

### Relevante Nährstoff-Features

| Feature | Beschreibung |
|---|---|
| `energy-kcal_100g` | Energie (kcal/100 g) |
| `fat_100g` | Fett (g/100 g) |
| `saturated-fat_100g` | Gesättigte Fettsäuren (g/100 g) |
| `carbohydrates_100g` | Kohlenhydrate (g/100 g) |
| `sugars_100g` | Zucker (g/100 g) |
| `fiber_100g` | Ballaststoffe (g/100 g) |
| `proteins_100g` | Eiweiß (g/100 g) |
| `salt_100g` | Salz (g/100 g) |
| `sodium_100g` | Natrium (g/100 g) |

### Datenqualität

- Von 100.000 Rohprodukten haben **24.420** eine gültige NOVA-Gruppe und **29.703** einen gültigen Nutri-Score (A–E).
- Nach Plausibilitätsregeln und Label-Filter: **23.406** (NOVA) bzw. **28.924** (Nutri-Score) Produkte.
- Fehlende Werte werden erst **nach** dem Train/Test-Split per Median-Imputation in der Pipeline behandelt (kein Data Leakage).
- Labels fehlen nicht zufällig (stark länderabhängig) → faktisch europäisch geprägter Teildatensatz.

**NOVA-Verteilung (n = 23.406):** Gruppe 4 dominiert mit 64,7 %; Gruppe 2 nur 4,8 %.

## Notebook-Struktur

Hauptnotebook: [`PyTUMs_Final_Project_Niklas-2.ipynb`](PyTUMs_Final_Project_Niklas-2.ipynb)

Kapitelübersicht: [`Kapitel-Erklaerung.md`](Kapitel-Erklaerung.md)

| Kapitel | Inhalt |
|---|---|
| 1–2 | Domain Knowledge, Problem Statement & Hypothesen |
| 3–4 | Setup, Bibliotheken, Datenextraktion (Reservoir Sampling) |
| 5–8 | Datenverständnis, Zielvariablen, Feature-Extraktion, Cleaning |
| 9 | EDA (Verteilungen, Korrelationen, Nutri-Score × NOVA) |
| 10 | Preprocessing (stratifizierter Split, ML-Pipeline) |
| 11 | Clustering (K-Means, Vergleich mit Labels via ARI) |
| 12–14 | NOVA-Klassifikation, Modellvergleich, Hyperparameter-Tuning |
| 15 | Nutri-Score-Klassifikation (Vergleichsaufgabe) |
| 16 | Conclusion & Hypothesenbewertung |

Weitere Doku: [`Nova-Algorithmus_OpenFoodFacts.md`](Nova-Algorithmus_OpenFoodFacts.md) (wie OFF die `nova_group` vergibt).

## Setup

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt jupyter
jupyter notebook PyTUMs_Final_Project_Niklas-2.ipynb
```

Abhängigkeiten: `datasets`, `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `scipy`, `pyarrow`.

Optional: Hugging-Face-Token setzen (`HF_TOKEN`), um höhere Rate Limits beim Daten-Download zu erhalten. Die fertige Stichprobe liegt bereits als Parquet vor — ein erneutes Streaming ist nur nötig, wenn die Rohdaten neu gezogen werden sollen.
