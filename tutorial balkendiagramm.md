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

![Screenshot Console](https://github.com/ginseng666/graz_19102016/blob/master/console.jpg) 
(Console in Firefox 49)

Ebenfalls nützlich ist der Inspector (Firefox)/Elements (Chrome)/Dom Inspector (Explorer): Er zeigt den gesamten Inhalt der Seite an. Wenn man also ein Element darstellen will, es aber nicht erscheint und auch kein Fehler in der Console ausgegeben wird, dann kann man es hier suchen (und findet vermutlich den Grund, warum es nicht dargestellt wird - etwa fehlende geschlossene `<>` oder fehlerhafte Koordinaten).

![Screenshot Inspector](https://github.com/ginseng666/graz_19102016/blob/master/inspector.jpg) 
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

![Screenshot Console Funktioniert](https://github.com/ginseng666/graz_19102016/blob/master/console_funktioniert.jpg) 


Hilfreich können auch die Zahlenangaben rechts sein: Sie stehen für die Zeile und die Spalte, in der der Befehl steht, der die Ausgabe erzeugt. Ein Klick darauf zeigt den Code an.


Als nächstes beginnen direkt die Vorbereitungen für die Grafik. Wie oben gezeigt brauchen wir zunächst ein SVG, um darauf zeichnen zu können, und dieses SVG braucht eine Breite und eine Höhe. Diese Werte kann man statisch setzen (auf z.B. 400 Pixel). Wenn es allerdings keinen Grund gibt, warum das SVG genau 400 Pixel breit sein soll, dann vergibt man damit aber viel Flexibilität. Besser ist es, die Werte nach dem verfügbaren Platz festzulegen:
```javascript
<script>
var width = window.innerWidth;
var height = window.innerHeight;
</script>
```
Der Befehl `window.innerWidth` gibt die Breite des Fensters als Zahl zurück (siehe [hier](https://developer.mozilla.org/en/docs/Web/API/window/innerWidth), `window.innerHeight` macht das Gleiche für die Höhe. Zum Testen kann man in der Console diese Befehle eingeben, dann erhält man die entsprechenden Werte.

![Screenshot Console innerWidth](https://github.com/ginseng666/graz_19102016/blob/master/console_width.jpg) 


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

![Screenshot Inspector SVG](https://github.com/ginseng666/graz_19102016/blob/master/inspector_svg.jpg) 


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

![Screenshot Balken 1](https://github.com/ginseng666/graz_19102016/blob/master/balken_1.jpg) 


Das geht schon in die richtige Richtung, intuitiv würde man aber Balken von unten nach oben erwarten. Die Grafik steht auf dem Kopf, da - wie oben geschrieben - das Koordinatensystem oben links beginnt. Nachdem man keine negative Höhe zeichnen kann, müssen wir umdenken: Wir setzen die y-Koordinate (bezeichnet die linke obere Ecke) dorthin, wo die Säule aufhören soll, basierend auf einer gemeinsamen Grundlinie bei 100. Die Höhe bleibt gleich, dafür geben wir jedem Element eine eigene Farbe:

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

![Screenshot Balken 2](https://github.com/ginseng666/graz_19102016/blob/master/balken_2.jpg) 
