---
title: "Vollständiges Sprech- und Begriffsskript"
subtitle: "Verbesserte Objekterkennung auf Formularen mit synthetischen Daten und Self-Training"
author: "Okan Yakin"
lang: de-DE
geometry: margin=1.9cm
fontsize: 10pt
colorlinks: true
toc: true
toc-depth: 1
---

# Sprachregel für den Vortrag

Die erklärenden Sätze bleiben deutsch. Etablierte Fachbegriffe und offizielle Modellnamen bleiben englisch. Dadurch entsteht kein unnötiges Denglisch, die fachliche Bezeichnung bleibt aber korrekt.

**Bewusst englisch bleiben:**

- Faster R-CNN, ResNet-50, Feature Pyramid Network, Region Proposal Network
- Feature, Feature Map, Region Proposal, Anchor Box, Objectness Score
- RoI Align, Classification, Box Regression, Detection, Prediction, Non-Maximum Suppression
- Bounding Box, Ground Truth, Teacher, Student, Pseudo-Label
- Self-Training, Multi-View Prediction, Confidence Score, Adaptive Threshold, Loss, Refinement
- Precision, Recall, F1, Intersection over Union
- Optimizer, Stochastic Gradient Descent, Learning Rate, Momentum, Weight Decay
- Hyperparameter Search, Checkpoint, Baseline, Ablation, Pseudo-only, Random Seed, Validation Set

**Deutsch bleiben:**

- Daten statt „Data“, wenn nicht gerade `Labeled Data` oder `Unlabeled Data` als ASTOD-Bezeichnung erklärt wird
- Erkennung statt „Detection“, wenn nur allgemein über das Ergebnis gesprochen wird
- Vorhersage statt „Prediction“, nachdem der Fachbegriff einmal eingeführt wurde
- Trainingsrunde statt „Training Iteration“, wenn eine vollständige Teacher-Student-Runde gemeint ist
- Realitätslücke statt „Domain Gap“
- Hyperparameter Search statt „Sweep“, weil dies die gewünschte fachliche Bezeichnung ist

# Folie 1 – Titel

## Das sage ich

- Guten Tag, mein Name ist Okan Yakin.
- In dieser Arbeit untersuchen wir die automatische Erkennung von Textregionen auf Formularen.
- Dafür werden synthetische Daten und Self-Training kombiniert.
- Die zentrale Frage ist, wie reale Formulare genutzt werden können, ohne für jede Seite neue Bounding Boxes manuell einzuzeichnen.

## Begriffe

- **Objekterkennung / Object Detection:** Ein Modell bestimmt, welche relevanten Objekte in einem Bild vorkommen und wo sie liegen. Die Position wird hier durch eine Bounding Box beschrieben.
- **Textregion:** Zusammenhängender Bildbereich, der Text enthält. In dieser Arbeit ist jede Textregion ein zu erkennendes Objekt.
- **Synthetische Daten:** Künstlich erzeugte Trainingsbilder. Ihre Annotationen sind automatisch bekannt, weil Text und Boxen gemeinsam erzeugt werden.
- **Self-Training:** Trainingsverfahren, bei dem ein bereits trainiertes Modell automatische Labels für nicht annotierte Daten erzeugt. Ein neues Modell lernt anschließend aus sicheren und automatisch erzeugten Labels.

## Nicht falsch sagen

- Self-Training bedeutet nicht, dass das Modell ohne Daten lernt.
- Es lernt weiterhin aus Trainingsbildern. Ein Teil der Labels stammt jedoch vom Teacher statt von Menschen.

# Folie 2 – Motivation: Formulare sind überall

## Das sage ich

- Formulare werden im Gesundheitswesen, im Finanzbereich und in der Verwaltung täglich verarbeitet.
- Die Beispiele zeigen, wie stark sich Layout, Schrift, Scanqualität und Handschrift unterscheiden können.
- Eine automatische Lösung sollte deshalb nicht nur ein einziges starres Formularlayout verarbeiten können.

## Begriffe

- **Layout:** Räumliche Anordnung von Text, Linien, Tabellen, Logos und weiteren Elementen auf einer Seite.
- **Formulartyp:** Gruppe von Formularen mit ähnlichem Zweck oder Aufbau, beispielsweise Rechnungen oder Fragebögen.
- **RVL-CDIP:** Großer Datensatz mit gescannten Dokumentbildern. Die Bilder auf dieser Folie dienen als Beispiele für verschiedene Dokumentarten.
- **Scanqualität:** Eigenschaften eines eingescannten Dokuments, beispielsweise Auflösung, Rauschen, Schieflage oder unscharfer Druck.

## Nicht falsch sagen

- Die vier Bilder sind Motivationsbeispiele. Sie sind nicht automatisch alle Teil des FUNSD-Trainingssatzes.

# Folie 3 – Warum Textregionen automatisch erkannt werden sollen

## Das sage ich

- Rechts ist die vollständige Verarbeitungskette dargestellt.
- Zuerst liegt nur die gesamte Formularseite als Bild vor.
- Danach müssen die Textregionen lokalisiert werden.
- OCR wandelt die gefundenen Bildbereiche in maschinenlesbaren Text um.
- Erst anschließend kann eine Dokumentenanalyse beispielsweise Inhalte oder Beziehungen auswerten.
- Wenn Bounding Boxes manuell eingezeichnet werden müssen, ist die Verarbeitung nicht vollständig automatisiert.
- In dieser Arbeit sollen deshalb zusätzliche manuelle Annotationen möglichst reduziert werden.

## Begriffe

- **OCR – Optical Character Recognition:** Optische Zeichenerkennung. Sie wandelt Schrift in einem Bild in Zeichen beziehungsweise maschinenlesbaren Text um.
- **Lokalisierung:** Bestimmung der Position eines Objekts im Bild. Hier wird die Position durch eine Bounding Box angegeben.
- **Dokumentenanalyse:** Nachgelagerte Auswertung des erkannten Textes und seiner Struktur, zum Beispiel die Zuordnung eines Wertes zu einem Feld.
- **Annotation:** Zusatzinformation zu einem Trainingsbild. In dieser Arbeit ist das hauptsächlich eine Bounding Box um eine Textregion.
- **Manuelle Annotation:** Eine Person zeichnet oder korrigiert die Bounding Boxes.
- **Automatisierung:** Verarbeitung ohne wiederholte manuelle Eingriffe. Das Ziel betrifft die gesamte Kette und nicht nur OCR.

## Nicht falsch sagen

- OCR und Text Detection sind nicht dasselbe.
- Text Detection findet die Position des Textes. OCR erkennt danach die Zeichen innerhalb dieser Position.

# Folie 4 – Aufgabe: Textregionen statt Feldbedeutungen

## Das sage ich

- Links sehen wir beispielhafte Bounding Boxes aus FUNSD.
- Der Detektor besitzt nur eine relevante Objektklasse: `text`.
- Eine Bounding Box beschreibt jeweils einen zusammenhängenden Textabschnitt.
- Es wird nicht entschieden, ob der Text ein Name, ein Datum oder eine Adresse ist.
- Die semantische Bedeutung gehört zu einem späteren Verarbeitungsschritt.

## Begriffe

- **Bounding Box:** Rechteck, das Position und Ausdehnung eines Objekts beschreibt. Üblich sind vier Koordinaten oder Mittelpunkt, Breite und Höhe.
- **Klasse / Class:** Kategorie, die ein Modell einem Objekt zuordnet. Hier existiert nur die Objektklasse `text`.
- **Hintergrund / Background:** Bildbereiche, die nicht zur gesuchten Objektklasse gehören. Faster R-CNN behandelt den Hintergrund intern als Gegenklasse.
- **Textsegment:** Zusammenhängender Textbereich, für den eine gemeinsame Bounding Box verwendet wird.
- **Semantik:** Inhaltliche Bedeutung. Die Zeichenfolge „12.05.2026“ kann beispielsweise als Datum interpretiert werden, wird hier aber nur als Textregion erkannt.
- **Feldsemantik:** Bedeutung eines Formularfeldes, beispielsweise Name, Datum oder Adresse.

## Nicht falsch sagen

- Das Modell liest den Text auf dieser Stufe noch nicht.
- Es erkennt auch keine Feldtypen. Es erkennt nur die Position von Textregionen.

# Folie 5 – Herausforderung: Annotationsaufwand und Realitätslücke

## Das sage ich

- Reale Trainingsseiten benötigen normalerweise manuell geprüfte Bounding Boxes.
- Dieser Arbeitsschritt ist langsam und muss bei neuen Formulartypen teilweise wiederholt werden.
- Synthetische Seiten lösen das Kostenproblem teilweise, weil ihre Ground Truth automatisch bekannt ist.
- Sie unterscheiden sich aber von realen Scans.
- Diese Realitätslücke erklärt, warum ein rein synthetisch trainiertes Modell auf realen Formularen Leistung verliert.

## Begriffe

- **Annotationsaufwand:** Zeit und Kosten für das Erstellen und Prüfen von Labels.
- **Ground Truth:** Als korrekt angenommene Referenzannotation. Sie wird für Training oder Auswertung verwendet.
- **Realitätslücke:** Unterschied zwischen künstlich erzeugten Trainingsbildern und echten Zielbildern. In englischer Literatur wird häufig `domain gap` gesagt.
- **Datenverteilung:** Statistische Eigenschaften einer Datenmenge, beispielsweise Schriftarten, Bildrauschen, Layouts und Objektgrößen.
- **Generalisierung:** Fähigkeit eines Modells, auf neuen, nicht im Training gesehenen Bildern korrekt zu arbeiten.
- **Scanrauschen:** Unregelmäßige Helligkeits- oder Farbschwankungen, die beim Drucken, Kopieren oder Scannen entstehen.

## Nicht falsch sagen

- Synthetische Daten sind nicht grundsätzlich schlecht.
- Das Problem ist, dass sie bestimmte Eigenschaften realer Formulare nur unvollständig nachbilden.

# Folie 6 – Datengrundlage: FUNSD

## Das sage ich

- FUNSD ist ein Datensatz mit realen, verrauschten Formularscans.
- Er enthält 199 vollständig annotierte Formulare.
- Davon gehören 149 Seiten zum Training Split und 50 zum Test Split.
- Die synthetischen Seiten werden aus Layouts des Training Splits erzeugt.
- Die Leistung wird auf den realen Seiten des Test Splits gemessen.

## Begriffe

- **FUNSD:** `Form Understanding in Noisy Scanned Documents`. Datensatz für die Analyse gescannter Formulare.
- **Datensatz / Dataset:** Geordnete Sammlung von Bildern und gegebenenfalls zugehörigen Annotationen.
- **Training Split:** Teil des Datensatzes, der zum Lernen der Modellgewichte verwendet wird.
- **Test Split:** Getrennter Teil, der die abschließende Leistung auf nicht trainierten Beispielen misst.
- **dpi – dots per inch:** Maß für die Rasterauflösung. 100 dpi bedeutet ungefähr 100 Bildpunkte pro Zoll.
- **Gerastertes Dokument:** Dokument, das als Pixelbild und nicht als direkt bearbeitbarer Text vorliegt.
- **Reales Druck- und Scanrauschen:** Bildfehler, die aus dem physischen Dokument und dem Scanprozess stammen.
- **Trainingslayout:** Räumliche Struktur einer Trainingsseite, die als Vorlage für synthetische Seiten verwendet wird.

## Nicht falsch sagen

- Der reale Test Split wird nicht synthetisch ersetzt.
- Er bleibt die gemeinsame reale Vergleichsbasis der Experimente.

# Folie 7 – Erzeugung synthetischer Trainingsseiten

## Das sage ich

- Die Abbildung wird von links nach rechts gelesen.
- Zunächst geben die vorhandenen FUNSD-Bounding-Boxes die zu verändernden Textbereiche vor.
- Inpaint Anything entfernt den ursprünglichen Text und rekonstruiert den Hintergrund.
- Neuer Text wird passend umbrochen und in die Region eingesetzt.
- Da seine Position beim Erzeugen bekannt ist, wird die neue Ground Truth automatisch gespeichert.

## Begriffe

- **Synthetic Data Generation:** Automatische Erzeugung künstlicher Trainingsbilder und ihrer Labels.
- **Inpainting:** Entfernen eines Bildbereichs und Rekonstruktion eines plausiblen Hintergrunds aus der Umgebung.
- **Inpaint Anything:** Verwendetes Verfahren beziehungsweise Werkzeug zur Segmentierung und Rekonstruktion zu entfernender Bildbereiche.
- **Text Rendering:** Zeichnen von Text mit einer Schriftart an einer bestimmten Bildposition.
- **Text Wrapping:** Umbrechen eines längeren Textes auf mehrere Zeilen, damit er in eine Region passt.
- **Direktes Einfügen / Direct Insertion:** Neuer Text wird direkt in den rekonstruierten Bereich eingesetzt.
- **Automatisch erzeugte Ground Truth:** Bounding Boxes werden aus den bekannten Positionen des gerenderten Textes berechnet und müssen nicht von Hand gezeichnet werden.
- **Augmentation:** Zusätzliche künstliche Veränderung eines Trainingsbildes, beispielsweise Rauschen, Unschärfe, Helligkeit oder Kontrast.

## Nicht falsch sagen

- Inpainting erzeugt nicht den neuen Text.
- Es entfernt zuerst den alten Text und rekonstruiert den Hintergrund. Das Text Rendering ist ein eigener Schritt.

# Folie 8 – Detektor: Faster Region-Based Convolutional Neural Network

## Das sage ich – zuerst an der linken Grafik

- Die Grafik wird von unten nach oben gelesen.
- Unten liegt das Eingabebild.
- Die Convolutional Layers berechnen daraus Feature Maps.
- Auf diesen Feature Maps erzeugt das Region Proposal Network mögliche Objektregionen.
- RoI Align bringt die Features unterschiedlich großer Proposals auf eine gemeinsame Größe.
- Der Detection Head klassifiziert die Proposals und korrigiert ihre Bounding Boxes.

## Das sage ich – danach die fünf Punkte rechts

1. ResNet-50 übernimmt die Feature Extraction. Das Feature Pyramid Network stellt Features auf mehreren Skalen bereit.
2. Das Region Proposal Network erzeugt Region Proposals mit hoher Wahrscheinlichkeit für ein Objekt.
3. RoI Align vereinheitlicht die Feature-Darstellung der unterschiedlich großen Proposals.
4. Classification unterscheidet Text und Background. Box Regression verbessert gleichzeitig Lage und Größe der Bounding Box.
5. Non-Maximum Suppression entfernt stark überlappende doppelte Predictions.

## Begriffe

- **Faster R-CNN:** `Faster Region-Based Convolutional Neural Network`. Zweistufiger Object Detector.
- **Convolutional Neural Network – CNN:** Neuronales Netz, das lokale Filter auf Bilder oder Feature Maps anwendet.
- **Convolution / Faltung:** Ein kleiner Filter wird über lokale Bereiche verschoben. Aus den gewichteten Eingaben entsteht an jeder Position ein neuer Wert.
- **Filter / Kernel:** Kleine Matrix mit trainierbaren Gewichten. Sie reagiert nach dem Training beispielsweise auf Kanten, Linien oder komplexere Muster.
- **Gewicht / Weight:** Trainierbarer Zahlenwert eines neuronalen Netzes. Die Gewichte werden während des Trainings durch den Optimizer angepasst.
- **Feature:** Vom Netz berechnete Eigenschaft eines Bildbereichs. Ein Feature kann einfache Kanten oder komplexere Textstrukturen repräsentieren.
- **Feature Map:** Räumlich angeordnete Aktivierungen eines Features. Die Positionen bleiben grob mit Positionen im Eingabebild verknüpft.
- **ResNet-50:** CNN-Architektur mit ungefähr 50 gewichteten Schichten und Residual Connections. Sie dient hier als Backbone.
- **Residual Connection:** Verbindung, die Informationen an mehreren Schichten vorbeileitet und zum späteren Ergebnis addiert. Sie erleichtert das Training tiefer Netze.
- **Backbone:** Hauptnetz zur Feature Extraction. Hier ist das ResNet-50.
- **Feature Pyramid Network – FPN:** Verbindet Feature Maps aus mehreren Tiefen und erzeugt eine Feature-Pyramide für unterschiedlich große Objekte.
- **Skala / Scale:** Größenebene. Kleine Textobjekte benötigen oft feinere Feature Maps, große Objekte können auf gröberen Feature Maps verarbeitet werden.
- **Region Proposal Network – RPN:** Netzteil, der mögliche Objektregionen erzeugt. Er liefert noch nicht die endgültige Klassifikation.
- **Region Proposal:** Kandidaten-Bounding-Box, die wahrscheinlich ein Objekt enthält.
- **RoI – Region of Interest:** Region im Bild beziehungsweise auf den Feature Maps, die genauer untersucht werden soll.
- **RoI Align:** Liest Features einer RoI durch bilineare Interpolation aus und bringt sie auf eine feste Ausgabengröße.
- **Bilineare Interpolation:** Schätzung eines Wertes aus benachbarten Positionen. Dadurch muss RoI Align Koordinaten nicht grob auf ganze Rasterzellen runden.
- **Classification:** Vorhersage der Objektklasse. Hier wird zwischen Text und Background unterschieden.
- **Box Regression:** Vorhersage kontinuierlicher Korrekturwerte für Position und Größe einer Bounding Box.
- **Detection Head:** Letzter Teil des Detektors, der Classification und Box Regression ausführt.
- **Prediction:** Ausgabe des Modells, beispielsweise Klasse, Bounding Box und Confidence Score.
- **Non-Maximum Suppression – NMS:** Filtert stark überlappende Predictions. Meist bleibt die Bounding Box mit dem höchsten Score erhalten.
- **COCO Pretraining:** Vorheriges Training auf dem großen COCO-Objektdatensatz. Dadurch besitzt das Netz bereits allgemeine Bild-Features.
- **Fine-Tuning:** Weitertraining eines vortrainierten Netzes auf der neuen Aufgabe. In dieser Arbeit wird das gesamte Netz gemeinsam angepasst.

## Was wird trainiert?

- Die Gewichte von ResNet-50.
- Die Gewichte des Feature Pyramid Network.
- Die Gewichte des Region Proposal Network.
- Der Classification Head.
- Der Box Regression Head.

## Nicht falsch sagen

- ResNet-50 enthält nicht nur unveränderliche Filter. Die vortrainierten Filter sind der Startpunkt und werden beim Fine-Tuning weiter angepasst.
- RoI Align verändert nicht die ursprüngliche Größe der Bounding Box. Es vereinheitlicht die zugehörige Feature-Darstellung für den Detection Head.
- Das RPN liefert noch keine endgültige Textentscheidung. Es erzeugt zunächst Kandidaten.

# Folie 9 – Wie entstehen Region Proposals?

## Das sage ich

- Die Originalgrafik aus dem Faster-R-CNN-Paper wird von unten nach oben gelesen.
- Unten liegt die Convolutional Feature Map. Jede Position steht für einen lokalen Bereich des Eingabebildes.
- Das Sliding Window wendet dasselbe kleine Netz nacheinander auf alle Positionen der Feature Map an.
- An jeder Position werden mehrere Anchor Boxes mit unterschiedlichen Größen und Seitenverhältnissen betrachtet.
- Die `cls layer` schätzt im Originalpapier für jeden Anchor Objekt oder Hintergrund.
- Die `reg layer` sagt für jeden Anchor vier Korrekturwerte für Position und Größe voraus.
- Aus korrigierten Anchor Boxes mit hohen Scores entstehen die Region Proposals. NMS entfernt starke Dopplungen.
- Erst die zweite Faster-R-CNN-Stufe trifft die endgültige Klassenentscheidung und verfeinert die Box erneut.

## Begriffe

- **Anchor Box:** Vordefinierte Ausgangsbox mit bestimmter Größe und bestimmtem Seitenverhältnis.
- **k:** Anzahl der Anchor Boxes pro Position. Das Originalpapier verwendet standardmäßig drei Größen und drei Seitenverhältnisse, also `k = 9`.
- **Seitenverhältnis / Aspect Ratio:** Verhältnis von Breite zu Höhe einer Bounding Box.
- **Objectness Score:** Geschätzte Wahrscheinlichkeit, dass eine Region irgendein relevantes Objekt und nicht nur Background enthält.
- **Box-Regression-Werte / Box Offsets:** Vier relative Korrekturwerte für horizontale Position, vertikale Position, Breite und Höhe.
- **Sliding Window:** Lokaler Ausschnitt der Feature Map, auf den das kleine RPN-Netz angewendet wird. Das Fenster wird über alle räumlichen Positionen verschoben.
- **Intermediate Layer:** Zwischenschicht, welche die lokalen Features des Sliding Windows verarbeitet, bevor sie an die beiden Ausgabepfade weitergegeben werden.
- **cls layer:** Classification Branch des RPN. Sie sagt Objectness voraus, also Objekt gegen Hintergrund, aber noch nicht die endgültige Objektklasse.
- **reg layer:** Regression Branch des RPN. Sie berechnet die vier Box Offsets für jeden Anchor.
- **2k Scores:** Darstellung im Originalpapier: zwei Klassenwerte für jeden der `k` Anchors, nämlich Objekt und Hintergrund.
- **4k Coordinates:** Vier Korrekturwerte für jeden der `k` Anchors.
- **256-d:** Dimensionalität der Intermediate Layer in der im Paper gezeigten ZF-Konfiguration. Sie ist keine exakte Dimensionsangabe für das in dieser Arbeit verwendete ResNet-50-FPN-Modell.
- **Dense Prediction:** Vorhersagen werden an vielen räumlichen Positionen erzeugt und nicht nur einmal für das gesamte Bild.
- **Score Map / Heatmap:** Anschauliche Darstellung räumlicher Scores. Technisch erzeugt das RPN mehrere Objectness Scores pro Position und Anchor Box.
- **Two-Stage Detector:** Detektor mit zwei Stufen. Stufe eins erzeugt Proposals, Stufe zwei klassifiziert und korrigiert sie.
- **One-Stage Detector:** Detektor, der direkt endgültige Klassen und Boxen vorhersagt.
- **YOLO:** Bekannte Familie von One-Stage Detectors. YOLO erzeugt direkt endgültige Detections; das RPN erzeugt nur Kandidaten für die zweite Faster-R-CNN-Stufe.

## Nicht falsch sagen

- Die Anchor Boxes werden nicht bereits als endgültige Detections ausgegeben.
- Die Feature Map besteht nicht aus fertig erkannten Textboxen. Sie enthält gelernte Features, aus denen das RPN Scores und Offsets berechnet.
- Die Angaben `256-d` und `2k scores` stammen aus der Originalkonfiguration der Paper-Abbildung. Sie dürfen nicht als exakte interne Dimensionen der torchvision-Implementierung dieser Arbeit ausgegeben werden.

# Folie 10 – Auswertung der Erkennungen

## Das sage ich

- Blau ist die Prediction und grün die Ground Truth.
- Die Intersection ist die gemeinsame Fläche beider Bounding Boxes.
- Die Union Area umfasst alles, was in mindestens einer der beiden Boxen liegt.
- Intersection over Union teilt die gemeinsame Fläche durch diese Union Area.
- Ab einer IoU von 0,5 wird eine Prediction einer Ground-Truth-Box als Match zugeordnet.
- Precision misst die Zuverlässigkeit der Predictions.
- Recall misst, wie vollständig die Ground Truth gefunden wird.
- F1 fasst Precision und Recall zusammen.

## Begriffe

- **Prediction:** Vom Modell vorhergesagte Bounding Box.
- **Ground Truth:** Referenz-Bounding-Box aus der Annotation.
- **Intersection Area:** Fläche, die gleichzeitig in Prediction und Ground Truth liegt.
- **Union Area:** Gesamtfläche beider Boxen, wobei die Intersection nur einmal gezählt wird.
- **Intersection over Union – IoU:** Intersection Area geteilt durch Union Area. Der Wert liegt zwischen null und eins.
- **IoU Threshold:** Mindestwert für einen Match. Hier wird 0,5 verwendet.
- **Match:** Zuordnung einer Prediction zu einer Ground-Truth-Box.
- **True Positive – TP:** Korrekt zugeordnete Prediction.
- **False Positive – FP:** Prediction ohne passende Ground Truth.
- **False Negative – FN:** Ground-Truth-Box, die nicht gefunden wurde.
- **Precision:** `TP / (TP + FP)`. Anteil korrekter Predictions unter allen Predictions.
- **Recall:** `TP / (TP + FN)`. Anteil gefundener Ground-Truth-Objekte.
- **F1:** Harmonisches Mittel aus Precision und Recall. `2 · Precision · Recall / (Precision + Recall)`.
- **Harmonisches Mittel:** Mittelwert, der nur hoch wird, wenn beide beteiligten Werte hoch sind. Ein sehr niedriger Wert kann nicht einfach durch einen sehr hohen ausgeglichen werden.

## Nicht falsch sagen

- Union Area ist nicht einfach die Summe beider Flächen. Die überlappende Fläche würde dann doppelt gezählt und muss einmal abgezogen werden.
- IoU misst die räumliche Übereinstimmung einer einzelnen Boxzuordnung. F1 fasst anschließend viele Zuordnungen über den Datensatz zusammen.

# Folie 11 – Self-Training: Teacher und Student

## Das sage ich

- Der erste Teacher wird auf synthetischen Seiten mit Ground Truth trainiert.
- Er erzeugt Pseudo-Labels für reale, nicht annotierte Trainingsseiten.
- Der Student lernt aus zwei Quellen: synthetischer Ground Truth und realen Pseudo-Labels.
- Die synthetischen Ausgangsdaten bleiben in jeder regulären Runde enthalten.
- Der trainierte Student wird zum Teacher der nächsten Runde.

## Begriffe

- **Teacher:** Modell, das Pseudo-Labels für nicht annotierte Bilder erzeugt.
- **Student:** Neues Modell, das mit Ground Truth und Pseudo-Labels trainiert wird.
- **Labeled Data:** Daten mit verfügbaren, als sicher angenommenen Labels.
- **Unlabeled Data:** Daten, deren Ground Truth nicht für das Training verwendet wird.
- **Pseudo-Label:** Automatische Teacher-Prediction, die vorübergehend als Trainingslabel verwendet wird.
- **Teacher-Student-Runde:** Vollständiger Ablauf aus Pseudo-Label-Erzeugung, Student-Training und Übergang zum nächsten Teacher.
- **Bootstrapping:** Start eines iterativen Verfahrens mit einem ersten Modell. Hier entsteht der erste Teacher aus synthetischer Ground Truth.
- **Trusted Labels:** Verlässlich annotierte Labels. In dieser Arbeit übernimmt die synthetische Ground Truth diese Rolle.

## Nicht falsch sagen

- Der Student ersetzt den Teacher nicht während desselben Trainingsschritts. Erst nach seinem Training und Refinement wird er zum nächsten Teacher.
- Die reguläre Methode verwirft die synthetischen Daten nicht. Sie bleiben zusätzlich zu den realen Pseudo-Labels im Training.

# Folie 12 – ASTOD: fünf Schritte einer Runde

## Das sage ich

1. Ein Teacher wird auf Labeled Data trainiert.
2. Der Teacher erzeugt Multi-View Predictions für jedes nicht annotierte Bild.
3. Die Predictions werden zurücktransformiert, mit NMS zusammengeführt und durch den Adaptive Threshold gefiltert.
4. Der Student lernt aus Labeled Data und Pseudo-Labeled Data. Unsichere Pseudo-Labels erhalten ein geringeres Gewicht im Loss.
5. Im Refinement wird der Student noch einmal nur auf Labeled Data trainiert. Danach ersetzt er den Teacher.

## Begriffe

- **ASTOD:** `Adaptive Self-Training for Object Detection`.
- **Semi-Supervised Object Detection – SSOD:** Object Detection mit einer kleinen gelabelten und einer größeren ungelabelten Datenmenge.
- **Multi-View Prediction:** Dasselbe Bild wird in mehreren Ansichten verarbeitet, um stabilere Kandidaten zu erhalten.
- **Original View:** Unveränderte Bildansicht.
- **Horizontal Flip:** Horizontale Spiegelung des Bildes.
- **Scaling:** Größenänderung des Bildes. ASTOD verwendet unter anderem eine Vergrößerung um den Faktor zwei.
- **Inverse Transformation:** Rückgängigmachen von Spiegelung oder Skalierung für die vorhergesagten Boxkoordinaten.
- **Aggregation:** Zusammenführen der Predictions aus mehreren Ansichten.
- **Confidence Score:** Vom Modell geschätzte Sicherheit einer Prediction.
- **Histogramm:** Häufigkeitsdarstellung der Confidence Scores in Wertebereichen.
- **Bin:** Einzelner Wertebereich eines Histogramms.
- **Adaptive Threshold:** Grenzwert, der aus den Confidence Scores des aktuellen Modells bestimmt wird und nicht vorher dauerhaft feststeht.
- **Ground Threshold:** Bezeichnung im ASTOD-Paper für den aus dem Score-Histogramm gewählten Grenzwert.
- **Thresholding:** Verwerfen von Predictions, deren Score unter dem Grenzwert liegt.
- **Loss:** Zahlenwert, der den Trainingsfehler beschreibt. Der Optimizer passt die Gewichte so an, dass der Loss kleiner wird.
- **Classification Loss:** Fehler der vorhergesagten Klasse.
- **Regression Loss:** Fehler der vorhergesagten Bounding-Box-Koordinaten.
- **Confidence Weighting:** Pseudo-Labels mit hoher Confidence beeinflussen das Training stärker als unsichere Pseudo-Labels.
- **Refinement:** Zusätzliches Weitertraining des Student nur auf Labeled Data, um den Einfluss fehlerhafter Pseudo-Labels zu begrenzen.
- **Iteration:** Wiederholung des gesamten ASTOD-Ablaufs mit dem verfeinerten Student als neuem Teacher.

## Tieferes ASTOD-Wissen für Rückfragen

- ASTOD verwendet Original, horizontal gespiegelte Ansicht, vergrößerte Ansicht sowie gespiegelte und vergrößerte Ansicht.
- Die Boxen werden durch inverse Transformation in das Koordinatensystem des Originalbildes zurückgeführt.
- NMS wird beim Bereinigen und Zusammenführen der Kandidaten verwendet.
- Im Paper wird das Score-Histogramm zwischen 0,5 und 1 mit 21 Bins betrachtet.
- Der Bin mit geringer Dichte dient als Trennung zwischen eher unsicheren und eher sicheren Predictions.
- Der verfeinerte Student ersetzt den Teacher erst nach dem Refinement.

## Nicht falsch sagen

- Der Adaptive Threshold ist kein zusätzlicher trainierter Netzwerkparameter.
- Er wird aus der Verteilung der aktuellen Confidence Scores bestimmt.
- Refinement bedeutet im originalen ASTOD: Training auf den sicher gelabelten Daten, nicht auf den Pseudo-Labels.

# Folie 13 – Übertragung von ASTOD auf diese Arbeit

## Das sage ich

- Im originalen ASTOD sind Labeled und Unlabeled Data Teilmengen derselben grundlegenden Datenverteilung.
- In dieser Arbeit übernehmen synthetische Seiten die Rolle der Labeled Data.
- Reale FUNSD-Trainingsseiten übernehmen die Rolle der Unlabeled Data.
- Der Student lernt aus synthetischer Ground Truth und realen Pseudo-Labels.
- Das Refinement verwendet erneut nur die sichere synthetische Ground Truth.
- Dadurch bleiben die synthetischen Daten bis zum Ende jeder regulären Runde enthalten.

## Begriffe

- **Übertragung / Mapping:** Zuordnung der Bestandteile eines bestehenden Verfahrens zur neuen Aufgabenstellung.
- **Source Domain:** Datenverteilung, aus der die sicheren Trainingslabels stammen. Hier sind dies die synthetischen Seiten.
- **Target Domain:** Datenverteilung, auf der das Modell letztlich funktionieren soll. Hier sind dies reale FUNSD-Seiten.
- **Domain Shift:** Statistischer Unterschied zwischen Source Domain und Target Domain.
- **ASTOD-inspired:** Das Verfahren orientiert sich an ASTOD, wird aber in einem abweichenden synthetisch-realen Szenario eingesetzt.
- **Sichere synthetische Ground Truth:** Automatisch bekannte Bounding Boxes der synthetisch erzeugten Textregionen.

## Nicht falsch sagen

- Die Arbeit ist keine identische Reproduktion des originalen ASTOD-Setups.
- Die Rollen der Daten werden übertragen: synthetisch entspricht hier labeled, real entspricht unlabeled.

# Folie 14 – Versuchsaufbau

## Das sage ich

- Verwendet wird Faster R-CNN mit ResNet-50 und Feature Pyramid Network.
- Die einzige relevante Detection Class ist `text`; Background wird intern behandelt.
- Als Optimizer wird Stochastic Gradient Descent verwendet.
- Bewertet werden Precision, Recall und F1 bei IoU 0,5.
- Der Initial Teacher wird auf synthetischen Seiten trainiert.
- Im Self-Training werden synthetische Ground Truth und reale Pseudo-Labels kombiniert.

## Begriffe

- **Versuchsaufbau / Experimental Setup:** Gesamtheit der festgelegten Modelle, Daten, Trainingsverfahren und Metriken.
- **Detection Class:** Objektkategorie, die der Detektor vorhersagen soll.
- **Optimizer:** Algorithmus, der Modellgewichte anhand der Gradienten aktualisiert.
- **Stochastic Gradient Descent – SGD:** Optimizer, der Gradienten aus kleinen Teilmengen der Trainingsdaten berechnet und die Gewichte schrittweise verändert.
- **Gradient:** Ableitung des Loss nach einem Gewicht. Er zeigt, in welche Richtung der Loss lokal steigt.
- **Gradient Descent:** Aktualisierung entgegen der Gradientenrichtung, damit der Loss kleiner wird.
- **Mini-Batch:** Kleine Teilmenge der Trainingsdaten für einen Update-Schritt.
- **Epoch:** Ein vollständiger Durchlauf durch den Trainingsdatensatz.
- **Initial Teacher:** Erstes Teacher-Modell vor Beginn der wiederholten Self-Training-Runden.
- **Pretrained Model:** Modell, dessen Gewichte bereits auf einem anderen Datensatz gelernt wurden.
- **Fine-Tuning:** Anpassung des Pretrained Model an die neue Aufgabe.

## Nicht falsch sagen

- Nicht nur der letzte Classification Head wird trainiert. Laut Methodik wird das gesamte Faster-R-CNN-Netz gemeinsam fine-tuned.

# Folie 15 – Hyperparameter Search

## Das sage ich

- Die grundlegenden Einstellungen orientieren sich an FUNSD und Faster R-CNN.
- Die endgültigen Werte werden aber in eigenen Vorversuchen systematisch ausgewählt.
- Danach bleiben sie in den Hauptvergleichen fest.
- Gewählt werden Learning Rate 0,006, Momentum 0,8 und Weight Decay 0,0005.

## Begriffe

- **Hyperparameter:** Vor dem Training festgelegte Einstellung. Sie wird nicht wie ein Netzwerkgewicht direkt aus den Daten gelernt.
- **Hyperparameter Search:** Systematischer Vergleich mehrerer Hyperparameterwerte, um eine geeignete feste Einstellung auszuwählen.
- **Parameter / Modellgewicht:** Wert, den das Modell während des Trainings lernt.
- **Learning Rate:** Bestimmt die Stärke eines Gewichts-Updates. Zu groß kann das Training instabil machen, zu klein kann es stark verlangsamen.
- **Momentum:** Bezieht vorherige Update-Richtungen ein. Dadurch werden zufällige Schwankungen geglättet und konsistente Richtungen verstärkt.
- **Weight Decay:** Regularisierung, die große Gewichte bestraft und der Überanpassung entgegenwirken kann.
- **Regularisierung:** Maßnahme, die zu starke Anpassung an Trainingsdaten begrenzen soll.
- **Overfitting / Überanpassung:** Modell funktioniert auf Trainingsdaten sehr gut, generalisiert aber schlechter auf neue Daten.
- **Vorversuch / Preliminary Experiment:** Experiment zur Auswahl von Einstellungen vor den eigentlichen Hauptvergleichen.
- **Fixed Setting:** Nach der Auswahl unverändert verwendete Einstellung, damit Vergleiche fairer bleiben.

## Unterschied zu FUNSD

- FUNSD liefert einen fachlichen Ausgangspunkt für Faster R-CNN als Textdetektor.
- Die endgültigen Werte dieser Präsentation werden nicht als unverändert aus FUNSD übernommen dargestellt.
- Sie stammen aus der eigenen Hyperparameter Search auf diesem Setup.

## Nicht falsch sagen

- Eine Hyperparameter Search trainiert nicht automatisch das endgültige Modellgewicht.
- Sie vergleicht Einstellungen, unter denen anschließend Modellgewichte gelernt werden.

# Folie 16 – Ausgangsleistung ohne Self-Training

## Das sage ich

- Beide Baselines werden auf denselben realen FUNSD-Testformularen bewertet.
- Synthetic-only erreicht ungefähr 0,70 F1.
- Real-only erreicht ungefähr 0,80 F1.
- Die Differenz zeigt die Ausgangslücke vor dem Self-Training.

## Begriffe

- **Baseline:** Vergleichsmodell oder Vergleichswert, gegen den eine neue Methode bewertet wird.
- **Synthetic-only:** Training nur mit synthetischen Seiten und ihrer Ground Truth.
- **Real-only:** Training nur mit realen, manuell annotierten Trainingsseiten.
- **Supervision / Aufsicht:** Labels, die dem Modell im Training vorgeben, was korrekt ist.
- **Real Supervision:** Aufsicht durch echte manuelle Annotationen realer Seiten.
- **Performance Gap:** Leistungsunterschied zwischen zwei Trainingsstrategien auf derselben Auswertung.

## Nicht falsch sagen

- Der Real-only-Wert ist kein label-freies Ergebnis. Er benötigt reale manuelle Annotationen.

# Folie 17 – Entwicklung über viele Self-Training-Runden

## Das sage ich

- Die horizontale Achse zeigt die Self-Training-Runden.
- Die vertikale Achse zeigt Precision, Recall und F1.
- Der F1-Wert steigt von ungefähr 0,708 auf ein beobachtetes Maximum von 0,7726.
- Der größte Gewinn entsteht früh.
- Später schwankt die Leistung in einem Plateau.

## Begriffe

- **Self-Training-Runde:** Eine vollständige Wiederholung von Pseudo-Labeling, Student-Training und Refinement.
- **Checkpoint:** Gespeicherter Zustand der Modellgewichte zu einem bestimmten Zeitpunkt.
- **Peak / Maximum:** Höchster beobachteter Einzelwert in einem Versuchslauf.
- **Plateau:** Bereich, in dem trotz weiterer Runden kein dauerhafter deutlicher Leistungsanstieg mehr entsteht.
- **Oszillation / Schwankung:** Wechselnde Werte um ein ähnliches Leistungsniveau.
- **Training Curve:** Verlauf einer Metrik über Trainingsschritte oder Runden.

## Nicht falsch sagen

- Der höchste Checkpoint ist ein beobachtetes Maximum und nicht automatisch der statistisch erwartete Wert eines neuen Laufs.
- Das Plateau bedeutet nicht, dass alle Checkpoints exakt denselben F1-Wert besitzen.

# Folie 18 – Zwei Varianten der synthetischen Seiten

## Das sage ich

- Links wird der ursprüngliche Text durch Inpainting entfernt.
- Der neue Text wird direkt in den rekonstruierten Scan-Hintergrund eingesetzt.
- Rechts wird die alte Textregion stattdessen durch ein weißes Rechteck ersetzt.
- Die White-Background-Variante ist einfacher, kann aber auffällige künstliche Flächen erzeugen.

## Begriffe

- **Inpainted Background:** Durch Inpainting rekonstruierter Hintergrundbereich.
- **Direct Insertion:** Direkte Platzierung des neuen Textes auf dem rekonstruierten Hintergrund.
- **White Background Variant:** Variante, bei der die zu ersetzende Textregion zuerst weiß überdeckt wird.
- **Scan Context:** Lokale Hintergrundstruktur eines Scans, beispielsweise Rauschen, Linien oder Verfärbungen.
- **Artifact / Artefakt:** Künstliches Bildmerkmal, das im realen Zielmaterial so nicht vorkommt.
- **White-Box Cue:** Auffällige weiße Fläche, anhand derer das Modell unbeabsichtigt synthetische Regionen erkennen könnte.

## Nicht falsch sagen

- „Direct Insertion“ bedeutet hier nicht, dass der alte Text sichtbar bleibt. Er wird vorher durch Inpainting entfernt.

# Folie 19 – Vergleich der Erzeugungsverfahren

## Das sage ich

- Der rekonstruierte Hintergrund erreicht ungefähr 0,76 F1.
- Die White-Background-Variante erreicht ungefähr 0,75 F1.
- Der realistischere lokale Scan Context liefert damit einen kleinen Vorteil.
- Der Unterschied ist jedoch gering und sollte nicht überinterpretiert werden.

## Begriffe

- **Method Comparison:** Vergleich zweier Erzeugungsverfahren unter dem beschriebenen Setup.
- **Transfer to Real Forms:** Übertragung eines mit synthetischen Daten trainierten Modells auf reale Testformulare.
- **Effect Size:** Größe eines beobachteten Unterschieds. Hier ist die Differenz klein.
- **Überinterpretation:** Ableitung einer zu starken allgemeinen Aussage aus einem kleinen oder begrenzt abgesicherten Unterschied.

# Folie 20 – Einfluss der Menge synthetischer Daten

## Das sage ich

- Hier werden verschiedene Anteile synthetischer Trainingsdaten betrachtet.
- Kleinere Teilmengen können bereits nützlich sein.
- Der vollständige lange Lauf erreicht das stärkste aufgeführte Ergebnis.
- Die Werte stammen aus den jeweils beschriebenen Versuchslängen und sind deshalb keine vollständig isolierte Skalierungskurve.

## Begriffe

- **Synthetic Share:** Verwendeter Anteil der verfügbaren synthetischen Daten.
- **Subset / Teilmenge:** Auswahl aus dem vollständigen synthetischen Datensatz.
- **Full-Data Run:** Versuch mit dem vollständigen synthetischen Datensatz.
- **Long Run:** Versuch mit einer größeren Zahl an Self-Training-Runden.
- **Comparability / Vergleichbarkeit:** Ergebnisse sind besonders gut vergleichbar, wenn außer dem untersuchten Faktor alle Bedingungen gleich sind.

## Nicht falsch sagen

- Nicht behaupten, dass eine bestimmte Prozentzahl unabhängig von Versuchslänge immer genau diesen F1-Wert erzeugt.
- Die Folie zeigt die beobachteten Ergebnisse der zugehörigen Versuche.

# Folie 21 – Ablation: synthetische Daten entfernen

## Das sage ich

- Für die Ablation werden fünf starke Checkpoints aus dem Plateau gewählt.
- Die Regular-Fortsetzung behält synthetische Seiten im Student-Training und führt danach synthetisches Refinement aus.
- In der Pseudo-only-Fortsetzung werden synthetische Seiten ab dem gewählten Checkpoint vollständig entfernt.
- Dadurch bleiben nur reale Pseudo-Labels und das synthetische Refinement entfällt.
- Jede Pseudo-only-Fortsetzung liegt unter ihrer Regular-Vergleichsfortsetzung.

## Klarer Hauptsatz

- Ab den ausgewählten Plateau-Checkpoints werden die synthetischen Seiten vollständig aus dem weiteren Training entfernt; dadurch entfallen sowohl ihr Anteil am Student-Training als auch das anschließende synthetische Refinement.

## Begriffe

- **Ablation:** Gezieltes Entfernen eines Methodenbestandteils, um dessen Beitrag zu untersuchen.
- **Regular:** Unveränderte reguläre Fortsetzung mit synthetischer Ground Truth und realen Pseudo-Labels.
- **Pseudo-only:** Fortsetzung nur mit realen Pseudo-Labels.
- **Continuation / Fortsetzung:** Weiteres Training ausgehend von einem bereits gespeicherten Checkpoint.
- **Plateau Checkpoint:** Gespeicherter Modellzustand aus dem Bereich ohne dauerhaften weiteren Anstieg.
- **Synthetic Refinement:** Refinement nur mit synthetischer Ground Truth.
- **Difference / Differenz:** Pseudo-only-F1 minus Regular-F1. Negative Werte bedeuten, dass Pseudo-only schlechter ist.
- **Paired Comparison:** Beide Fortsetzungen beginnen jeweils am zugehörigen Ausgangs-Checkpoint, wodurch ihr Unterschied gezielter verglichen werden kann.

## Nicht falsch sagen

- Die synthetischen Daten werden nicht schon am Anfang des gesamten Self-Trainings entfernt.
- Sie werden erst ab den ausgewählten Plateau-Checkpoints entfernt.
- Die Ablation trennt nicht den Effekt von synthetischem Student-Training und synthetischem Refinement voneinander. Beide entfallen gemeinsam.

# Folie 22 – Synthetische Daten bei realer Aufsicht

## Das sage ich

- Real-only erreicht 0,8011 F1.
- Direktes gemeinsames Training mit realen und synthetischen Daten erreicht 0,8007 und liefert keinen sichtbaren Gewinn.
- Die Einbindung über Self-Training erreicht 0,8144.
- Der Gewinn ist klein, zeigt aber, dass die Trainingsstrategie wichtiger sein kann als bloßes Mischen der Daten.

## Begriffe

- **Real Supervision:** Training mit echten manuellen Labels realer Seiten.
- **Direct Training / Direct Mixing:** Reale und synthetische Beispiele werden direkt gemeinsam als gelabelte Trainingsdaten verwendet.
- **Self-Training Integration:** Synthetische Daten werden innerhalb des Teacher-Student-Verfahrens eingebunden.
- **Incremental Gain:** Kleine zusätzliche Leistungsverbesserung gegenüber einer bereits starken Baseline.

## Nicht falsch sagen

- Dieses Experiment besitzt bereits reale Supervision und ist daher nicht identisch mit dem synthetisch gestarteten Hauptversuch.
- 0,8144 ist nur geringfügig höher als 0,8011. Die Verbesserung sollte als klein bezeichnet werden.

# Folie 23 – Zentrale Ergebnisse

## Das sage ich

- Synthetic-only erreicht 0,70 F1, Real-only 0,80 F1.
- Synthetisch gestartetes Self-Training verbessert den F1-Wert auf ungefähr 0,77.
- Das Entfernen der synthetischen Grundlage verschlechtert alle getesteten Fortsetzungen.
- Bei vorhandener realer Supervision erreicht synthetisch unterstütztes Self-Training 0,814 F1.
- Self-Training verkleinert die Realitätslücke, ersetzt reale Annotationen aber nicht vollständig.

## Begriffe

- **Main Finding:** Zentrale durch die Experimente gestützte Aussage.
- **Synthetic Start:** Initial Teacher wird auf synthetischer Ground Truth trainiert.
- **Real-only Baseline:** Modell mit realen manuellen Trainingslabels.
- **Stabilizing Data Source:** Datenquelle, die im Training einen verlässlichen Bezugspunkt liefert. Hier übernimmt die synthetische Ground Truth diese Rolle.

## Nicht falsch sagen

- Nicht behaupten, dass synthetische Daten reale Annotationen vollständig ersetzen.
- Der Wert 0,77 bleibt unter der Real-only-Baseline von ungefähr 0,80.

# Folie 24 – Grenzen der Untersuchung

## Das sage ich

- Die Hauptauswertung verwendet nur einen Formulardatensatz und eine Detection Class.
- Synthetische Seiten verwenden Layouts annotierter Ausgangsformulare wieder.
- Schriftarten und Handschrift variieren weniger stark als in vielen realen Anwendungen.
- Die Ergebnisse wurden nicht über viele unabhängige Random Seeds gemittelt.
- Ein eigener Validation Set würde Model Selection und finale Testauswertung sauberer trennen.

## Begriffe

- **Limitation:** Einschränkung, die bei der Interpretation oder Übertragbarkeit der Ergebnisse berücksichtigt werden muss.
- **Single-Dataset Evaluation:** Auswertung auf nur einem Datensatz.
- **Single-Class Detection:** Object Detection mit nur einer relevanten Objektklasse.
- **Layout Reuse:** Synthetische Seiten übernehmen räumliche Strukturen vorhandener Seiten.
- **Variation:** Unterschiedlichkeit der Trainingsbeispiele, beispielsweise Schriftarten, Handschrift oder Layouts.
- **Random Seed:** Startwert, der zufällige Prozesse wie Datenreihenfolge oder Initialisierung reproduzierbar macht.
- **Independent Runs:** Getrennte Trainingsläufe mit unterschiedlichen Random Seeds.
- **Mean / Mittelwert:** Durchschnitt mehrerer Läufe. Er ist stabiler als ein einzelner Lauf.
- **Standard Deviation / Standardabweichung:** Maß für die Schwankung der Ergebnisse zwischen Läufen.
- **Validation Set:** Separate Datenmenge für Hyperparameter- und Checkpoint-Auswahl.
- **Model Selection:** Auswahl eines Modells oder Checkpoints anhand einer Validierungsmetrik.
- **Test Set:** Datenmenge für die abschließende Leistungsbewertung nach allen Auswahlentscheidungen.
- **Clean Evaluation:** Klare Trennung zwischen Auswahl auf dem Validation Set und finaler Messung auf dem Test Set.

## Nicht falsch sagen

- Die Limitations machen die Experimente nicht wertlos.
- Sie begrenzen, wie weit kleine Unterschiede und allgemeine Aussagen über andere Datensätze hinausgetragen werden können.

# Folie 25 – Fazit und Ausblick

## Das sage ich

- Synthetische Daten bilden einen brauchbaren automatischen Ausgangspunkt.
- Self-Training bindet reale Seiten ohne neue manuelle Bounding Boxes ein.
- Die synthetische Ground Truth bleibt in regulären Runden erhalten und stabilisiert das Training.
- Reale Annotationen liefern weiterhin die stärkste direkte Supervision.
- Zukünftig sollten realistischere Schriftarten, Handschrift, Layouts und zusätzliche Datensätze untersucht werden.

## Begriffe

- **Conclusion / Fazit:** Zusammenfassung dessen, was durch die Arbeit gezeigt wurde.
- **Outlook / Ausblick:** Sinnvolle nächste Forschungsschritte, die noch nicht Bestandteil der aktuellen Untersuchung sind.
- **Robustness / Robustheit:** Stabilität eines Modells gegenüber Variationen und Störungen.
- **Newer Detector:** Neuere Object-Detection-Architektur, die zukünftig als Alternative zu Faster R-CNN untersucht werden kann.
- **Generalization Study:** Untersuchung der Übertragbarkeit auf weitere Datensätze oder Dokumentarten.

## Schlusssatz

- Zusammenfassend zeigt diese Arbeit, dass synthetisch gestartetes Self-Training die Lücke zu realen Daten deutlich verkleinern und damit zu einer stärker automatisierten Formularverarbeitung beitragen kann. Vielen Dank für Ihre Aufmerksamkeit.

# Folie 26 – Literatur

## Das sage ich

- Hier sind die zentralen Originalquellen zu FUNSD, RVL-CDIP, Faster R-CNN, ASTOD und Inpaint Anything aufgeführt.

## Begriffe

- **Originalpaper:** Wissenschaftliche Veröffentlichung, in der eine Methode oder ein Datensatz ursprünglich beschrieben wurde.
- **Quelle / Reference:** Nachweis für eine übernommene Aussage, Methode, Abbildung oder Datensatzbeschreibung.
- **Adapted Figure:** Abbildung, die auf einer publizierten Darstellung basiert, aber für die eigene Präsentation zugeschnitten oder angepasst wurde.
- **Citation:** Kurzverweis im Text oder auf einer Folie, der auf den vollständigen Literatureintrag verweist.

# Sehr kurze Gesamtwiederholung

1. Formulare sollen vollständig automatisiert verarbeitet werden.
2. Dafür müssen Textregionen vor OCR automatisch lokalisiert werden.
3. Manuelle Bounding Boxes sind teuer.
4. Synthetische Seiten liefern Ground Truth automatisch, besitzen aber eine Realitätslücke.
5. Faster R-CNN erkennt Textregionen zweistufig über RPN und Detection Head.
6. Self-Training nutzt einen Teacher, um reale Pseudo-Labels zu erzeugen.
7. Der Student lernt weiterhin aus synthetischer Ground Truth und zusätzlich aus realen Pseudo-Labels.
8. ASTOD ergänzt Multi-View Predictions, Adaptive Threshold, Confidence Weighting und Refinement.
9. Der F1-Wert steigt von ungefähr 0,70 auf ungefähr 0,77.
10. Reale Annotationen erreichen weiterhin ungefähr 0,80.
11. Das Entfernen synthetischer Daten im Plateau verschlechtert alle getesteten Fortsetzungen.
12. Synthetische Daten helfen bei realer Supervision leicht, wenn sie über Self-Training eingebunden werden.

# Fünf typische Rückfragen

## Warum braucht OCR vorher Text Detection?

- Weil OCR effizienter und gezielter arbeiten kann, wenn bekannt ist, welche Bildbereiche tatsächlich Text enthalten.

## Woher kommen die Region Proposals?

- Das RPN bewertet Anchor Boxes an vielen Positionen der Feature Maps. Aus Objectness Scores und Box Offsets entstehen die Proposals.

## Bleiben die synthetischen Daten im Self-Training enthalten?

- Ja. In der regulären Methode lernt jeder Student aus synthetischer Ground Truth und realen Pseudo-Labels. Danach folgt zusätzlich synthetisches Refinement.

## Was ist bei der Pseudo-only-Ablation anders?

- Ab einem ausgewählten Plateau-Checkpoint werden die synthetischen Seiten entfernt. Dadurch entfallen sowohl ihr Anteil am Student-Training als auch das synthetische Refinement.

## Ist das exakt das originale ASTOD?

- Nein. Es ist ASTOD-inspired. Im Original stammen Labeled und Unlabeled Data aus derselben grundlegenden Verteilung. Hier werden synthetische Daten als gelabelte Source Domain und reale Formulare als ungelabelte Target Domain eingesetzt.
