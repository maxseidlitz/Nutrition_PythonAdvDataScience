# Kapitel-Erklärung zum Notebook

Einfache Übersicht zu `PyTUMs_Final_Project_Niklas-2.ipynb` — Kapitel für Kapitel.

**Kurz gesagt:** Das Projekt sagt voraus, zu welcher **NOVA-Gruppe** (Verarbeitungsgrad) ein Lebensmittel gehört — und vergleicht das mit der Vorhersage des **Nutri-Score** (Nährwert-Label A–E).

---

## 1. Domain Knowledge (Fachwissen)

**Worum geht es?** Grundlagen, die man vor der Analyse verstehen muss.

- **NOVA (Hauptthema):** Lebensmittel werden in 4 Gruppen eingeteilt — von unverarbeitet (1) bis ultra-verarbeitet (4). Wichtig sind vor allem Zutaten, Zusatzstoffe und Verarbeitung — nicht nur Nährwerte.
- **Nutri-Score (Vergleich):** Ampel von A (gut) bis E (schlecht), berechnet direkt aus Nährwerten wie Zucker, Fett und Salz.

**Erkenntnisse:**

- NOVA und Nutri-Score messen **zwei verschiedene Dimensionen**: Verarbeitungsgrad vs. Nährwertprofil.
- Ein Produkt kann **gut bewertet sein (A) und trotzdem ultra-verarbeitet (NOVA 4)** — das wird später in Kapitel 9 bestätigt.
- Wichtiger Hinweis: Die `nova_group`-Spalte in Open Food Facts wird **nicht manuell** vergeben, sondern per Regel-Algorithmus (Kategorie + Zusatzstoffe). Deshalb prüft Kapitel 13.1, ob das Modell „schummelt“.

---



## 2. Problem Statement, Project Goals & Hypotheses

**Worum geht es?** Die eigentliche Forschungsfrage und die Hypothesen.

**Hauptfrage:** Kann man die NOVA-Gruppe aus Nährwerten, Zusatzstoff-Anzahl und Zutaten-Anzahl vorhersagen?

**Vier Hypothesen (kurz):**


| Hypo | Inhalt                                                         | Ergebnis                        |
| ---- | -------------------------------------------------------------- | ------------------------------- |
| H1   | Mehr Zusatzstoffe/Zutaten → eher NOVA 4                        | Bestätigt                       |
| H2   | NOVA ist schwerer vorherzusagen als Nutri-Score                | Nicht bestätigt (mit Erklärung) |
| H3   | Nutri-Score und NOVA hängen zusammen, sind aber nicht dasselbe | Bestätigt                       |
| H4   | Clustering nach Nährwerten ähnelt eher Nutri-Score als NOVA    | Bestätigt (schwach)             |


**Erkenntnisse:**

- Das Projekt hat **zwei Zielvariablen**: NOVA (Hauptproblem) und Nutri-Score (Vergleich).
- Zusätzlich wird **Clustering** genutzt, um zu prüfen, ob Nährwerte allein sinnvolle Gruppen bilden.
- Die Hypothesen geben der gesamten Analyse eine klare Struktur — am Ende wird jede einzeln bewertet (Kapitel 16).

---



## 3. Project Setup and Library Imports

**Worum geht es?** Technische Vorbereitung.

- Pakete installieren (`requirements.txt`)
- Bibliotheken laden: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `datasets`
- Fester Zufallswert (`SEED = 42`) für reproduzierbare Ergebnisse

**Erkenntnisse:**

- Keine inhaltlichen Erkenntnisse — hier wird nur das technische Fundament gelegt.
- Der feste Seed sorgt dafür, dass Train/Test-Split, Modelltraining und Clustering bei jedem Durchlauf gleich bleiben.

---



## 4. Dataset Access and Data Extraction

**Worum geht es?** Woher die Daten kommen und wie die Stichprobe entsteht.

- Datenquelle: **Open Food Facts** über Hugging Face (~4,6 Mio. Produkte)
- Methode: **Reservoir Sampling** (echte Zufallsstichprobe aus dem gesamten Stream)
- Stichprobengröße: **100.000 Produkte**
- Ergebnis wird als `openfoodfacts_raw_sample.parquet` gespeichert

**Erkenntnisse:**

- Die alte Methode (nur die ersten Zeilen des Streams) hätte **verzerrte** Ergebnisse geliefert — z. B. zu viele gut gepflegte Produkte mit NOVA-Label.
- Reservoir Sampling ist langsamer (einmaliger Voll-Scan), liefert aber eine **repräsentative** Stichprobe.
- Die Stichprobe ist dokumentiert (`openfoodfacts_raw_sample_meta.json`: 4.610.617 gestreamte Zeilen → 100.000 gezogen).

---



## 5. Initial Data Understanding

**Worum geht es?** Erster Blick auf die Rohdaten.

- Wie viele Zeilen und Spalten gibt es?
- Sind wichtige Spalten da? (`nutriscore_grade`, `nutriments`, `nova_group`, …)
- Wie viele Werte fehlen bei NOVA und Nutri-Score?
- Hängt Nutri-Score-Verfügbarkeit vom **Land** ab?

**Erkenntnisse:**

- **100.000 Zeilen, 111 Spalten** — große, verschachtelte Rohdaten.
- **NOVA fehlt bei ~75,6 %** der Produkte (75.580 von 100.000) — viel Lücken in den Labels.
- **Nutri-Score hängt stark vom Land ab**: In Pakistan, Japan, Russland etc. fast 0 % Verfügbarkeit; in Frankreich/Deutschland deutlich höher.
- Die Daten sind **nicht zufällig fehlend** — nach dem Filtern auf gültige Labels entsteht faktisch ein europäisch geprägter Teildatensatz.

---



## 6. Target Variable Preparation: Nutri-Score & NOVA Group

**Worum geht es?** Die Zielvariablen vorbereiten (das, was die Modelle später vorhersagen sollen).

- **Nutri-Score:** Nur gültige Klassen A–E behalten (`unknown` usw. raus)
- **NOVA:** Gruppen 1–4 sauber extrahieren (verschiedene Spaltenformate werden vereinheitlicht)

**Erkenntnisse:**

- **Nutri-Score:** Von 100.000 bleiben **29.703** mit gültigem Label (A–E).
- **NOVA:** Von 100.000 haben nur **24.420** eine gültige Gruppe (1–4).
- NOVA-Verteilung in den Rohdaten: Gruppe 4 dominiert (15.549), Gruppe 2 ist selten (1.468).
- Die Normalisierungsfunktion war nötig, weil NOVA in Open Food Facts in **unterschiedlichen Formaten** gespeichert ist (Zahl vs. Tag-Liste).

---



## 7. Nutrient Feature Extraction

**Worum geht es?** Eingabevariablen (Features) aus den Rohdaten bauen.

- Nährwerte aus der verschachtelten Spalte `nutriments` in eigene Spalten ziehen (z. B. Zucker, Fett, Salz pro 100 g)
- Zusätzlich: `additives_n`, `ingredients_n`, Produktkategorie

**Features insgesamt:** 9 Nährwerte + 2 Zähler (Zusatzstoffe/Zutaten) + Kategorie

**Erkenntnisse:**

- Aus verschachtelten Listen/Dicts werden **normale Zahlen-Spalten** — ohne das geht kein Machine Learning.
- **Kategorien sind sehr unvollständig:** 60.600 von 100.000 Produkte haben keine Kategorie (`missing`); die Top-15-Kategorien decken den Rest ab.
- `additives_n` und `ingredients_n` sind besonders wichtig für NOVA — sie kommen direkt aus der NOVA-Definition.

---



## 8. Data Cleaning and Preparation

**Worum geht es?** Datenqualität verbessern und finale Datensätze bauen.

- Unrealistische Werte entfernen (z. B. negative Gramm, Zucker > Kohlenhydrate)
- Doppelte Produkte (gleicher Barcode) entfernen
- Fehlende Werte werden **später** im ML-Pipeline-Schritt behandelt (verhindert Data Leakage)
- Zwei finale Datensätze: `df_model_nova` und `df_model_nutri`

**Erkenntnisse:**

- **25.161 von 100.000 Zeilen** werden durch Plausibilitätsregeln entfernt (100.000 → 74.839).
- Nach Filter auf gültige Labels:
  - **NOVA-Datensatz: 23.406 Produkte**
  - **Nutri-Score-Datensatz: 28.924 Produkte**
- 0 Duplikate per Barcode (bei 100.000 aus 4,6 Mio. unwahrscheinlich).
- Die strengen Regeln (ganze Zeile löschen statt einzelne Werte) sind bewusst konservativ — lieber weniger, aber saubere Daten.

---



## 9. Exploratory Data Analysis (EDA)

**Worum geht es?** Muster in den Daten **verstehen**, bevor Modelle trainiert werden.

- Wie sind NOVA und Nutri-Score verteilt?
- Wie unterscheiden sich Nährwerte zwischen den Klassen?
- Wie stark korrelieren Features miteinander?
- **Kreuztabelle Nutri-Score × NOVA** → testet Hypothese H3

**Erkenntnisse:**

*Nutri-Score-Verteilung (n = 28.924):*

- E und D sind am häufigsten (27,5 % und 25,4 %), A und B sind unterrepräsentiert (14,4 % und 11,6 %).
- Die Stichprobe ist **leicht unbalanciert** — deshalb wird später gewichteter F1-Score statt Accuracy genutzt.

*NOVA-Verteilung (n = 23.406):*

- **Gruppe 4 (ultra-verarbeitet): 64,7 %** — stark dominierend.
- Gruppe 2 nur 4,8 % — die seltenste Klasse.

*Nährwerte nach NOVA-Gruppe:*

- Gruppe 4 hat im Schnitt **3,19 Zusatzstoffe** und **22,4 Zutaten** — deutlich mehr als Gruppe 1 (0,07 / 2,6).
- Bestätigt die NOVA-Logik: mehr Verarbeitung = mehr Zusatzstoffe.

*Kreuztabelle Nutri-Score × NOVA (H3):*

- Selbst bei **Nutri-Score A** sind **27 %** NOVA-Gruppe 4 (ultra-verarbeitert).
- Bei **Nutri-Score E** sind **77 %** NOVA-Gruppe 4.
- Diese Werte stimmen fast exakt mit publizierten Studien überein (z. B. 26,1 % für A in Romero Ferreiro et al., 2021).

*Korrelationen:*

- Salz/Sodium, Fett/gesättigtes Fett und Kohlenhydrate/Zucker sind stark korreliert — Baum-Modelle kommen damit besser klar als lineare.

---



## 10. Preprocessing for Machine Learning

**Worum geht es?** Daten für das Training technisch vorbereiten.

- **Train/Test-Split** (80 % / 20 %), stratifiziert nach Zielklasse
- **Pipeline:** fehlende Werte mit Median füllen, Zahlen skalieren, Kategorien one-hot encodieren
- Wichtig: Alles wird nur auf den **Trainingsdaten** angepasst, dann auf Test angewendet

**Erkenntnisse:**

- NOVA-Trainingsdaten: **18.628 Zeilen**, Test: **4.658 Zeilen**.
- Klassenverteilung bleibt im Train- und Testset gleich (z. B. 67 % NOVA 4 in beiden).
- Die Pipeline verhindert **Data Leakage** — im Gegensatz zur früheren Notebook-Version, die Median-Werte vor dem Split berechnet hatte.

---



## 11. Clustering (Unsupervised Learning)

**Worum geht es?** Produkte **ohne** Zielvariable in Gruppen clustern — nur nach Nährwerten.

- Methode: **K-Means** mit k = 8 (gewählt per Silhouette-Score)
- Frage: Ähneln die Cluster eher **Nutri-Score** oder **NOVA**? → testet H4

**Erkenntnisse:**

- Die ersten zwei PCA-Komponenten erklären **51,2 %** der Varianz — moderate Zusammenfassung.
- **ARI mit Nutri-Score: 0,082** (schwach positiv)
- **ARI mit NOVA: 0,030** (noch schwächer)
- H4 ist bestätigt: Nährwert-Cluster ähneln Nutri-Score etwas mehr als NOVA — aber beide Werte sind absolut niedrig.
- Clustering allein reicht **nicht** als Ersatz für die Labels — deshalb folgen die überwachten Modelle in Kapitel 12–15.

---



## 12. Machine Learning Models: NOVA Group (Main Problem)

**Worum geht es?** Sechs Modelle trainieren, um **NOVA** vorherzusagen (Hauptaufgabe).


| Modell                | Typ                 | F1 (gewichtet) |
| --------------------- | ------------------- | -------------- |
| **Random Forest**     | Ensemble (Bagging)  | **0,857**      |
| Gradient Boosting     | Ensemble (Boosting) | 0,841          |
| Decision Tree         | Baum                | 0,831          |
| K-Nearest Neighbors   | Nachbarn            | 0,816          |
| MLP (Neuronales Netz) | Deep Learning       | 0,815          |
| Logistic Regression   | Linear (Baseline)   | 0,808          |


**Erkenntnisse:**

- **Random Forest ist das beste Modell** für NOVA (F1 = 0,857).
- Lineare Modelle schneiden schlechter ab — die Beziehung zwischen Features und NOVA ist **nicht-linear** (z. B. viele Zusatzstoffe + viele Zutaten → eher Gruppe 4).
- Alle Modelle liegen über 80 % F1 — NOVA ist aus den vorhandenen Features **gut vorhersagbar**, zumindest mit Zusatzstoff-/Kategorie-Features.

---



## 13. Model Comparison (NOVA)

**Worum geht es?** Die sechs NOVA-Modelle vergleichen und Feature-Wichtigkeit analysieren.

- Hauptmetrik: **gewichteter F1-Score**
- **Feature Importance** (Random Forest)
- **Abschnitt 13.1:** Kontrollmodell **ohne** `additives_n` und Kategorie

**Erkenntnisse:**

*Feature Importance (Random Forest):*


| Feature               | Wichtigkeit   |
| --------------------- | ------------- |
| `ingredients_n`       | 22,1 %        |
| `additives_n`         | 19,4 %        |
| `energy-kcal_100g`    | 7,6 %         |
| (restliche Nährwerte) | jeweils < 7 % |


- Zusatzstoffe + Zutaten machen zusammen **~41,5 %** der Wichtigkeit aus → bestätigt **H1**.

*Leakage-Check (Kapitel 13.1):*


| Modell              | Mit allen Features | Nur Nährwerte | Verlust          |
| ------------------- | ------------------ | ------------- | ---------------- |
| Random Forest       | 0,857              | 0,744         | **−11,3 Punkte** |
| Logistic Regression | 0,808              | 0,596         | **−21,1 Punkte** |


- Ohne Zusatzstoffe/Kategorie fällt die Vorhersage deutlich — ein Teil der Modellgüte kommt von Features, die Open Food Facts selbst für die NOVA-Berechnung nutzt.
- Trotzdem bleibt F1 = 0,744 nur mit Nährwerten — also ist NOVA **teilweise** aus Nährwerten vorhersagbar.

---



## 14. Hyperparameter Tuning

**Worum geht es?** Die zwei besten Modelle noch einmal optimieren (GridSearchCV, 5-fach Kreuzvalidierung).


| Modell            | Baseline | Getunt | Änderung          |
| ----------------- | -------- | ------ | ----------------- |
| Random Forest     | 0,857    | 0,856  | ≈ 0 (kein Gewinn) |
| Gradient Boosting | 0,841    | 0,849  | **+0,8 Punkte**   |


**Erkenntnisse:**

- **Random Forest** war schon nahezu optimal — Tuning bringt nichts.
- **Gradient Boosting** profitiert vom Tuning, überholt Random Forest aber nicht (0,849 vs. 0,857).
- Random Forest bleibt das **beste Gesamtmodell** für NOVA.

---



## 15. Nutri-Score Classification (Secondary Problem)

**Worum geht es?** Dasselbe Spiel für **Nutri-Score** — als Vergleichsaufgabe (nur 9 Nährwert-Features, 3 Modelle).


| Modell              | F1 (gewichtet) |
| ------------------- | -------------- |
| **Random Forest**   | **0,827**      |
| Gradient Boosting   | 0,815          |
| Logistic Regression | 0,645          |


**Direkter Vergleich NOVA vs. Nutri-Score:**


| Problem                 | Bestes Modell | F1        |
| ----------------------- | ------------- | --------- |
| NOVA (Hauptproblem)     | Random Forest | **0,857** |
| Nutri-Score (Vergleich) | Random Forest | **0,827** |


**Erkenntnisse:**

- **H2 ist nicht bestätigt:** NOVA wird sogar etwas besser vorhergesagt als Nutri-Score.
- Grund: Das NOVA-Modell nutzt **mehr Features** (Zusatzstoffe, Kategorie), die sehr stark sind.
- Nutri-Score ist mit **nur Nährwerten** schon gut vorhersagbar (0,827) — das war zu erwarten, weil Nutri-Score direkt aus Nährwerten berechnet wird.
- Der faire Vergleich (nur Nährwerte für beide) steht in Kapitel 13.1: dort fällt NOVA auf 0,744 — **unter** Nutri-Score.

---



## 16. Conclusion

**Worum geht es?** Alles zusammenfassen und Grenzen benennen.

**Erkenntnisse — Hypothesen im Überblick:**


| Hypothese                         | Ergebnis            | Kurzfassung                                                        |
| --------------------------------- | ------------------- | ------------------------------------------------------------------ |
| H1: Zusatzstoffe/Zutaten → NOVA 4 | Bestätigt           | Wichtigste Features (41,5 % Importance)                            |
| H2: NOVA schwerer als Nutri-Score | Nicht bestätigt     | NOVA-F1 (0,857) > Nutri-F1 (0,827), aber unterschiedliche Features |
| H3: NOVA ≠ Nutri-Score            | Bestätigt           | 27 % der A-Produkte sind trotzdem NOVA 4                           |
| H4: Clustering ≈ Nutri-Score      | Bestätigt (schwach) | ARI 0,082 vs. 0,030                                                |


**Wichtigste Gesamterkenntnisse:**

1. **Nährwert und Verarbeitung sind verschiedene Dimensionen** — ein „gesundes“ Label sagt nichts über den Verarbeitungsgrad aus.
2. **Random Forest** ist das beste Modell für beide Aufgaben.
3. **Reservoir Sampling** war entscheidend — die alte Stichprobe war verzerrt und lieferte zu optimistische Ergebnisse.
4. **Limitationen:** Europäischer Datenbias, viele fehlende NOVA-Labels, mögliches Leakage durch Open-Food-Facts-NOVA-Algorithmus, starke Klassen-Ungleichgewichte (besonders NOVA Gruppe 2 mit nur 4,8 %).

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

*Bezug:* `PyTUMs_Final_Project_Niklas-2.ipynb` *· Team: Maximilian Seidlitz, Niklas Matusik, Bilal Mert, Bui Ngoc Minh Quach*