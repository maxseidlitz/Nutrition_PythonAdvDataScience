# Kapitel-Erklärung zum Notebook

Einfache Übersicht zu `PyTUMs_Final_Project_Niklas-2.ipynb` — Kapitel für Kapitel.

**Kurz gesagt:** Das Projekt sagt voraus, zu welcher **NOVA-Gruppe** (Verarbeitungsgrad) ein Lebensmittel gehört — und vergleicht das mit der Vorhersage des **Nutri-Score** (Nährwert-Label A–E).

---

## 1. Domain Knowledge (Fachwissen)

**Worum geht es?** Grundlagen, die man vor der Analyse verstehen muss.

- **NOVA (Hauptthema):** Lebensmittel werden in 4 Gruppen eingeteilt — von unverarbeitet (1) bis ultra-verarbeitet (4). Wichtig sind vor allem Zutaten, Zusatzstoffe und Verarbeitung — nicht nur Nährwerte.
- **Nutri-Score (Vergleich):** Ampel von A (gut) bis E (schlecht), berechnet direkt aus Nährwerten wie Zucker, Fett und Salz.

**Warum wichtig?** NOVA und Nutri-Score messen unterschiedliche Dinge. Ein Produkt kann gut bewertet sein (A), aber trotzdem stark verarbeitet (NOVA 4).

---

## 2. Problem Statement, Project Goals & Hypotheses

**Worum geht es?** Die eigentliche Forschungsfrage und die Hypothesen.

**Hauptfrage:** Kann man die NOVA-Gruppe aus Nährwerten, Zusatzstoff-Anzahl und Zutaten-Anzahl vorhersagen?

**Vier Hypothesen (kurz):**

| Hypo | Inhalt |
|------|--------|
| H1 | Mehr Zusatzstoffe/Zutaten → eher NOVA 4 |
| H2 | NOVA ist schwerer vorherzusagen als Nutri-Score |
| H3 | Nutri-Score und NOVA hängen zusammen, sind aber nicht dasselbe |
| H4 | Clustering nach Nährwerten ähnelt eher Nutri-Score als NOVA |

---

## 3. Project Setup and Library Imports

**Worum geht es?** Technische Vorbereitung.

- Pakete installieren (`requirements.txt`)
- Bibliotheken laden: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `datasets`
- Fester Zufallswert (`SEED = 42`) für reproduzierbare Ergebnisse

**Einfach gesagt:** Hier wird das „Werkzeug“ bereitgestellt, bevor mit Daten gearbeitet wird.

---

## 4. Dataset Access and Data Extraction

**Worum geht es?** Woher die Daten kommen und wie die Stichprobe entsteht.

- Datenquelle: **Open Food Facts** über Hugging Face
- Die komplette Datenbank ist zu groß → es wird eine **Stichprobe** gezogen
- Methode: **Reservoir Sampling** (echte Zufallsstichprobe aus dem gesamten Stream)
- Stichprobengröße: **100.000 Produkte**
- Ergebnis wird als `openfoodfacts_raw_sample.parquet` gespeichert (muss nicht jedes Mal neu geladen werden)

**Einfach gesagt:** Hier holt man sich die Rohdaten und speichert eine feste, zufällige Teilmenge ab.

---

## 5. Initial Data Understanding

**Worum geht es?** Erster Blick auf die Rohdaten.

- Wie viele Zeilen und Spalten gibt es?
- Sind wichtige Spalten da? (`nutriscore_grade`, `nutriments`, `nova_group`, …)
- Wie viele Werte fehlen bei NOVA und Nutri-Score?
- Hängt Nutri-Score-Verfügbarkeit vom **Land** ab?

**Einfach gesagt:** Hier schaut man sich an, was überhaupt in den Daten steht — bevor man aufräumt.

---

## 6. Target Variable Preparation: Nutri-Score & NOVA Group

**Worum geht es?** Die Zielvariablen vorbereiten (das, was die Modelle später vorhersagen sollen).

- **Nutri-Score:** Nur gültige Klassen A–E behalten (`unknown` usw. raus)
- **NOVA:** Gruppen 1–4 sauber extrahieren (verschiedene Spaltenformate werden vereinheitlicht)

**Einfach gesagt:** Hier werden die „Antworten“ für das Machine Learning festgelegt und bereinigt.

---

## 7. Nutrient Feature Extraction

**Worum geht es?** Eingabevariablen (Features) aus den Rohdaten bauen.

- Nährwerte aus der verschachtelten Spalte `nutriments` in eigene Spalten ziehen (z. B. Zucker, Fett, Salz pro 100 g)
- Zusätzlich: `additives_n`, `ingredients_n`, Produktkategorie

**Features insgesamt:** 9 Nährwerte + 2 Zähler (Zusatzstoffe/Zutaten) + Kategorie

**Einfach gesagt:** Aus unübersichtlichen Rohdaten werden normale Zahlen-Spalten fürs Modell.

---

## 8. Data Cleaning and Preparation

**Worum geht es?** Datenqualität verbessern und finale Datensätze bauen.

- Unrealistische Werte entfernen (z. B. negative Gramm, Zucker > Kohlenhydrate)
- Doppelte Produkte (gleicher Barcode) entfernen
- Fehlende Werte werden **später** im ML-Pipeline-Schritt behandelt (nicht vor dem Train/Test-Split — verhindert Data Leakage)
- Zwei finale Datensätze:
  - `df_model_nova` → NOVA vorhersagen
  - `df_model_nutri` → Nutri-Score vorhersagen

**Einfach gesagt:** Hier wird aus „chaotischen“ Daten ein sauberer Datensatz fürs Training.

---

## 9. Exploratory Data Analysis (EDA)

**Worum geht es?** Muster in den Daten **verstehen**, bevor Modelle trainiert werden.

- Wie sind NOVA und Nutri-Score verteilt?
- Wie unterscheiden sich Nährwerte zwischen den Klassen?
- Wie stark korrelieren Features miteinander?
- **Kreuztabelle Nutri-Score × NOVA** → testet Hypothese H3

**Einfach gesagt:** Hier werden die Daten visualisiert und beschrieben — „Was steckt drin?“

---

## 10. Preprocessing for Machine Learning

**Worum geht es?** Daten für das Training technisch vorbereiten.

- **Train/Test-Split** (80 % / 20 %), stratifiziert nach Zielklasse
- **Pipeline:** fehlende Werte mit Median füllen, Zahlen skalieren, Kategorien one-hot encodieren
- Wichtig: Alles wird nur auf den **Trainingsdaten** angepasst, dann auf Test angewendet

**Einfach gesagt:** Hier wird aus den sauberen Daten ein fairer Trainings- und Testdatensatz.

---

## 11. Clustering (Unsupervised Learning)

**Worum geht es?** Produkte **ohne** Zielvariable in Gruppen clustern — nur nach Nährwerten.

- Methode: **K-Means** (Anzahl Cluster per Silhouette-Score gewählt)
- Frage: Ähneln die Cluster eher **Nutri-Score** oder **NOVA**? → testet H4
- Ergebnis: Leichte Übereinstimmung mit Nutri-Score, kaum mit NOVA

**Einfach gesagt:** „Finden Nährwerte allein schon sinnvolle Produktgruppen — und passen die zu unseren Labels?“

---

## 12. Machine Learning Models: NOVA Group (Main Problem)

**Worum geht es?** Sechs Modelle trainieren, um **NOVA** vorherzusagen (Hauptaufgabe).

| Modell | Typ |
|--------|-----|
| Logistic Regression | Linear (Baseline) |
| K-Nearest Neighbors | Nachbarn im Feature-Raum |
| Decision Tree | Entscheidungsbaum |
| Random Forest | Ensemble (Bagging) |
| Gradient Boosting | Ensemble (Boosting) |
| MLP | Neuronales Netz |

**Einfach gesagt:** Hier wird aus den Features die NOVA-Gruppe vorhergesagt — mit verschiedenen Algorithmen.

---

## 13. Model Comparison (NOVA)

**Worum geht es?** Die sechs NOVA-Modelle vergleichen.

- Hauptmetrik: **gewichteter F1-Score** (wegen unbalancierter Klassen)
- Zusätzlich: Accuracy, Precision, Recall, Confusion Matrix
- **Feature Importance:** Welche Variablen sind am wichtigsten? → bestätigt H1 (Zusatzstoffe/Zutaten)
- **Abschnitt 13.1:** Kontrollmodell **ohne** `additives_n` und Kategorie — prüft, ob das Modell „schummelt“ (Data Leakage durch Open-Food-Facts-eigene NOVA-Regeln)

**Einfach gesagt:** Welches Modell ist am besten — und worauf basiert die Vorhersage wirklich?

---

## 14. Hyperparameter Tuning

**Worum geht es?** Die zwei besten Modelle noch einmal optimieren.

- **GridSearchCV** mit 5-fach Kreuzvalidierung
- Optimiert: Random Forest und Gradient Boosting
- Vergleich: Baseline vs. getunt

**Einfach gesagt:** Feintuning der besten Modelle — bringt bei Random Forest kaum etwas, bei Gradient Boosting etwas mehr.

---

## 15. Nutri-Score Classification (Secondary Problem)

**Worum geht es?** Dasselbe Spiel für **Nutri-Score** — als Vergleichsaufgabe.

- Nur die 9 Nährwert-Features (keine Zusatzstoffe/Kategorie)
- Drei Modelle: Logistic Regression, Random Forest, Gradient Boosting
- Direkter Vergleich: NOVA vs. Nutri-Score → testet H2

**Einfach gesagt:** Kann man Nutri-Score aus Nährwerten vorhersagen — und ist das leichter als NOVA?

**Ergebnis (überraschend):** NOVA-Modell schneidet etwas besser ab, weil es mehr Features nutzt.

---

## 16. Conclusion

**Worum geht es?** Alles zusammenfassen.

- Alle vier Hypothesen bewerten (H1 ✓, H2 ✗ mit Erklärung, H3 ✓, H4 ✓)
- Bestes Modell: **Random Forest** für beide Aufgaben
- Wichtige Limitationen: Stichprobengröße, fehlende Werte, Länderbias, mögliches Leakage bei NOVA
- Fazit: Nährwert-Labels und Verarbeitungsgrad sind **verschiedene Dimensionen** der Lebensmittelbewertung

**Einfach gesagt:** Was haben wir gelernt — und was sind die Grenzen der Analyse?

---

## Grober Ablauf auf einen Blick

```
1–2   Verstehen (Was ist NOVA? Was wollen wir?)
  ↓
3–4   Setup + Daten laden
  ↓
5–8   Daten verstehen, aufräumen, Features bauen
  ↓
9     EDA (Muster entdecken)
  ↓
10    ML-Vorbereitung (Split, Pipeline)
  ↓
11    Clustering (ohne Labels)
  ↓
12–14 NOVA-Modelle trainieren, vergleichen, tunen
  ↓
15    Nutri-Score als Vergleich
  ↓
16    Fazit
```

---

*Bezug: `PyTUMs_Final_Project_Niklas-2.ipynb` · Team: Maximilian Seidlitz, Niklas Matusik, Bilal Mert, Bui Ngoc Minh Quach*
