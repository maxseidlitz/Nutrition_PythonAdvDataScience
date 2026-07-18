# Council-Transkript — 18.07.2026

**Ursprüngliche Frage:** „Sollte ich diese Adaption des Codes noch machen, oder ist unser Notebook schon gut genug für die 1,0?"

---

## Gerahmte Frage (an alle fünf Berater identisch)

Soll ein 4-köpfiges Master-Team (M.Sc. Management and Digital Technology, TU München) drei Tage vor Abgabe noch eine Code-Adaption an seinem Abschlussprojekt vornehmen — oder ist das Notebook bereits gut genug für die Bestnote 1,0?

**Zeitrahmen:** Samstag 18.07.2026, Abgabe Dienstag 21.07.2026. Vier Personen, alle arbeiten aktiv parallel am selben Notebook.

**Projekt:** Jupyter Notebook, Vorhersage von NOVA-Verarbeitungsgruppen (Hauptziel) und Nutri-Score (Nebenziel) aus Open-Food-Facts-Daten. Kurs „Python and Advanced Data Science", TUM.

**Verifizierter Stand:**
- 106 Zellen, läuft fehlerfrei durch (exec 1–53 lückenlos), 50/53 Code-Zellen mit Outputs
- Reservoir Sampling (Algorithm R) über 4,6 Mio Produkte → 100.000 gezogen
- Selbst entdeckte und dokumentierte Sampling-Verzerrung: 74,9 % gültige NOVA-Labels im alten Sample vs. 23,4 % bei echter Zufallsstichprobe
- Validierung: H3-Kreuztabelle trifft publizierte Literatur (27 % vs. 26,1 % bei Romero Ferreiro et al. 2021), verzerrtes Sample ergab 42 %
- Selbst entdeckte Target-Leakage: OFF berechnet `nova_group` aus Additiven + Kategorie, genau den Features. Kontrollmodell: RF −11,3 F1-Punkte (0,857 → 0,744), LR −21,1 Punkte, Gruppe-3-Recall kollabiert auf 0,32
- Leakage-Analyse rehabilitiert nachträglich Hypothese H2
- Cross-Nutrient-Cleaning, 6 ML-Modelle, GridSearchCV, Clustering (11.3–11.9), 4 Hypothesen, `%%time`, ausführliche Limitations

**Bekannte Schwächen:**
1. Klassenungleichgewicht gemessen, nicht behandelt (kein `class_weight`, kein Macro-F1)
2. Alle Vergleiche auf einem einzigen 80/20-Split ohne Varianzschätzung
3. README beschreibt drei Versionen alten Stand
4. Zwei `[TODO]`-Platzhalter (waren gefüllt, beim Merge verloren)

**Vorgeschlagene Adaption (~1 h):** (a) `class_weight="balanced"` + Macro-F1, (b) Learning-Curve-Zelle, (c) README + TODOs

**Risiko:** Drei dokumentierte Vorfälle still zerstörter Arbeit, vier parallele Editoren an einem 1,3-MB-Notebook.

---

## Runde 1 — Die fünf Berater

### Der Contrarian

Der fatale Fehler ist nicht im Notebook — er ist im Prozess. Drei dokumentierte Vorfälle stillen Arbeitsverlusts, der dritte war heute. Die Reaktion darauf ist: mehr parallel editieren. Ein 1,4-MB-JSON-Artefakt mit vier gleichzeitigen Schreibern wird behandelt, als wäre es Code mit Zeilendiff. Das Risiko der Adaption ist nicht „1 Stunde Arbeit" — es ist, dass ein Merge am Montagabend die Leakage-Analyse frisst, den besten Teil des Projekts, und es niemand merkt, weil 106 Zellen niemand nochmal liest.

Zweiter Fehler: Das Ergebnis der Learning Curve ist schon geschrieben („vermutlich zeigt die Kurve ein Plateau"). Wenn sie das nicht tut, steht drei Tage vor Abgabe eine Zelle im Notebook, die dem Dozenten empirisch recht gibt. Lokal ausführen, *bevor* entschieden wird, ob sie reinkommt.

Dritter Punkt: `class_weight="balanced"` kann die Hauptzahl senken. Dann steht ein schlechteres Modell in der Vergleichstabelle — und in der Verteidigung muss souverän erklärt werden, warum das Fortschritt ist. Können das alle vier?

Die Notenschätzung ist Fiktion. 1,7 → 1,0 durch eine Stunde sklearn? Der Abstand zwischen 1,3 und 1,0 ist in deutschen Gruppenprojekten fast nie technisch, sondern Präsentation, Kohärenz und ob der Dozent seine Feedback-Punkte adressiert sieht.

**Priorität:** TODOs füllen. README. Notebook einfrieren, ein Schreiber. Dann, falls Zeit übrig, (a) und (b) — von einer Person, in einem Branch.

### Der First-Principles-Denker

Die falsche Frage wird gestellt. „Adaption ja oder nein" setzt voraus, dass die Note eine Funktion des Notebook-Inhalts ist. Ist sie nicht. Sie ist eine Funktion davon, was ein Mensch in 40 Minuten Lesezeit wahrnimmt — multipliziert mit der Wahrscheinlichkeit, dass das Artefakt am Dienstag noch intakt ist.

Die vier Schwächen gehören in zwei verschiedene Kategorien. **Integritätsdefekte (3, 4):** Ein `[TODO]` und ein README, das ein anderes Projekt beschreibt, sind keine methodischen Lücken — sie sind Beweise für Unsauberkeit. Ein Korrektor, der ein TODO sieht, liest den Rest misstrauischer. Das kostet besonders viel, weil der ganze Notenmotor „wir sind rigoros und selbstkritisch" heißt. **Verfeinerungen (1, 2):** machen ein 1,3-Notebook zu einem 1,0-Notebook — aber nur, wenn der Leser so weit kommt.

Die zu entfernende Annahme ist eine andere: „Vier Leute editieren parallel dasselbe Notebook." Das ist keine Randbedingung, das ist die Ursache aller drei Datenverluste. Kein Modellierungsproblem — ein Concurrency-Problem, das als gegeben behandelt wird.

Erste Prinzipien: ab jetzt ein Schreiber. Die anderen drei an README, Verteidigungsvorbereitung und Review. Erst Integrität herstellen, committen, taggen. Dann die eine Stunde Adaption, sequenziell, mit Commit nach jeder Zelle.

### Der Expansionist

Das Team unterschätzt, was es in der Hand hat. Diskutiert wird über eine 1,0 — das eigentliche Aufwärtspotenzial liegt gar nicht in der Note. Zwei Dinge wurden produziert, die in Master-Abschlussprojekten praktisch nie vorkommen: eine selbst entdeckte und quantifizierte Sampling-Verzerrung, deren Korrektur die Ergebnisse auf 0,9 Prozentpunkte an publizierte Literatur heranführt — und eine saubere Target-Leakage-Analyse mit Kontrollmodell, die eine eigene, scheinbar widerlegte Hypothese rehabilitiert. Das ist der Kern eines Workshop-Papers oder mindestens eines Blogposts, den die Open-Food-Facts-Community zitieren würde.

Die Learning Curve ist der unterbewertetste Zug: Sie beantwortet nicht nur „more data would be better", sie widerlegt den Dozenten empirisch mit seiner eigenen Priorität. Ein Plateau bei 100k zeigt, dass der Engpass Label-Qualität ist, nicht Stichprobengröße. Das dreht in der Verteidigung die Rollen um.

Macro-F1 ist mehr als Kosmetik: Es macht sichtbar, dass NOVA-Gruppe 3 der eigentlich schwierige Fall ist — und Gruppe 3 ist genau die, die unter Leakage-Entfernung kollabiert (Recall 0,32). Zwei Befunde zeigen auf denselben Punkt; verbindet man sie, wird aus zwei Einzelbeobachtungen eine These.

Framing: nicht „Wir sagen NOVA vorher", sondern „Wir haben untersucht, ob NOVA überhaupt vorhersagbar ist — und gefunden, dass die scheinbare Vorhersagbarkeit größtenteils Konstruktionsartefakt ist."

Vier Köpfe nutzen: A macht (a), B (b), C README + TODOs, D ist ausschließlich Merge-Wächter.

### Der Außenstehende

**Hat jemand die Bewertungsrubrik gelesen?** In der gesamten Darstellung kommt kein Wort über die tatsächlichen Bewertungskriterien vor. „1,7–2,0" und „potenziell 1,0–1,3" sind erfundene Zahlen. Optimiert wird gegen ein Modell des Dozenten im eigenen Kopf, nicht gegen das Dokument. Die einzigen echten Signale sind vier Sätze früheres Feedback — und drei davon sind bereits erfüllt.

**Was sieht der Korrektor in den ersten 90 Sekunden?** Er öffnet das Repo: eine README, die ein anderes Projekt beschreibt. Dann das Notebook: zwei `[TODO]`. Das ist der komplette Ersteindruck — veraltet und unfertig. Die Leakage-Analyse sitzt bei Zelle 70, ein Macro-F1 noch tiefer. 60 Minuten Modellarbeit werden diskutiert, während das Schaufenster kaputt ist. Punkt (c) ist keine Adaption, das ist Schadensbehebung.

**Warum editieren vier Leute dieselbe Datei?** Drei Vorfälle sind keine Pechsträhne, das ist ein Systemzustand — für einen Außenstehenden das größte Risiko im ganzen Brief.

**Und die Frage, die niemand stellt:** Erklärt das Notebook einem Fremden, was NOVA ist und warum die Leakage-Entdeckung wichtig ist — oder setzt es voraus, dass der Leser die letzten sechs Wochen mitgemacht hat? Der Fluch des Wissens trifft genau hier: Die beste Arbeit ist die, die am meisten Vorwissen zum Verstehen braucht.

### Der Umsetzer

Machen. Aber nicht heute alles gleichzeitig, und nicht zu viert am selben File.

**Heute (Samstag), 20 Minuten, eine Person, sonst rührt niemand das Notebook an:**
1. `git checkout -b freeze-abgabe` + Tag auf den aktuellen fehlerfreien Stand. Ohne diesen Schritt passiert gar nichts anderes.
2. Die zwei TODOs füllen. Keine Adaption — Schadensbehebung an einem sichtbaren Loch, höchste Priorität.
3. README. 15 Minuten, null technisches Risiko, kein Notebook-Merge.

**Sonntag, Adaption (a) + (b) von EINER Person in EINEM Sitzungsblock:**
- (a) `class_weight="balanced"` als zusätzliche Zeile, nicht als Ersatz. `f1_macro` in die bestehende Tabelle. Läuft das nach 45 Minuten nicht sauber: abbrechen, Freeze-Branch, fertig.
- (b) `learning_curve` mit `train_sizes=np.linspace(0.1,1.0,5)`, `cv=3`, `n_jobs=-1`. Auf 100k Zeilen mit RF teuer — vorher auf 20k testen. Einzige Zelle mit Laufzeitrisiko.
- Die anderen drei arbeiten an der mündlichen Verteidigung, nicht am Notebook.

**Gestrichen:** Punkt 2 (Varianzschätzung). Kein sauberer Ein-Stunden-Pfad; Repeated-CV über sechs Modelle mit GridSearch sprengt die Laufzeit und zwingt dazu, die halbe Tabelle neu zu erzählen. Zwei Sätze im Limitations-Abschnitt.

**Montag:** kein Code mehr. Einmal Restart & Run All, Outputs prüfen, committen. Dienstag nur Abgabe.

---

## Runde 2 — Peer-Review (anonymisiert)

**Anonymisierungs-Mapping:** A = Der Außenstehende · B = Der Contrarian · C = Der Umsetzer · D = Der Expansionist · E = Der First-Principles-Denker

### Review 1 (Standard)
1. **Stärkste: C.** Einzige mit ausführbarem Zeitplan, Abbruchkriterium, Freeze-Tag als Vorbedingung, konkreten Learning-Curve-Parametern inklusive Laufzeitwarnung — und streicht Schwäche 2 begründet. E diagnostiziert am schärfsten, C handelt.
2. **Blinder Fleck: D.** Verteilt vier parallele Editoren auf ein Notebook, dessen Concurrency bereits dreimal Arbeit vernichtet hat; „Merge-Wächter" ist Symptomkosmetik. Verkauft das Plateau als gegeben, obwohl ungemessen.
3. **Alle übersehen:** Abgabeformat und Reproduzierbarkeitsbedingungen. Läuft das Notebook auf dem Rechner des Korrektors (Datenpfade, requirements, Seeds, Laufzeit)? Zweitens: Rollback-Test — ein Tag, der nie verifiziert wurde, ist keine Absicherung.

### Review 2 (eigenständiges Urteil)
1. **Stärkste: C.** Einzige mit Reihenfolge, Rollen und Abbruchkriterium; sieht als einzige den Laufzeit-Fallstrick von (b) und streicht aktiv Scope. B und E diagnostizieren schärfer, produzieren aber keinen Plan.
2. **Blinder Fleck: D.** Diagnostiziert nichts vom Prozessrisiko und verteilt dann parallele Arbeit an alle vier — exakt die Ursache der Datenverluste. Workshop-Paper-Ambition ist am Dienstag irrelevant.
3. **Alle übersehen:** Die TODO-Inhalte existierten bereits — `git log -S` / `git show` könnte sie zurückholen, niemand schlägt Wiederherstellung statt Neuschreiben vor. Die technische Ursache der Merge-Verluste ist eingebettetes Output-JSON; `nbstripout`/`nbdime` kostet 5 Minuten und entschärft strukturell, während alle fünf es nur sozial behandeln. Und: schon Sonntag eine abgabefähige Version hochladen, dann ist jeder Montagsschaden folgenlos.

### Review 3 (skeptisch, Fokus Belegbarkeit)
1. **Stärkste: C.** Einzige vollständig handlungsfähige Antwort; streicht (2) mit Begründung statt Vollständigkeitsritual.
2. **Blinder Fleck: D.** Rhetorisch am stärksten, sachlich am schwächsten. Behauptungen ohne Beleg („Workshop-Paper", „Community würde zitieren", „Plateau zeigt Label-Qualität" — das Plateau ist noch nicht gerechnet). Empfiehlt ausgerechnet parallele Editoren. *(Anmerkung des Chairmans: Der Vorwurf, „Recall 0,32" sei erfunden, ist nachprüfbar falsch — die Zahl steht real im Output der Leakage-Check-Zelle.)* Auch B übertreibt: „nahezu 100 %" ist Dramatik, nicht Basisrate — sein Kern (Learning Curve erst lokal prüfen) ist aber der einzige echte Falsifikationstest im Council.
3. **Alle übersehen:** Abgabeformat (Datei, Repo-Link, PDF?), Rendering auf GitHub, Reproduzierbarkeit von Null. Und: Die Rubrik-Antwort muss *vor* jeder Priorisierung feststehen.

### Review 4 (Fokus deutsche Bewertungsrealität)
1. **Stärkste: C.** Einzige mit Zeitplan, Abbruchkriterium und aktivem Scope-Schnitt. B liefert die realistischste Notendiagnose — der Abstand 1,3 → 1,0 ist Kohärenz und Verteidigung, nicht sklearn — hat aber keinen Plan. Kombination: C-Ablauf + B-Erwartungsmanagement.
2. **Blinder Fleck: D.** Parallele Arbeit trotz dokumentierter Verlustursache; verwechselt Publikationswert mit Notenwert. Ein Dozent belohnt keine Workshop-Paper-Ambition, sondern erfüllte Rubrikpunkte.
3. **Alle übersehen:** Wie viel Prozent trägt das Notebook überhaupt (Notebook *plus* Präsentation, oft separat gewichtet)? Wird ein HTML/PDF-Export abgegeben — dann entscheidet der Export, und „Restart & Run All" erzeugt einen kompletten Output-Diff, der Merge-Konflikte garantiert. Rückfrage beim Dozenten zu „more data" ist billiger als jede Learning Curve. Und: Die 1,0 ist oft faktisch kontingentiert.

### Review 5 (Fokus Umsetzbarkeit unter Zeitdruck)
1. **Stärkste: C.** Einzige mit Zeitfenstern, Personenzahl und Abbruchkriterium; streicht Punkt 2 mit korrekter Begründung.
2. **Blinder Fleck: D.** Bester Einzelgedanke des Councils (Macro-F1 ↔ NOVA-3-Kollaps), aber drei parallele Editoren an genau der Datei mit dem Concurrency-Problem. „Merge-Wächter ist kein Lock."
3. **Alle übersehen:** Git-Rekonstruktion des TODO-Inhalts statt Neuschreiben. `class_weight` erzwingt ein vollständiges Re-Run aller abhängigen Zellen — kein Ein-Zeilen-Eingriff. `nbdime`/`nbstripout` entschärft Merge-Konflikte technisch statt per Disziplin.

---

## Nachträgliche Faktenprüfung (Chairman)

Nach der Review-Runde wurden die aufgeworfenen Punkte am realen Repository verifiziert:

| Prüfung | Ergebnis |
|---|---|
| TODO-Inhalt aus Git wiederherstellbar? | **Nein** — `git log -S "11.3 F1 points"` liefert nichts; der Text wurde nie committet. Er liegt aber vollständig im Sitzungsverlauf vor und kann direkt neu eingesetzt werden. |
| Parquet-Cache im Repository? | **Nein** — `.gitignore:3:*.parquet` schließt ihn aus. Getrackt sind nur Notebook, README, `requirements.txt`, `Kapitel-Erklaerung.md`, `Nova-Algorithmus_OpenFoodFacts.md` und die Metadaten-JSON. |
| Konsequenz für den Korrektor | Beim Klonen + „Run All" schlägt `os.path.exists(RAW_SAMPLE_PATH)` fehl → `else`-Zweig → **60–90 Minuten Streaming von 4,6 Mio Produkten**, mit Internet- und HuggingFace-Abhängigkeit. |
| Notebook-Größe / Rendering | 1,3 MB — unterhalb der GitHub-Rendergrenze (~10 MB), Anzeige unproblematisch. |
| Aktueller Branch | `Minh_Section_11_Integration` (nicht `main`) — drei neue Commits von Teammitgliedern seit der letzten Prüfung. |

---

## Chairman-Synthese

### Wo der Council übereinstimmt
Vier von fünf Beratern kommen unabhängig auf denselben Kern: Das Risiko liegt nicht im Notebook-Inhalt, sondern im Arbeitsprozess. Integritätsdefekte (TODOs, README) schlagen Verfeinerungen (Macro-F1, Varianz). Erst einfrieren, dann anfassen. Ein Schreiber am Notebook. Die Adaption ist machbar — aber sequenziell und mit Abbruchkriterium.

### Wo der Council sich streitet
**Learning Curve:** Expansionist sieht den stärksten Zug (widerlegt den Dozenten mit seiner eigenen Priorität), Contrarian sieht eine Falle (Ergebnis vorweggenommen). Auflösung: Contrarian gewinnt prozedural, Expansionist inhaltlich — erst lokal rechnen, dann entscheiden.
**Notenschätzung:** Contrarian nennt sie Fiktion, Außenstehender fragt nach der ungelesenen Rubrik, ein Reviewer ergänzt die faktische Kontingentierung der 1,0.
**Vier Köpfe parallel:** Expansionist dafür, alle fünf Peer-Reviewer dagegen.

### Blinde Flecken aus der Review-Runde
Abgabeformat · Reproduzierbarkeit beim Korrektor (bestätigt: Datengrundlage fehlt im Repo) · unverifizierter Freeze-Tag · `nbstripout`/`nbdime` als struktureller statt sozialer Fix · `class_weight` erzwingt vollständiges Re-Run · Sonntag bereits abgabefähig hochladen · Rückfrage beim Dozenten statt Learning Curve.

### Empfehlung
**Ja zur Adaption — aber erst an Position vier.** Das Notebook ist inhaltlich bereits auf 1,0-Niveau; was fehlt, ist nicht Methodik, sondern Auslieferung.

1. Reproduzierbarkeit sichern (Parquet aus `.gitignore` nehmen oder dokumentierten Download-Pfad ergänzen)
2. TODOs füllen (Text liegt vollständig vor)
3. README aktualisieren
4. Dann, mit Restzeit: `class_weight` + Macro-F1 und Learning Curve — eine Person, sequenziell, Abbruch nach 45 Minuten

Varianzschätzung wird gestrichen; zwei Sätze im Limitations-Abschnitt decken vollständig ab.

### Das eine, was zuerst zu tun ist
Den aktuellen fehlerfrei durchlaufenden Stand einfrieren (Branch/Tag) — und aus diesem Tag einmal frisch in ein leeres Verzeichnis klonen und prüfen, ob das Notebook dort lauffähig ist. Dieser eine Test beantwortet gleichzeitig die Freeze-Frage und die Reproduzierbarkeitsfrage.
