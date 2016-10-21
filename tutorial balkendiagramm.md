#Balkendiagramm mit Javascript

Das folgende Tutorial versucht eine Schritt-für-Schritt-Einführung in die Visualisierung von Wahlergebnissen mittels Javascript/d3js zu geben.

###Voraussetzungen

* eine grobe Vorstellung, wie eine HTML-Seite aussieht
* eine ungefähre Vorstellung von Javascript (link Folien Peter?)

Vorab kann man auch einen Blick in dieses Tutorial werfen, das einige grundlegende Punkte zum Thema Visualisierung von Wahldaten anschneidet: [Tutorial offenewahlen.at](https://github.com/ginseng666/offenewahlen/blob/gh-pages/visualisierungen.md)


###Nützliches
Der Code kann in jedem Textprogramm geschrieben werden, ein etwas besserer Texteditor hat aber viele Vorteile, wie etwa die Nummerierung der Zeilen. Zwei sehr gute (freie) Programme:
* [Notepad++](https://notepad-plus-plus.org/)
* [Sublime](https://www.sublimetext.com/)

Wenn man Code schreibt, passieren zwangsläufig jede Menge Fehler, vom Tippfehler bis zur vergessenen Klammer. Damit man diese Fehler auch sehen kann, sollte man beim Testen die Javascript-Console im Browser offen haben. Mit F12 öffnet man in verschiedenen Browsern mehrere Entwicklungstools, unter denen sich auch die Console befindet.

![Screenshot Console](https://github.com/ginseng666/graz_19102016/blob/master/img/console.jpg) 
(Console in Firefox 49)

Ebenfalls nützlich ist der Inspector (Firefox)/Elements (Chrome)/Dom Inspector (Explorer): Er zeigt den gesamten Inhalt der Seite an. Wenn man also ein Element darstellen will, es aber nicht erscheint und auch kein Fehler in der Console ausgegeben wird, dann kann man es hier suchen (und findet vermutlich den Grund, warum es nicht dargestellt wird - etwa fehlende geschlossene `<>` oder fehlerhafte Koordinaten).

![Screenshot Inspector](https://github.com/ginseng666/graz_19102016/blob/master/img/inspector.jpg) 
(Inspector in Firefox 49)

###Das Grundgerüst

```javascript
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.17/d3.js" charset="utf-8"></script>
</head>
<body>

<script>
</script>

</body>
</html>
```

Dieser Code umreißt die Struktur der Seite: Neben dem `<html>`-Tag am Anfang steht im `<head>` ein Titel und der Zeichensatz wird angegeben (`<meta charset="utf-8">` - siehe [hier](https://developer.mozilla.org/de/docs/Web/HTML/Element/meta#attr-charset)). Danach wird die d3.js-library eingebunden, damit später die entsprechenden Befehle verfügbar sind. Das `</head>`-Tag wird geschlossen, der `<body>` beginnt und enthält noch einen leeren `<script>`-Block. Hier wird der Javascript-Code eingefügt. Anschließend werden der `</body>` und das `</html>` wieder geschlossen.

Diesen Teil kann man standardmäßig als Ausgangsbasis für eine Seite verwenden.


###SVG
Visualisierungen mit d3.js erzeugen so genannte [SVG](https://developer.mozilla.org/en-US/docs/Web/SVG) oder Scalable Vector Graphics - das sind im Kern einfach geometrische Formen wie Rechtecke, Kreise, Linien usw., die in Kombination zu komplexen Formen zusammengesetzt werden können. Ein großer Vorteil eines SVG: Es kann verlustfrei skaliert werden, man kann es also beliebig vergrößern, ohne dass es unscharf oder "pixelig" wird.

Um ein SVG zu definieren benötigt man noch kein Javascript, das geht direkt im HTML:
```
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
</head>
<body>
<svg width="200" height="200"></svg>
</body>
</html>
```

Das Tag `<svg>` erzeugt eine - noch unsichtbare - Fläche in der Struktur unserer Seite, auf der wir jetzt z.B. ein Rechteck zeichnen können. Die Fläche ist 200 Pixel breit und 200 Pixel hoch. Um das Rechteck zu definieren brauchen wir vier Attribute:
* Breite
* Höhe
* x-Koordinate der linken oberen Ecke
* y-Koordinate der linken oberen Ecke

_Wichtig: Alle Positionsangaben auf dem Bildschirm, wie eben die x/y-Koordinate, werden am Computer immer von der linken oberen Ecke des Bildschirms aus berechnet (dort liegt der Punkt x=0, y=0). Von dort geht es nach links und nach unten._

Wir können nun das Rechteck einfach durch Angabe dieser Attribute zeichnen:
```
<svg width="200" height="200">
<rect x="0" y="0" width="50" height="80"></rect>
</svg>
```

Dieser Code erzeugt ein schwarzes Rechteck in der linken oberen Ecke des Bildschirms, das 50 Pixel breit und 80 Pixel hoch ist. Es gibt jede Menge weiterer Attribute, die verwendet werden können, um das Rechteck zu verändern, `fill="red"` färbt es etwa rot ein. Der Blick in die [Dokumentation](https://developer.mozilla.org/en-US/docs/Web/SVG) lohnt sich.

So gesehen kann man alles, was man für ein Balkendiagramm braucht, auch direkt in dieser SVG-Form schreiben. Javascript kann diesen statischen Code aber sehr viel variabler, flexibler und interaktiver machen und einem selbst einige Arbeit abnehmen.


###erste Schritte in Javascript
Wir löschen das SVG-Beispiel wieder und wenden uns Javascript zu. Als erstes ein Test, ob es im Browser auch funktioniert:
```javascript
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.17/d3.js" charset="utf-8"></script>
</head>
<body>

<script>
console.log("Funktioniert");
</script>

</body>
</html>
```

Wenn wir nun die Console (mit F12) öffnen, dann sollte dort das Wort "Funktioniert" stehen. Ist das nicht der Fall (und es steht auch keine Fehlermeldung dort), dann ist Javascript im Browser blockiert (etwa durch Sicherheitseinstellungen) und muss erst aktiviert werden.

![Screenshot Console Funktioniert](https://github.com/ginseng666/graz_19102016/blob/master/img/console_funktioniert.jpg) 


Hilfreich können auch die Zahlenangaben rechts sein: Sie stehen für die Zeile und die Spalte, in der der Befehl steht, der die Ausgabe erzeugt. Ein Klick darauf zeigt den Code an.


Als nächstes beginnen direkt die Vorbereitungen für die Grafik. Wie oben gezeigt brauchen wir zunächst ein SVG, um darauf zeichnen zu können, und dieses SVG braucht eine Breite und eine Höhe. Diese Werte kann man statisch setzen (auf z.B. 400 Pixel). Wenn es allerdings keinen Grund gibt, warum das SVG genau 400 Pixel breit sein soll, dann vergibt man damit aber viel Flexibilität. Besser ist es, die Werte nach dem verfügbaren Platz festzulegen:
```javascript
<script>
var width = window.innerWidth;
var height = window.innerHeight;
</script>
```
Der Befehl `window.innerWidth` gibt die Breite des Fensters als Zahl zurück (siehe [hier](https://developer.mozilla.org/en/docs/Web/API/window/innerWidth), `window.innerHeight` macht das Gleiche für die Höhe. Zum Testen kann man in der Console diese Befehle eingeben, dann erhält man die entsprechenden Werte.

![Screenshot Console innerWidth](https://github.com/ginseng666/graz_19102016/blob/master/img/console_width.jpg) 


Das SVG würde nun das gesamte Browserfenster ausfüllen. Das kann (browserabhängig) dazu führen, dass Scrollbalken erscheinen, da es minimal über den verfügbaren Platz hinausreicht. Daher reduzieren wir die Breite und Höhe auf jeweils 95% und erzeugen das SVG:
```javascript
<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);
</script>
```

Zunächst müssen wir festlegen, wo das SVG angehängt werden soll. Dafür nutzen wir den Befehl `d3.select`, der in d3.js zur Verfügung steht. Wir wählen damit den `<body>` aus und hängen mit `append` ein SVG an. Diesem geben wir die Breite `width` und die Höhe `height`. (_In umfangreicheren Projekten werden SVGs z.B. in ein `div` gezeichnet, da man sie damit einfach positionieren kann._)

Wir speichern das SVG gleich in der Variablen `svg`, um danach einfacher darauf zugreifen zu können. Alternativ könnte man es später auch so aufrufen: `d3.select("body").select("svg")`

Der Inspector im Browser sollte das SVG-Element bereits anzeigen. Am Bildschirm sieht man (noch) nichts).

![Screenshot Inspector SVG](https://github.com/ginseng666/graz_19102016/blob/master/img/inspector_svg.jpg) 


Dieser Schritt zeigt gut, wie die Befehle in d3.js aufgebaut sind: 
* man wählt ein Element aus
* anschließendund hängt man alle Beschreibungen und Befehle an das Element an
* diese Angaben werden mit einem . verbunden

Das kann in einer Zeile passieren, aber auch untereinander geschrieben werden, letzteres ist meistens übersichtlicher.

Schließlich zeichnen wir unser Rechteck:
```javascript
<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);

svg.append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 80)
  .style("fill", "red");

</script>
```

Die Angaben sehen etwas anders aus als oben, entsprechen ihnen aber:
* x und y bezeichnen den Ausgangspunkt des Rechtecks, seine linke obere Ecke
* width und height bestimmen Breite und Höhe
* fill bestimmt die Farbe

Hier wird eine weitere Unterscheidung sichtbar: In d3.js verwendet man `.attr` zur Festlegung von Attributen wie Position und Form, `.style` für die Festlegung der Art und Weise, wie das Element aussehen soll (_teilweise funktionieren die Befehle mit der einen und der anderen Bezeichnung gleichermaßen, man sollte aber konsequent versuchen, den Style vom Rest zu trennen_).


An dieser Stelle: Am meisten lernt man, wenn man Sachen einfach ausprobiert - also einfach einmal die Werte beim Rechteck verändern und schauen, was passiert, und die Console und den Inspector verwenden. Welchen Effekt hat es, wenn das Rechteck größer als das SVG ist? Was ist, wenn man es außerhalb positioniert?


###ein erstes Balkendiagramm
Nach diesen Grundlagen jetzt (endlich) zum Balkendiagramm. Zum Testen verwenden wir das Ergebnis des ersten Wahlgangs der Bundespräsidentenwahl 2016 von [hier](http://www.bmi.gv.at/cms/BMI_wahlen/bundespraes/bpw_2016/Ergebnis.aspx). Ein Balkendiagramm besteht banal aus mehreren Rechtecken nebeneinander, die entweder die Breite oder die Höhe dafür verwenden, Werte anzuzeigen. Das jeweils andere Attribut ist konstant.

Wir versuchen ein Säulendiagramm, verwenden also die Höhe, um einen Wert auszudrücken, als Balkenbreite 50 Pixel. Also zeichnen wir testweise je ein Rechteck für die drei ersten KandidatInnen (alphabetisch sind das Irmgard Griss, Norbert Hofer und Rudolf Hundstorfer):
```javascript
<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);

svg.append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 18.94)
  .style("fill", "grey");

svg.append("rect")
  .attr("x", 55)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 35.05)
  .style("fill", "grey");
  
svg.append("rect")
  .attr("x", 110)
  .attr("y", 0)
  .attr("width", 50)
  .attr("height", 11.28)
  .style("fill", "grey");
</script>
```

Damit die Balken nebeneinander stehen, verschieben wir sie jeweils um die Breite 50 plus 5 weiteren Pixeln, um einen kleinen Abstand zu haben. Beim Attribut `height` verwenden wir den Stimmenanteil der KandidatInnen.

![Screenshot Balken 1](https://github.com/ginseng666/graz_19102016/blob/master/img/balken_1.jpg) 


Das geht schon in die richtige Richtung, intuitiv würde man aber Balken von unten nach oben erwarten. Die Grafik steht auf dem Kopf, da - wie oben geschrieben - das Koordinatensystem oben links beginnt. Nachdem man keine negative Höhe zeichnen kann, müssen wir umdenken: Wir setzen die y-Koordinate (bezeichnet die linke obere Ecke) dorthin, wo die Säule aufhören soll, basierend auf einer gemeinsamen Grundlinie bei 100. Die Höhe bleibt gleich, dafür geben wir jedem Element eine eigene Farbe (für Farbnamen siehe [hier](https://en.wikipedia.org/wiki/Web_colors):

```javascript
svg.append("rect")
  .attr("x", 0)
  .attr("y", 100 - 18.94)
  .attr("width", 50)
  .attr("height", 18.94)
  .style("fill", "purple");

svg.append("rect")
  .attr("x", 55)
  .attr("y", 100 - 35.05)
  .attr("width", 50)
  .attr("height", 35.05)
  .style("fill", "blue");
  
svg.append("rect")
  .attr("x", 110)
  .attr("y", 100 - 11.28)
  .attr("width", 50)
  .attr("height", 11.28)
  .style("fill", "red");
```

![Screenshot Balken 2](https://github.com/ginseng666/graz_19102016/blob/master/img/balken_2.jpg) 


###Variable Größe
Nachdem wir schon die Größe des SVG variabel gemacht haben, liegt es nahe, dasselbe für die Größe des Diagramms zu machen. So können wir die Balkenbreite aus der verfügbaren Breite berechnen, ebenfalls legen wir einen fixen Abstand zwischen den Balken von 10 Pixeln fest:
```javascript
var abstand = 10;
var balken_breite = width / 6 - abstand;
```

Die Zahl 6 steht hier für die sechs KandidatInnen, die das Diagramm letzten Endes darstellen soll. Da wir Prozentangaben verwenden, können wir die Höhe direkt als Maximalhöhe verwenden - jeder Balken soll demnach X Prozent dieser Maximalhöhe einnehmen:
```javascript
var max_hoehe = height;
```

Fügen wir diese Code-Schnipsel in unser bisheriges Script ein, dann passen sich die Balken an den verfügbaren Platz an. Verändert man die Fenstergröße und lädt die Seite neu, dann hat sich die Balkengröße entsprechend verändert (über diesen Weg lassen sich später responsive Visualisierungen umsetzen). Zu beachten: Wir dividieren die Werte jeweils durch 100, um den Prozentwert zu erhalten.
```javascript
svg.append("rect")
  .attr("x", 0)
  .attr("y", max-hoehe - max_hoehe * 18.94 / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * 18.94 / 100)
  .style("fill", "purple");

svg.append("rect")
  .attr("x", balken_breite + abstand)
  .attr("y", max-hoehe - max_hoehe * 35.05 / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * 35.05 / 100)
  .style("fill", "blue");
  
svg.append("rect")
  .attr("x", balken_breite * 2 + abstand * 2)
  .attr("y", max_hoehe - max_hoehe * 11.28 / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * 11.28 / 100)
  .style("fill", "red");
```

###Automatisierung
Bisher haben wir, trotz einiger Berechnungen, die Balken praktisch von Hand gezeichnet. Wir haben drei Blöcke für drei Säulen geschrieben, die großteils identisch sind und sich nur durch die Werte unterscheiden. Höchste Zeit also, Arbeit an den Computer zu übergeben.
Um das zu tun, müssen wir unsere Ergebnisse zuerst in einem anderen Variablen-Typ speichern, einem so genannten _array_. Das ist im Grunde nichts anderen als eine Liste von Werten und sieht so aus:
```javascript
var ergebnis = [18.94, 35.05, 11.28, 11.12, 2.26, 21.34];
```

Die Werte stehen zwischen `[]` und sind alphabetisch geordnet (nun alle sechs KandidatInnen). Arrays können nicht nur Zahlen, sondern beliebige Inhalte speichern, das wird später noch wichtig. 

Ein großer Vorteil von arrays ist, dass man direkt den Wert an Position x ansprechen kann, und zwar so: `ergebnis[2]`

Gibt man das in die Console ein, dann erhält man den Wert an Position zwei, konkret 11.28. Achtung: Arrays nummerieren ihren Inhalt immer beginnend mit 0, nicht mit 1. 

Um jetzt den array gut nutzen zu können, brauchen wir noch eine andere Technik, eine so genannte for-Schleife oder for-loop (siehe [hier](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Statements/for). Damit sagt man dem Programm, dass es einen Block Code hintereinander mehrmals ausführen soll. Angewandt auf unser Programm sieht das so aus:
```javascript
var ergebnis = [18.94, 35.05, 11.28, 11.12, 2.26, 21.34];
var farben = ["purple", "blue", "red", "black", "yellow", "green"];

for (var i = 0; i < ergebnis.length; i++)
  {
  svg.append("rect")
    .attr("x", balken_breite * i + abstand * i)
    .attr("y", max_hoehe - max_hoehe * ergebnis[i] / 100)
    .attr("width", balken_breite)
    .attr("height", max_hoehe * ergebnis[i] / 100)
    .style("fill", farben[i]);
  }
```
Hier passiert Folgendes: Wir definieren zunächst den array mit den Ergebnissen, und auch einen zweiten array mit Farben für die KandidatInnen. Die Farben sind gleich geordnet wie die Ergebnisse. Dann beginnt die for-Schleife: Wir legen zunächst eine Zählvariable `i` an und setzen sie auf 0 (die Anfangsposition im array). Dann müssen wir der Schleife sagen, wie lange sie laufen soll - hier so lange, so lange ihr Wert kleiner als die Länge des Ergebnis-arrays ist, sprich bis alle Werte abgearbeitet sind. 

Schließlich muss man noch definieren, was mit der Zählvariablen in jedem Durchgang passieren soll: Wir wollen den array Wert für Wert durcharbeiten, also erhöhen wir `i` um eins, dafür steht `i++` (alternativ könnte man schreiben: `i = i + 1`). Anschließend öffnet man eine geschwungene Klammer und ergänzt den Code, der wiederholt werden soll - und schließt die geschwungene Klammer wieder.

Der Code ist wiederum identisch mit den vorherigen Beispielen, wir ersetzen nur die zuvor statischen Werte durch variable Angaben: `balkenbreite * i` multipliziert die Variable balkenbreite mit dem Wert der Zählvariablen, um jeden Balken weiter nach rechts zu rücken. Beim ersten Durchlauf ist `i` 0, x ist also insgesamt auch 0, beim zweiten 1, x also 60 usw.

Bei `y` ersetzen wir den fixen Wert durch den Wert im array ergebnis, der an Stelle `i` steht, das gleiche machen wir bei der Höhe. Abschließend holen wir uns noch die Farbe aus dem Farben-array, wiederum über die Position.
![Screenshot Balken 3](https://github.com/ginseng666/graz_19102016/blob/master/img/balken_3.jpg) 

Mit der Schleife haben wir den Code erheblich verkürzt. Zusätzlich brauchen wir nur die Werte im array ändern, um andere Ergebnisse abzubilden, unser Diagramm passt sich automatisch an die Zahl der Werte an. Um das zu perfektionieren, müssen wir noch die Berechnung der Balkenbreite anpassen:
```javascript
var balken_breite = width / ergebnis.length - abstand;
```

Achtung, diese Definition muss im Code unterhalb des Ergebnis-arrays stehen, da Javascript sonst eine Variable sucht, die noch nicht definiert ist. Jetzt wäre es Zeit, ein wenig zu experimentieren, also die Werte zu ändern, weitere einzugeben oder vorhandene zu löschen und zu testen, was dann passiert. Auch die Farben kann man ändern, es gibt sehr viel schönere Farben als die archetypischen rot/blau/grün-Töne, z.B. [hier](http://tristen.ca/hcl-picker/#/hlc/6/1/15534C/E2E062)


###Objekte
Das Schöne (oder Schreckliche) am Programmieren ist, dass man praktisch immer Sachen verbessern oder ausbauen kann. Wir haben zwar schon einiges erreicht, aber ein paar Sachen sind noch unbefriedigend. So sind die Balken etwa nicht nach Größe sortiert, was bei einer Ergebnis-Visualisierung aber sinnvoll wäre. Das kann man händisch machen, muss dann aber auch die Farben händisch anpassen. Es wäre also gut, wenn wir das Ergebnis und die dazugehörige Farbe verbinden könnten.

Das lässt sich in Javascript über ein [object](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Object) erreichen. Sehr vereinfacht ist das ein, naja, Objekt mit beliebigen Eigenschaften, z.B.:
```javascript
var mensch = {"name": "Sepp", "alter":17, "hobby":"Daten visualisieren"};
```

Auf diese Weise haben wir object `mensch` definiert. Dieses object hat den Namen "Sepp" und das Alter 17. Man kann alle einzelnen Eigenschaften ansprechen, indem man object.eigenschaft schreibt - oder konkret `console.log(mensch.name, mensch.alter, mensch.hobby);`.

Ein object besteht immer aus einem oder mehreren key/value-Paaren, die durch , voneinander getrennt sind: Unter Anführungszeichen steht zunächst ein key, hier eben z.B. `"name"`, dann folgt ein `:` und dann der value `"Sepp"`. Anders als ein array stehen die Inhalte eines objects zwischen `{}`.

Zu beachten: Keys brauchen immer Anführungszeichen, Werte je nach ihrem Typ. Strings, also Texte, brauchen Anführungszeichen, Zahlen aber nicht. Man kann auch Variable als value in einem object speichern, arrays, Formeln, Funktionen, weitere objects usw.

Für unser Beispiel brauchen wir also ein object, dass das Ergebnis und die Farbe des/der KandidatIn enthält - und weils leicht geht, nehmen wir den Namen auch noch mit und schreiben die Variable ergebnis um:
```javascript
var ergebnis = {"name":"Griss", "wert":18.94, "farbe":"#91678A"};
```

Wir brauchen insgesamt sechs solcher objects als Liste, um sie anschließend in unsere Schleife schicken zu können: Also stellen wir einen array zusammen, der diese objects enthält:
```javascript
var ergebnis = [
	{"name":"Griss", "wert":18.94, "farbe":"#91678A"},
	{"name":"Hofer", "wert":35.05, "farbe":"#356F7F"},
	{"name":"Hundstorfer", "wert":11.28, "farbe":"#B7615A"},
	{"name":"Khol", "wert":11.12, "farbe":"#000000"},
	{"name":"Lugner", "wert":2.26, "farbe":"#E2E062"},
	{"name":"Van der Bellen", "wert":21.34, "farbe":"#437C4F"},
	];
```

Dann passen wir noch den Code in der for-Schleife an - da unser array jetzt keine einfachen Zahlen mehr enthält, sondern eben mehrere objects, müssen wir einfach die jeweils notwendige Eigenschaft abrufen:

```javascript
svg.append("rect")
  .attr("x", balken_breite * i + abstand * i)
  .attr("y", max_hoehe - max_hoehe * ergebnis[i].wert / 100)
  .attr("width", balken_breite)
  .attr("height", max_hoehe * ergebnis[i].wert / 100)
  .style("fill", ergebnis[i].farbe);
```

Weiterhin wird die Balkenbreite und die Zahl der Balken automatisch berechnet, wenn man Einträge löscht oder ergänzt, dann verändert sich die Darstellung. Bei den Farben verwenden wir hier übrigens testweise den Hex-Wert, da uns das mehr Möglichkeiten gibt.

Zurück zu dem, warum wir eigentlich ein object gebraucht haben: Wir wollen die Balken nach Größe sortieren. Auch das lässt sich sehr einfach mit Javascript lösen, wir tragen nach dem object (und vor der Schleife) folgenden Code ein:
```javascript
ergebnis.sort(function(a, b) { return a.wert < b.wert; });
```

Dieser Code geht durch den array und vergleicht immer zwei Werte - a und b - und sortiert sie entsprechend um. Das `<` führt zu einer absteigenden Sortierung - was `>` macht sollte man ausprobieren. Wichtig ist auch hier: Wir können nicht direkt die Werte im array abrufen, sondern wir rufen jeweils einen Eintrag im array auf, und dann die entsprechende Eigenschaft. So weit, so gut:

![Screenshot Balken 4](https://github.com/ginseng666/graz_19102016/blob/master/img/balken_4.jpg) 
