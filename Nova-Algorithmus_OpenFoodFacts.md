# NOVA-Zirkularität in Open Food Facts

Status: **im Notebook umgesetzt.** Abschnitt 1.1 enthält jetzt einen Caveat-Absatz, Abschnitt 13.1 ("Leakage Check: Control Model Without additives_n / Category") ein Kontrollmodell ohne `additives_n`/`main_category_grouped`, das Fazil (Abschnitt 16) verweist darauf. Dieses Dokument bleibt als ausführlicher Hintergrund bestehen.

**Offen:** Die neuen Zellen in Abschnitt 13.1 müssen einmal ausgeführt werden (Kernel neu starten oder ab dem Parquet-Cache weiterlaufen lassen), danach die `[TODO]`-Platzhalter in Abschnitt 13.1 und im Fazit mit den echten `drop_in_f1_points`-Werten füllen.

## Das Problem in Kürze

Das Projekt sagt `nova_group` mit `additives_n` und `main_category` vorher (Sektion 7/12-14 im Notebook). Beide Features sind aber Teil der Formel, mit der Open Food Facts `nova_group` selbst berechnet. Wir sagen die Formel mit ihren eigenen Eingaben vorher, statt einen unabhängigen Zusammenhang zu finden.

## Warum das ein Problem ist

- NOVA (Monteiro et al., 2016) ist als Konzept eine Einschätzung des Verarbeitungsgrads, ursprünglich manuell vergeben.
- Die Spalte `nova_group` in Open Food Facts ist aber **nicht** manuell vergeben, sondern von einem öffentlichen OFF-Algorithmus berechnet, der im Kern auf Additiv-Tags und Produktkategorie schaut (u. a. bestimmte Zusatzstoffe → Gruppe 4; Kategorien wie Öl/Zucker → Gruppe 2).
- Da unsere Modelle genau `additives_n` (Anzahl dieser Additive) und `main_category` als Features nutzen, bekommt das Modell einen Teil der Antwort direkt als Eingabe. Das ist klassisches **Target Leakage**.
- Praktische Folge: Modelle (v. a. Random Forest) werden auf NOVA verdächtig gut abschneiden — nicht weil NOVA gut aus Nährwerten vorhersagbar ist, sondern weil ein Teil der Zielformel selbst als Feature mitgegeben wurde.
- Das widerspricht der zentralen Prämisse des Projekts (Abschnitt 1/2: "NOVA ist das anspruchsvollere Target, weil es nicht direkt aus Nährwerten berechnet wird") — die Prämisse stimmt für NOVA als Konzept, nicht für die OFF-Spalte, die wir tatsächlich benutzen.

## Vorgeschlagene Lösung (noch nicht umgesetzt)

1. Im Text ehrlich benennen: OFF-`nova_group` ist algorithmisch aus Additiven/Kategorie abgeleitet, nicht unabhängig vergeben.
2. Zusätzliches Kontrollmodell rechnen: NOVA-Vorhersage **ohne** `additives_n` und `main_category`, nur auf den reinen Nährwert-Features (`energy-kcal_100g`, `fat_100g`, `saturated-fat_100g`, `carbohydrates_100g`, `sugars_100g`, `fiber_100g`, `proteins_100g`, `salt_100g`, `sodium_100g`).
3. Den Leistungsunterschied zwischen vollem Modell und Kontrollmodell explizit als Ergebnis diskutieren: Er beantwortet die eigentlich interessante Frage — "kann man den Verarbeitungsgrad allein am Nährwertprofil erkennen?" — sauberer als das Hauptmodell.

## Betroffene Notebook-Abschnitte (bei Umsetzung anzufassen)

- Abschnitt 1.1 / 2: Formulierung zur "Schwierigkeit" von NOVA als Target relativieren.
- Abschnitt 7 (`build_feature_frame`): Feature-Set für das Kontrollmodell separat definieren.
- Abschnitt 12-14 (Modelltraining/-vergleich für NOVA): zusätzliches Kontrollmodell + Vergleich ergänzen.
- Abschnitt 16 (Fazit): Limitation und Kontrollmodell-Ergebnis aufnehmen.

## Referenz

Der OFF-NOVA-Klassifizierungsalgorithmus ist öffentlich dokumentiert unter der Open-Food-Facts-Wiki-Seite zu NOVA-Gruppen; dort ist nachvollziehbar, welche Additiv-Tags und Kategorien welche Gruppe auslösen.
