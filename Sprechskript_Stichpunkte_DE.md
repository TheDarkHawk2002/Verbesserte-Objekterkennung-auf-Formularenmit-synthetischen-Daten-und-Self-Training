---
title: "Stichpunktartiges Sprechskript"
subtitle: "Verbesserte Objekterkennung auf Formularen mit synthetischen Daten und Self-Training"
author: "Okan Yakin"
lang: de-DE
geometry: margin=2.1cm
fontsize: 10pt
colorlinks: true
---

# Verwendung des Skripts

- Dieses Skript ist eine **Orientierung**, kein Text zum vollständigen Ablesen.
- Pro Folie zuerst die Kernaussage nennen, dann an der Grafik oder Tabelle entlang erklären.
- Die ersten beiden Folien und den Übergang zur Forschungsfrage möglichst auswendig beherrschen.
- Fachbegriffe beim ersten Auftreten kurz auf Deutsch erklären; danach darf die Abkürzung verwendet werden.
- Formulierungen wie „in dieser Arbeit wird“, „wir betrachten“ oder „die Ergebnisse zeigen“ verwenden.
- Die mit **Vertiefung** gekennzeichneten Punkte nur bei Rückfragen oder ausreichend Zeit verwenden.

# Folie 1 – Titel

**Kernaussage**

- Thema der Arbeit: automatische Erkennung von Textregionen auf Formularen bei möglichst geringem manuellem Annotationsaufwand.

**Sprechpunkte**

- Guten Tag, mein Name ist Okan Yakin.
- In dieser Arbeit untersuchen wir, wie sich die Objekterkennung auf Formularen durch synthetische Daten und Self-Training verbessern lässt.
- Im Mittelpunkt steht die Frage, wie reale Formulare genutzt werden können, ohne für jede Seite neue Bounding Boxes manuell einzuzeichnen.
- Dazu betrachten wir zunächst das Anwendungsproblem, anschließend den Detektor und das Self-Training-Verfahren und zuletzt die experimentellen Ergebnisse.

**Übergang**

- Zunächst möchte ich zeigen, weshalb Formulare trotz ihrer alltäglichen Erscheinung ein wichtiges Automatisierungsproblem darstellen.

# Folie 2 – Motivation: Formulare sind überall

**Kernaussage**

- Formulare kommen in vielen Bereichen und in sehr unterschiedlichen Erscheinungsformen vor.

**Sprechpunkte**

- Formulare werden im Gesundheitswesen, im Finanzbereich und in der Verwaltung täglich verarbeitet.
- Die Beispiele zeigen Rechnungen, Haushaltspläne, Fragebögen und handschriftlich ausgefüllte Formulare.
- Sie unterscheiden sich deutlich in Layout, Schrift, Druckqualität und handschriftlichen Anteilen.
- Eine automatische Lösung muss deshalb mehr als nur ein einziges festes Formularlayout verarbeiten können.

**Übergang**

- Für eine durchgängige Automatisierung reicht es nicht, die Seite nur einzuscannen; die einzelnen Verarbeitungsschritte müssen ebenfalls automatisiert werden.

# Folie 3 – Warum Textregionen automatisch erkannt werden sollen

**Kernaussage**

- Die Lokalisierung von Textregionen ist ein notwendiger Zwischenschritt, dessen manuelle Annotation möglichst vermieden werden soll.

**Sprechpunkte**

- Rechts ist die Verarbeitungskette dargestellt: vollständiges Formular, erkannte Textregionen, optische Zeichenerkennung und anschließend Dokumentenanalyse.
- OCR steht für **Optical Character Recognition**, also optische Zeichenerkennung. Sie wandelt Bildbereiche mit Schrift in maschinenlesbaren Text um.
- Bevor OCR zuverlässig arbeiten kann, muss bekannt sein, in welchen Bereichen sich Text befindet.
- Für das Training eines Detektors werden normalerweise viele manuell eingezeichnete Boxen benötigt.
- Diese Annotationen kosten Zeit und Geld und müssen für neue Formulartypen häufig erneut erstellt werden.
- Ziel ist daher nicht nur eine hohe Erkennungsleistung, sondern auch die Verringerung zusätzlicher manueller Annotationen.
- Je weniger manuelle Zwischenschritte erforderlich sind, desto besser lässt sich die gesamte Dokumentenverarbeitung automatisieren.

**Übergang**

- Dabei wird in dieser Arbeit bewusst nur die Position des Textes erkannt und noch nicht seine inhaltliche Bedeutung.

# Folie 4 – Aufgabe: Textregionen statt Feldbedeutungen

**Kernaussage**

- Der Detektor erkennt genau eine Objektklasse: Textregionen.

**Sprechpunkte**

- Links sehen wir ein FUNSD-Formular mit Boxen um zusammenhängende Textabschnitte.
- Eine **Bounding Box** ist ein rechteckiger Rahmen, der Position und Ausdehnung eines Objekts beschreibt.
- In dieser Arbeit existiert nur die Erkennungsklasse `text`; der Hintergrund wird intern als Gegenklasse behandelt.
- Es wird nicht unterschieden, ob eine Region beispielsweise einen Namen, ein Datum oder eine Adresse enthält.
- Diese semantische Zuordnung kann erst in einem späteren Schritt der Dokumentenanalyse erfolgen.

**Übergang**

- Für diese Textboxen werden Trainingsannotationen benötigt, und genau dort entsteht der zentrale Aufwand.

# Folie 5 – Herausforderung: Annotationsaufwand und Realitätslücke

**Kernaussage**

- Synthetische Daten liefern Annotationen automatisch, unterscheiden sich aber von realen Scans.

**Sprechpunkte**

- Für reale Trainingsseiten müsste jede Textregion manuell eingezeichnet und anschließend geprüft werden.
- Synthetische Seiten sind attraktiv, weil ihre Textpositionen beim Erzeugen bereits bekannt sind und die Bounding Boxes automatisch gespeichert werden können.
- Zwischen synthetischen und realen Seiten besteht jedoch eine **Realitätslücke**.
- Reale Seiten enthalten beispielsweise Scanrauschen, Logos, Handschrift, verschiedene Schriftarten und unregelmäßige Druckqualität.
- Deshalb lässt sich ein ausschließlich synthetisch trainiertes Modell nicht vollständig auf reale Formulare übertragen.
- Gesucht wird ein Verfahren, das reale Seiten einbezieht, ohne dort jede Box manuell zu annotieren.

**Übergang**

- Als reale Datengrundlage wird dafür der FUNSD-Datensatz verwendet.

# Folie 6 – Datengrundlage: FUNSD

**Kernaussage**

- FUNSD liefert reale, verrauschte Formulare sowie die Referenzannotation für Training und Auswertung.

**Sprechpunkte**

- FUNSD bedeutet **Form Understanding in Noisy Scanned Documents**.
- Der Datensatz umfasst 199 vollständig annotierte Formulare: 149 Trainings- und 50 Testseiten.
- Die Seiten liegen als gerasterte Dokumente mit 100 dpi und realistischem Druck- und Scanrauschen vor.
- In dieser Arbeit werden die Layouts der Trainingsseiten als Grundlage für die Erzeugung synthetischer Seiten verwendet.
- Die endgültige Bewertung erfolgt auf den realen FUNSD-Testseiten.
- Der Testsatz bleibt damit real und wird nicht durch synthetische Testseiten ersetzt.

**Übergang**

- Als Nächstes sehen wir, wie aus den vorhandenen Layouts automatisch neue synthetische Trainingsseiten entstehen.

# Folie 7 – Erzeugung synthetischer Trainingsseiten

**Kernaussage**

- Vorhandener Text wird entfernt, durch neuen Text ersetzt und gleichzeitig automatisch annotiert.

**Sprechpunkte**

- Wir lesen die Abbildung von links nach rechts.
- Zunächst bestimmen die vorhandenen FUNSD-Boxen, welche Textbereiche verändert werden sollen.
- **Inpainting** bezeichnet das Entfernen eines Bildbereichs und die Rekonstruktion eines möglichst passenden Hintergrunds.
- Inpaint Anything wird verwendet, um den ursprünglichen Text zu entfernen und den Hintergrund zu ergänzen.
- Anschließend wird Wikipedia-Text an die Größe der jeweiligen Region angepasst, umbrochen und eingesetzt.
- Da Position und Zeilen beim Erzeugen bekannt sind, können die neuen Bounding Boxes ohne zusätzliche Handarbeit gespeichert werden.
- Auf diese Weise entstehen synthetische Trainingsbeispiele mit automatisch verfügbarer Ground Truth.

**Begriffe**

- **Ground Truth:** als korrekt angenommene Referenzannotation, gegen die gelernt oder ausgewertet wird.
- **Inpainting:** bildbasierte Rekonstruktion eines entfernten Bereichs aus seiner Umgebung.

**Übergang**

- Diese synthetischen Seiten werden anschließend zum Training eines Faster R-CNN-Detektors verwendet.

# Folie 8 – Detektor: Faster Region-Based Convolutional Neural Network

**Kernaussage**

- Faster R-CNN verarbeitet das Bild zweistufig: zuerst mögliche Regionen, danach genaue Klassifikation und Box-Korrektur.

**An der linken Grafik erklären**

- Die Grafik wird von unten nach oben gelesen.
- Unten wird die Formularseite in ein faltungsbasiertes neuronales Netz gegeben.
- Bei einer **Faltung** werden gelernte Filter über lokale Bildbereiche verschoben. Dadurch entstehen Merkmalskarten, die beispielsweise Kanten, Strukturen oder komplexere Textmuster hervorheben.
- ResNet-50 berechnet aus dem Bild diese lokalen Merkmale. Die Zahl 50 bezeichnet die Tiefe der Netzarchitektur.
- Die dunkle Ebene in der Mitte stellt die Feature Maps, also Merkmalskarten, dar.
- Das Region Proposal Network erzeugt darauf mögliche Objektregionen.
- Diese Vorschläge werden im oberen Teil auf eine gemeinsame Merkmalsgröße gebracht und anschließend klassifiziert und räumlich korrigiert.

**Die fünf Punkte rechts erklären**

1. **Merkmalsextraktion:** ResNet-50 enthält gelernte Filtergewichte und erzeugt lokale Merkmale. Das Feature Pyramid Network stellt zusätzlich mehrere Auflösungs- beziehungsweise Größenebenen bereit, damit kleine und große Textregionen erkannt werden können.
2. **Regionenvorschläge:** Das Region Proposal Network, kurz RPN, sucht auf diesen Merkmalskarten nach Bereichen mit hoher Wahrscheinlichkeit für ein Objekt. Es entscheidet hier noch nicht endgültig über die Klasse.
3. **RoI Align:** Die Regionenvorschläge können unterschiedlich groß sein. RoI Align liest ihre Merkmale aus den Feature Maps aus und bringt sie durch Abtastung auf eine einheitliche räumliche Größe.
4. **Klassifikation und Box-Korrektur:** Der zweite Teil des Netzes entscheidet zwischen Text und Hintergrund. Gleichzeitig sagt eine Box-Regression vier Korrekturwerte voraus, mit denen Mittelpunkt beziehungsweise Ränder sowie Breite und Höhe der Box angepasst werden.
5. **Endgültige Erkennungen:** Non-Maximum Suppression, kurz NMS, behält bei stark überlappenden Boxen normalerweise die Vorhersage mit der höchsten Sicherheit und entfernt doppelte Erkennungen.

**Bezug zur Arbeit**

- Verwendet wird Faster R-CNN mit ResNet-50 und Feature Pyramid Network aus `torchvision`.
- Das Netz ist auf COCO vortrainiert; in dieser Arbeit wird anschließend das gesamte Netz gemeinsam weitertrainiert.
- Die beiden Ausgabeklassen sind Text und Hintergrund.

**Vertiefung bei Rückfragen**

- **Residual-Verbindungen** in ResNet leiten Merkmale an mehreren Schichten vorbei weiter und erleichtern dadurch das Training tiefer Netze.
- **Feature Pyramid Network:** verbindet Merkmale aus mehreren Tiefen, um semantisch starke Merkmale auf verschiedenen Skalen bereitzustellen.
- Die Box-Korrektur erfolgt durch gelernte relative Verschiebungen und Skalierungen gegenüber einem Ausgangsvorschlag.

**Übergang**

- Da die Herkunft der Regionenvorschläge leicht abstrakt bleibt, wird dieser Zwischenschritt auf der nächsten Folie genauer betrachtet.

# Folie 9 – Wie entstehen Region Proposals?

**Kernaussage**

- Das RPN bewertet an vielen Positionen vorgegebene Ausgangsboxen und korrigiert die aussichtsreichsten davon.

**Sprechpunkte**

- Die Originalgrafik aus dem Faster-R-CNN-Paper wird von unten nach oben gelesen.
- Unten liegt die **Convolutional Feature Map**. Jede Position steht für einen lokalen Bereich des ursprünglichen Bildes.
- Das **Sliding Window** wendet dasselbe kleine RPN-Netz auf jede Position dieser Feature Map an.
- An jeder Position werden mehrere **Anchor Boxes** mit unterschiedlichen Größen und Seitenverhältnissen betrachtet.
- Die `cls layer` schätzt für jeden Anchor die Objectness, also Objekt oder Hintergrund.
- Die `reg layer` berechnet pro Anchor vier Korrekturwerte für horizontale Position, vertikale Position, Breite und Höhe.
- Im Originalpapier bedeuten `2k scores` zwei Klassenwerte pro Anchor und `4k coordinates` vier Korrekturwerte pro Anchor.
- Boxen mit hohen Scores werden als Region Proposals weitergegeben. NMS entfernt stark überlappende Dopplungen.
- Die endgültige Classification und eine weitere Box Regression erfolgen erst im Detection Head.
- `256-d` gehört zur gezeigten ZF-Konfiguration des Originalpapiers und ist nicht die konkrete interne Dimension des ResNet-50-FPN-Modells dieser Arbeit.

**Vergleich mit YOLO – nur falls gefragt**

- Sowohl YOLO als auch das RPN treffen dichte Vorhersagen an vielen Bildpositionen.
- YOLO erzeugt als einstufiger Detektor direkt die endgültigen Klassen und Boxen.
- Beim zweistufigen Faster R-CNN erzeugt das RPN zunächst nur Vorschläge; die genaue Klassifikation und Korrektur übernimmt danach ein eigener Detektionskopf.

**Übergang**

- Um die Qualität der endgültigen Boxen zu messen, müssen Vorhersage und Referenzannotation zunächst einander zugeordnet werden.

# Folie 10 – Auswertung der Erkennungen

**Kernaussage**

- Eine Vorhersage gilt als passend, wenn sie ausreichend stark mit der Referenzbox überlappt.

**Sprechpunkte**

- Links unten ist die vorhergesagte Box blau und die Annotation grün dargestellt.
- Die **Intersection over Union**, kurz IoU, ist die Überschneidungsfläche geteilt durch die Vereinigungsfläche beider Boxen.
- Die Vereinigungsfläche umfasst alle Flächenpunkte, die in mindestens einer der beiden Boxen liegen. Der doppelt überlappende Bereich wird dabei nur einmal gezählt.
- Bei einer IoU von mindestens 0,5 wird eine Vorhersage in dieser Auswertung einer Annotation als Treffer zugeordnet.
- **Precision** beantwortet: Wie viele vorhergesagte Boxen sind richtig?
- **Recall** beantwortet: Wie viele vorhandene Referenzboxen wurden gefunden?
- Der **F1-Wert** ist das harmonische Mittel aus Precision und Recall und berücksichtigt damit beide Fehlerarten gemeinsam.

**Vertiefung bei Rückfragen**

- Eine richtige Zuordnung ist ein True Positive.
- Eine vorhergesagte Box ohne passende Annotation ist ein False Positive.
- Eine nicht gefundene Annotation ist ein False Negative.

**Übergang**

- Diese Kennzahlen werden nun verwendet, um den Nutzen von Self-Training zu untersuchen.

# Folie 11 – Self-Training: Teacher und Student

**Kernaussage**

- Self-Training ergänzt sichere synthetische Annotationen um automatisch erzeugte Annotationen realer Seiten.

**Sprechpunkte**

- Zunächst wird ein Teacher auf den synthetischen Seiten mit sicherer Ground Truth trainiert.
- Dieser Teacher erzeugt Vorhersagen auf realen FUNSD-Trainingsseiten ohne verwendete manuelle Boxen.
- Diese automatischen Vorhersagen heißen **Pseudo-Annotationen** oder Pseudo-Labels.
- Der Student wird anschließend aus zwei Datenquellen trainiert: aus den ursprünglichen synthetischen Seiten und den pseudo-annotierten realen Seiten.
- Wichtig ist: Die Daten vom Anfang verschwinden nicht. Die synthetische Ground Truth bleibt in jeder regulären Self-Training-Runde enthalten.
- Der trainierte Student ersetzt anschließend den bisherigen Teacher, und der Ablauf kann wiederholt werden.
- So fließen reale Bildmerkmale in das Training ein, ohne für jede reale Seite manuell neue Boxen zeichnen zu müssen.

**Begriffe**

- **Teacher:** Modell, das Pseudo-Annotationen erzeugt.
- **Student:** Modell, das mit sicheren und pseudo-annotierten Daten neu trainiert wird.
- **Pseudo-Annotation:** automatische Modellvorhersage, die vorübergehend wie eine Trainingsannotation verwendet wird.

**Übergang**

- Der konkrete Ablauf orientiert sich an ASTOD, einem adaptiven Self-Training-Verfahren für die Objekterkennung.

# Folie 12 – ASTOD: fünf Schritte einer Runde

**Kernaussage**

- ASTOD verbessert Pseudo-Annotationen durch mehrere Bildansichten, adaptive Filterung, gewichtetes Training und ein abschließendes Refinement.

**Die fünf Schritte**

1. Der Teacher wird zunächst auf verlässlich annotierten Daten trainiert.
2. Für jedes nicht annotierte Bild werden mehrere Ansichten verwendet: Original, horizontale Spiegelung, Vergrößerung sowie Spiegelung und Vergrößerung.
3. Die vorhergesagten Boxen werden in das Koordinatensystem des Originalbildes zurücktransformiert, zusammengeführt und mit NMS gefiltert. Ein adaptiver Grenzwert entfernt zu unsichere Vorhersagen.
4. Der Student lernt gemeinsam aus den verlässlich annotierten und den pseudo-annotierten Daten. Die Confidence Scores gewichten dabei den Verlust der Pseudo-Annotationen.
5. Im Refinement wird der Student noch einmal nur mit verlässlich annotierten Daten trainiert. Der verfeinerte Student wird zum Teacher der nächsten Runde.

**Begriffe verständlich erklären**

- **Confidence Score:** geschätzte Sicherheit des Modells für eine Vorhersage.
- **Adaptiver Grenzwert:** Die Mindest-Confidence wird aus den aktuell beobachteten Vorhersagen bestimmt und nicht für alle Runden unverändert festgelegt.
- **Loss beziehungsweise Verlustfunktion:** Zahlenwert, der den Fehler des Modells im Training beschreibt. Das Training verändert die Gewichte so, dass dieser Wert kleiner wird.
- **Gewichtung:** Eine sichere Pseudo-Annotation beeinflusst die Gewichtsaktualisierung stärker als eine unsichere.
- **Refinement:** abschließendes Weitertrainieren nur auf verlässlich annotierten Daten, um Fehler aus den Pseudo-Annotationen wieder zu begrenzen.

**Vertiefung zu ASTOD – für Rückfragen**

- ASTOD verwendet NMS zunächst innerhalb der Ansichten und erneut nach dem Zusammenführen der zurücktransformierten Vorhersagen.
- Der adaptive Grenzwert wird aus einem Histogramm der Confidence Scores bestimmt. Im Paper wird im Bereich von 0,5 bis 1 mit 21 Intervallen nach einem Bereich geringer Dichte gesucht.
- Die Grundidee ist, schwache und starke Vorhersagen möglichst datenabhängig voneinander zu trennen.
- Die mehreren Ansichten dienen nicht der Erzeugung neuer manueller Labels, sondern einer stabileren Pseudo-Annotation desselben Bildinhalts.

**Übergang**

- ASTOD wird in dieser Arbeit nicht unverändert übernommen, sondern auf die synthetisch-reale Aufgabenstellung übertragen.

# Folie 13 – Übertragung von ASTOD auf diese Arbeit

**Kernaussage**

- Die synthetischen Seiten übernehmen die Rolle sicher annotierter Daten; reale Seiten liefern die nicht annotierte Zielverteilung.

**An der Grafik von oben nach unten erklären**

- Oben stehen synthetische Seiten mit automatisch bekannter Ground Truth.
- Daraus entsteht der erste Teacher.
- Der Teacher erzeugt Pseudo-Annotationen für reale FUNSD-Trainingsseiten.
- Nach der Filterung lernt der Student aus synthetischen Seiten **und** pseudo-annotierten realen Seiten.
- Danach folgt das Refinement erneut ausschließlich mit der sicheren synthetischen Ground Truth.
- Der verfeinerte Student wird zum Teacher der nächsten Runde.
- Auch in späteren Runden bleiben die synthetischen Ausgangsdaten somit enthalten.

**Wichtige methodische Einordnung**

- Im ursprünglichen ASTOD stammen die annotierten und nicht annotierten Teilmengen grundsätzlich aus derselben Datenverteilung.
- In dieser Arbeit werden synthetische Daten als annotierte Quelle und reale Formulare als nicht annotiertes Ziel verwendet.
- Damit wird ASTOD als Inspiration für eine Übertragung zwischen zwei unterschiedlichen Verteilungen eingesetzt.
- Ziel ist, die Realitätslücke durch pseudo-annotierte reale Bilder schrittweise zu verkleinern.

**Übergang**

- Auf dieser Grundlage kann der konkrete Versuchsaufbau eingeordnet werden.

# Folie 14 – Versuchsaufbau

**Kernaussage**

- Detektor, Klasse und Auswertung bleiben gleich; verglichen werden unterschiedliche Trainingsstrategien.

**Sprechpunkte**

- Verwendet wird Faster R-CNN mit ResNet-50 und Feature Pyramid Network.
- Die einzige Erkennungsklasse ist Text; Hintergrund wird intern behandelt.
- Trainiert wird mit **Stochastic Gradient Descent**, kurz SGD.
- SGD berechnet aus kleinen Teilmengen der Trainingsdaten eine Richtung, in der die Modellgewichte zur Verringerung des Fehlers verändert werden.
- Ausgewertet werden Precision, Recall und F1 bei einem IoU-Grenzwert von 0,5.
- Der erste Teacher wird auf synthetischen Seiten trainiert.
- Im regulären Self-Training lernt der Student aus synthetischer Ground Truth und pseudo-annotierten realen Seiten.
- Dadurch lassen sich die späteren Unterschiede den Trainingsstrategien zuordnen, weil Detektor und Bewertungsverfahren gleich bleiben.

**Vertiefung bei Rückfragen**

- Das auf COCO vortrainierte Netz wird nicht nur als fester Merkmalsextraktor verwendet; das gesamte Netz wird gemeinsam weitertrainiert.
- Trainiert werden damit unter anderem die Filter von ResNet-50, die FPN-Schichten, das RPN sowie Klassifikations- und Box-Regressionskopf.

**Übergang**

- Vor den Hauptvergleichen wird zunächst eine stabile Optimierereinstellung gesucht.

# Folie 15 – Hyperparametersuche

**Kernaussage**

- FUNSD und Faster R-CNN liefern Ausgangswerte; die endgültige Einstellung wird in eigenen Versuchen systematisch gewählt.

**Sprechpunkte**

- Ein **Hyperparameter** ist eine vor dem Training festgelegte Einstellung und kein direkt aus den Daten gelerntes Modellgewicht.
- Die Ausgangswerte orientieren sich an FUNSD und Faster R-CNN.
- In dieser Arbeit werden mehrere Einstellungen systematisch miteinander verglichen. Daher sprechen wir von einer Hyperparametersuche und nicht einfach von unveränderten FUNSD-Werten.
- Für die Hauptversuche werden Learning Rate 0,006, Momentum 0,8 und Weight Decay 0,0005 festgehalten.

**Je ein Satz pro Begriff**

- **Learning Rate:** Sie bestimmt, wie stark die Modellgewichte bei einem Optimierungsschritt verändert werden.
- **Momentum:** Es berücksichtigt vorherige Aktualisierungsrichtungen und glättet dadurch schwankende Gewichtsänderungen.
- **Weight Decay:** Es bestraft sehr große Gewichte und wirkt damit als Regularisierung gegen Überanpassung.

**Übergang**

- Mit dieser festen Einstellung betrachten wir zunächst die Ausgangsleistung ohne Self-Training.

# Folie 16 – Ausgangsleistung ohne Self-Training

**Kernaussage**

- Rein synthetisches Training bleibt deutlich hinter Training mit realen Annotationen zurück.

**Sprechpunkte**

- Beide Modelle werden auf denselben realen FUNSD-Testseiten ausgewertet.
- Nur synthetische Trainingsdaten erreichen einen F1-Wert von 0,70.
- Reale Trainingsannotation erreicht einen F1-Wert von 0,80.
- Die Differenz von ungefähr 0,10 F1 zeigt die Realitätslücke vor dem Self-Training.
- Diese reale Aufsicht dient als starke Vergleichsgröße, benötigt jedoch die manuellen Annotationen, die eigentlich minimiert werden sollen.

**Übergang**

- Nun wird untersucht, wie weit wiederholtes Self-Training diese Lücke verkleinern kann.

# Folie 17 – Entwicklung über viele Self-Training-Runden

**Kernaussage**

- Self-Training verbessert das synthetische Ausgangsmodell vor allem in frühen Runden und erreicht später ein Plateau.

**Sprechpunkte**

- Auf der horizontalen Achse sehen wir die Self-Training-Runden, auf der vertikalen Achse den F1-Wert.
- Der F1-Wert steigt von ungefähr 0,708 auf maximal 0,7726.
- Der größte Leistungszuwachs entsteht in den frühen Runden.
- Danach schwankt die Kurve in einem Plateau, anstatt dauerhaft weiter anzusteigen.
- Self-Training verkleinert damit die Lücke zum real annotierten Modell deutlich, schließt sie aber nicht vollständig.
- Der beste einzelne Checkpoint ist nicht automatisch ein stabiler Durchschnitt; deshalb sollte das Ergebnis als beobachtetes Maximum eingeordnet werden.

**Begriff**

- **Checkpoint:** gespeicherter Modellzustand zu einem bestimmten Zeitpunkt des Trainings.

**Übergang**

- Neben dem Self-Training wurde untersucht, wie die Art der synthetischen Bilder das Ergebnis beeinflusst.

# Folie 18 – Zwei Varianten der synthetischen Seiten

**Kernaussage**

- Verglichen werden ein rekonstruierter Scan-Hintergrund und ein vereinfachter weißer Hintergrund.

**Sprechpunkte**

- Links wird der alte Text durch Inpainting entfernt und der neue Text direkt in den rekonstruierten Bildbereich eingesetzt.
- Dadurch bleiben Scanstruktur, Linien und lokale Hintergrundinformationen eher erhalten.
- Rechts wird die ursprüngliche Region zunächst durch ein weißes Rechteck ersetzt.
- Diese Variante ist einfacher, kann aber unnatürlich saubere Flächen im ansonsten verrauschten Scan erzeugen.
- Der Vergleich prüft, ob ein realistischerer lokaler Hintergrund die Übertragung auf reale Seiten verbessert.

**Übergang**

- Die Ergebnisse dieses direkten Vergleichs sind auf der nächsten Folie zusammengefasst.

# Folie 19 – Vergleich der Erzeugungsverfahren

**Kernaussage**

- Der rekonstruierte Hintergrund ist leicht besser, der Unterschied ist jedoch klein.

**Sprechpunkte**

- Das Einfügen in den rekonstruierten Hintergrund erreicht ungefähr 0,76 F1.
- Die Variante mit weißem Hintergrund erreicht ungefähr 0,75 F1.
- Der erhaltene Scan-Kontext scheint damit einen kleinen Vorteil zu liefern.
- Wegen des geringen Abstands sollte daraus jedoch keine starke allgemeine Überlegenheit abgeleitet werden.

**Übergang**

- Zusätzlich stellt sich die Frage, wie viel synthetisches Material überhaupt benötigt wird.

# Folie 20 – Einfluss der Menge synthetischer Daten

**Kernaussage**

- Auch kleinere synthetische Teilmengen sind nutzbar; der vollständige lange Lauf erzielt das beste Ergebnis.

**Sprechpunkte**

- Mit zehn Prozent der synthetischen Daten wird bereits ein F1-Wert von ungefähr 0,72 erreicht.
- Mit 25 Prozent steigt der Wert auf ungefähr 0,75.
- Der vollständige lange Lauf erreicht ungefähr 0,77.
- Damit zeigt sich auch bei kleineren Teilmengen ein Nutzen des Self-Trainings.
- Die Ergebnisse gehören jedoch zu den jeweils beschriebenen Versuchsläufen und sollten nicht als vollständig isolierte Skalierungskurve überinterpretiert werden.

**Übergang**

- Anschließend wird geprüft, ob die synthetischen Daten nach Erreichen des Plateaus überhaupt noch benötigt werden.

# Folie 21 – Ablation: synthetische Daten entfernen

**Kernaussage**

- Synthetische Daten werden erst ab fünf ausgewählten Plateau-Checkpoints entfernt; die Fortsetzungen werden dann mit den regulären Fortsetzungen verglichen.

**Sprechpunkte**

- Eine **Ablation** entfernt gezielt einen Bestandteil des Verfahrens, um dessen Beitrag zu untersuchen.
- Ausgangspunkt sind fünf Checkpoints mit hohem F1-Wert aus dem Plateau der regulären Self-Training-Kurve.
- In der regulären Fortsetzung lernt der Student weiterhin aus synthetischen und pseudo-annotierten realen Seiten; danach folgt das synthetische Refinement.
- In der Variante „nur Pseudo“ werden die synthetischen Seiten **ab dem gewählten Checkpoint** vollständig aus dem weiteren Training entfernt.
- Dadurch verbleiben nur pseudo-annotierte reale Seiten, und das synthetische Refinement kann ebenfalls nicht mehr stattfinden.
- Die Tabelle vergleicht jeweils den F1-Wert der regulären Fortsetzung mit der zugehörigen Pseudo-only-Fortsetzung desselben Ausgangs-Checkpoints.
- Alle fünf Differenzen sind negativ. Das Entfernen der synthetischen Grundlage verschlechtert damit jede getestete Fortsetzung, auch wenn der Rückgang relativ klein bleibt.

**Ein besonders klarer Satz für den Vortrag**

- „Ab den ausgewählten Plateau-Checkpoints werden die synthetischen Seiten vollständig aus dem weiteren Training entfernt; dadurch entfallen sowohl ihr Anteil am Studententraining als auch das anschließende synthetische Refinement.“

**Übergang**

- Zum Schluss der Experimente wird betrachtet, ob synthetische Daten auch dann noch helfen, wenn reale Annotationen bereits vorhanden sind.

# Folie 22 – Synthetische Daten bei realer Aufsicht

**Kernaussage**

- Synthetische Daten helfen bei realer Aufsicht nur dann leicht, wenn sie über Self-Training eingebunden werden.

**Sprechpunkte**

- Nur reale Annotationen erreichen 0,8011 F1.
- Das direkte gemeinsame Training mit realen und synthetischen Daten erreicht 0,8007 und liefert damit keinen erkennbaren Gewinn.
- Die Einbindung über Self-Training erreicht 0,8144 F1.
- Der Vorteil ist klein, zeigt aber, dass die Trainingsstrategie wichtiger sein kann als das bloße Hinzufügen weiterer Daten.
- Dieses Experiment ist von der synthetisch gestarteten Hauptfrage zu unterscheiden, weil hier reale Annotationen als Aufsicht vorhanden sind.

**Übergang**

- Aus diesen Einzelversuchen lassen sich vier zentrale Ergebnisse zusammenfassen.

# Folie 23 – Zentrale Ergebnisse

**Kernaussage**

- Self-Training verkleinert die Realitätslücke deutlich, ersetzt reale Annotationen aber nicht vollständig.

**Sprechpunkte**

- Erstens erreicht rein synthetisches Training 0,70 F1, während reale Annotationen 0,80 F1 erreichen.
- Zweitens erhöht synthetisch-reales Self-Training den F1-Wert auf ungefähr 0,77.
- Drittens verschlechtern sich alle getesteten Fortsetzungen, wenn die synthetische Grundlage im Plateau entfernt wird.
- Viertens steigt die Leistung bei vorhandener realer Aufsicht durch Self-Training mit synthetischer Unterstützung auf 0,814 F1.
- Insgesamt sind synthetische Daten damit ein nützlicher Ausgangspunkt und eine stabilisierende Datenquelle, aber kein vollständiger Ersatz für reale Annotationen.

**Übergang**

- Diese Aussagen müssen vor dem Fazit durch die Grenzen der Untersuchung eingeordnet werden.

# Folie 24 – Grenzen der Untersuchung

**Kernaussage**

- Die Ergebnisse sind aussagekräftig für den Versuchsaufbau, aber nicht ohne Weiteres auf alle Dokumente und Detektoren übertragbar.

**Sprechpunkte**

- Die Hauptauswertung verwendet nur einen Formulardatensatz und nur die Klasse Text.
- Die synthetischen Seiten übernehmen Layouts bereits annotierter Ausgangsformulare; völlig neue Layoutstrukturen werden dadurch nur begrenzt abgedeckt.
- Schriftarten und Handschrift variieren weniger stark als in realen Anwendungen.
- Die Ergebnisse wurden nicht über viele unabhängige Zufallsinitialisierungen gemittelt.
- Ein eigener Validierungssatz hätte die Modellauswahl klarer vom endgültigen Testsatz getrennt.
- Deshalb sollten kleine Ergebnisunterschiede vorsichtig interpretiert werden.

**Begriffe**

- **Zufallsinitialisierung beziehungsweise Random Seed:** kontrolliert zufällige Abläufe wie Gewichtsinitialisierung oder Datenreihenfolge; mehrere Seeds zeigen, wie stabil ein Ergebnis ist.
- **Validierungssatz:** separate Datenmenge zur Auswahl von Einstellungen und Checkpoints, bevor der Testsatz einmalig für die Abschlussbewertung verwendet wird.

**Übergang**

- Unter Berücksichtigung dieser Grenzen ergibt sich folgendes Fazit.

# Folie 25 – Fazit und Ausblick

**Kernaussage**

- Synthetische Daten und Self-Training reduzieren den Bedarf an zusätzlichen realen Box-Annotationen, lassen aber noch Verbesserungspotenzial.

**Sprechpunkte**

- Synthetische Daten bilden einen brauchbaren automatischen Ausgangspunkt für die Textdetektion.
- Self-Training bindet reale Seiten ein, ohne für jede dieser Seiten neue manuelle Bounding Boxes zu benötigen.
- Die synthetische Ground Truth bleibt in den regulären Runden erhalten und stabilisiert das Training.
- Reale Annotationen liefern dennoch weiterhin die stärkste direkte Aufsicht.
- Zukünftig sollten realistischere Schriftarten, Handschrift und größere Layoutvariation erzeugt werden.
- Zusätzlich sind weitere Datensätze, neuere Detektoren, mehrere Zufallsinitialisierungen und ein eigener Validierungssatz sinnvoll.

**Schlusssatz**

- Zusammenfassend zeigt die Arbeit, dass synthetisch gestartetes Self-Training die Lücke zu realen Daten deutlich verkleinern und damit einen weiteren Schritt zu einer stärker automatisierten Formularverarbeitung leisten kann. Vielen Dank für Ihre Aufmerksamkeit.

# Folie 26 – Literatur

**Sprechpunkte**

- Auf dieser Folie sind die zentralen Quellen zu FUNSD, RVL-CDIP, Faster R-CNN, ASTOD und Inpaint Anything aufgeführt.
- Bei Rückfragen zur Methodik kann gezielt auf die jeweilige Originalquelle verwiesen werden.

# Begriffsreserve für Rückfragen

| Begriff | Kurze, sichere Definition |
|---|---|
| Annotation | Manuell oder automatisch zugeordnete Referenzinformation, hier meist eine Bounding Box. |
| Bounding Box | Rechteck, das Lage und Größe einer Textregion beschreibt. |
| Ground Truth | Als korrekt angenommene Referenzannotation für Training oder Auswertung. |
| OCR | Optische Zeichenerkennung; wandelt Schrift in Bildern in maschinenlesbaren Text um. |
| Faltung / Convolution | Gelernte Filter werden lokal über ein Bild oder eine Merkmalskarte verschoben und erzeugen neue Merkmale. |
| ResNet-50 | Tiefes faltungsbasiertes neuronales Netz mit Residual-Verbindungen; hier zur Merkmalsextraktion. |
| Feature Map | Räumlich angeordnete Aktivierungen, die gelernte Bildmerkmale repräsentieren. |
| FPN | Feature Pyramid Network; kombiniert mehrere Größenebenen für kleine und große Objekte. |
| RPN | Region Proposal Network; erzeugt mögliche Objektregionen für die zweite Detektionsstufe. |
| Ankerbox | Vordefinierte Ausgangsbox einer bestimmten Größe und Form. |
| Objectness | Geschätzte Wahrscheinlichkeit, dass eine Box irgendein relevantes Objekt enthält. |
| RoI Align | Liest Merkmale eines Regionenvorschlags aus und bringt sie auf eine einheitliche Größe. |
| Box-Regression | Vorhersage von Korrekturwerten für Position, Breite und Höhe einer Box. |
| NMS | Non-Maximum Suppression; entfernt stark überlappende Mehrfachvorhersagen zugunsten der sichersten Box. |
| IoU | Überschneidungsfläche geteilt durch Vereinigungsfläche von Vorhersage und Referenz. |
| Precision | Anteil richtiger Boxen unter allen vorhergesagten Boxen. |
| Recall | Anteil gefundener Boxen unter allen vorhandenen Referenzboxen. |
| F1 | Harmonisches Mittel aus Precision und Recall. |
| Teacher | Modell, das Pseudo-Annotationen erzeugt. |
| Student | Modell, das aus sicheren und pseudo-annotierten Daten trainiert wird. |
| Pseudo-Annotation | Modellvorhersage, die vorübergehend als Trainingsannotation genutzt wird. |
| Confidence Score | Vom Modell geschätzte Sicherheit einer Vorhersage. |
| Loss | Zahlenwert für den Trainingsfehler, der durch Gewichtsaktualisierungen minimiert wird. |
| SGD | Optimierungsverfahren, das Gewichte anhand von Gradienten kleiner Daten-Teilmengen aktualisiert. |
| Learning Rate | Stärke eines einzelnen Gewichtsaktualisierungsschritts. |
| Momentum | Geglätteter Einfluss vorheriger Aktualisierungsrichtungen. |
| Weight Decay | Regularisierung, die sehr große Gewichte bestraft. |
| Hyperparameter | Vor dem Training festgelegte Einstellung, die nicht direkt als Modellgewicht gelernt wird. |
| Checkpoint | Gespeicherter Modellzustand während oder nach einer Trainingsphase. |
| Ablation | Gezieltes Entfernen eines Verfahrensbestandteils zur Untersuchung seines Beitrags. |
| Refinement | Abschließendes Weitertrainieren nur auf sicher annotierten Daten. |
| Datenverteilung | Statistische Eigenschaften der Bilder, Inhalte und Annotationen eines Datensatzes. |

# Vorbereitung ohne Ablesen

1. **Erster Durchlauf:** Nur mit den Folien sprechen. Das Skript anschließend verwenden, um ausgelassene Kernaussagen zu markieren.
2. **Zweiter Durchlauf:** Vortrag aufnehmen und mit Whisper in Text umwandeln.
3. **Transkript prüfen:** Denglische Formulierungen, Füllwörter, sehr lange Sätze und unklare Fachbegriffe markieren.
4. **Wortwahl verbessern:** Beispielsweise „Domain Gap“ durch „Realitätslücke“, „Sweep“ durch „Hyperparametersuche“ und „Labels“ je nach Kontext durch „Annotationen“ ersetzen.
5. **Dritter Durchlauf:** Nur noch Stichwörter und Übergänge ansehen; die ersten zwei Minuten vollständig frei sprechen.
6. **ASTOD-Prüfung:** Die fünf Schritte ohne Folie in der richtigen Reihenfolge erklären können und die Übertragung auf synthetische beziehungsweise reale Daten ausdrücklich nennen.
7. **Zeitreserve:** Detailpunkte zu RPN, ASTOD-Histogramm und Optimierung nur bei Rückfragen oder verbleibender Zeit verwenden.
