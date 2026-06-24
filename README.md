# PyTUMs: Open Food Facts

Abschlussprojekt für den TUM-Kurs **Python and Advanced Data Science** (Dozent: Batuhan Can).

**Team:** Maximillian Seidlitz, Niklas Matusik, Bilal Mert, Bui Ngoc Minh Quach  
**Abgabe:** 21.07.2026

## Ziel

Vorhersage des **Nutri-Score** (Klassen A–E) anhand der Nährwertangaben pro 100 g. Die Analyse basiert auf Produktdaten aus der Open-Food-Facts-Datenbank.

## Datensatz

| Eigenschaft | Beschreibung |
|---|---|
| **Quelle** | [Open Food Facts Product Database](https://huggingface.co/datasets/openfoodfacts/product-database) auf Hugging Face |
| **Laden** | `load_dataset("openfoodfacts/product-database", split="food", streaming=True)` |
| **Stichprobe** | 10.000 Produkte per Streaming (`dataset.take(10000)`) |
| **Spalten** | 111 (u. a. Produktname, Kategorien, Marken, Zutaten, `nutriments`, `nutriscore_grade`) |
| **Zielvariable** | `nutriscore_grade` — Buchstaben A (beste) bis E (schlechteste) |
| **Features** | Nährwerte pro 100 g, extrahiert aus dem verschachtelten Feld `nutriments` |

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

### Datenqualität (aktueller Stand)

- Von 10.000 Rohdatensätzen haben **8.517** einen gültigen Nutri-Score (A–E); `unknown` und `not-applicable` werden entfernt.
- Nach Bereinigung unrealistischer Werte (negative Werte, Gramm-Features > 100 g, Energie > 1.000 kcal/100 g) verbleiben **6.092** Zeilen.
- Fehlende Werte werden per **Median-Imputation** (`SimpleImputer`) aufgefüllt.

**Klassenverteilung nach Bereinigung:**

| Nutri-Score | Anteil |
|---|---|
| A | 27,6 % |
| B | 11,2 % |
| C | 15,7 % |
| D | 18,2 % |
| E | 27,3 % |

## Aktueller Stand im Notebook (`PyTUMs.ipynb`)

Das Notebook ist in drei Abschnitte gegliedert:

### 1. Data Extraction
- Installation und Import der Bibliotheken (`datasets`, `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`)
- Streaming-Laden der Open-Food-Facts-Datenbank
- Erstellung eines 10.000-Zeilen-DataFrames
- Erste Inspektion: Form, Spalten, Verteilung der Nutri-Score-Klassen

### 2. Data Cleaning and Preparation
- Filterung auf gültige Nutri-Score-Klassen (`a`–`e`)
- Extraktion der Nährwerte aus `nutriments` in eigene Spalten
- Aufbau des Modell-Datasets `df_model` (9 Features + Zielvariable)
- Analyse fehlender Werte
- Entfernung unrealistischer Einträge
- Median-Imputation der verbleibenden Lücken

### 3. Exploratory Data Analysis (EDA)
- Visualisierung der Klassenverteilung (Countplot der Nutri-Score-Grades)

### Geplante nächste Schritte
- Weitere EDA (Korrelationen, Verteilungen der Nährstoffe pro Klasse)
- Train/Test-Split und Modelltraining (z. B. Random Forest, Logistic Regression)
- Modellbewertung (Accuracy, Confusion Matrix, Classification Report)

## Setup

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install datasets pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook PyTUMs.ipynb
```

Optional: Hugging-Face-Token setzen (`HF_TOKEN`), um höhere Rate Limits beim Daten-Download zu erhalten.
