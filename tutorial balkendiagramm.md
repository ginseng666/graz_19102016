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

Ebenfalls nützlich ist der Inspector (Firefox)/Elements (Chrome)/Dom Inspector (Explorer): Er zeigt den gesamten Inhalt der Seite an. Wenn man also ein Element darstellen will, es aber nicht erscheint und auch kein Fehler in der Console ausgegeben wird, dann kann man es hier suchen (und findet vermutlich den Grund, warum es nicht dargestellt wird - etwa fehlende geschlossene <> oder fehlerhafte Koordinaten).


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

Dieser Code umreißt die Struktur der Seite: Neben dem `<html>`-Tag am Anfang steht im `<head>` ein Titel und der Zeichensatz wird angegeben (`<meta charset="utf-8">`). Danach wird die d3.js-library eingebunden, damit später die entsprechenden Befehle verfügbar sind. Das `</head>`-Tag wird geschlossen, der `<body>` beginnt und enthält noch einen leeren `<script>`-Block. Hier wird der Javascript-Code eingefügt. Anschließend werden der `</body>` und das `</html>` wieder geschlossen.

Diesen Teil kann man standardmäßig als Ausgangsbasis für eine Seite verwenden.


###Die Grafik - SVG
Visualisierungen mit d3.js erzeugen so genannte [SVG](https://developer.mozilla.org/en-US/docs/Web/SVG) oder Scalable Vector Graphics - das sind im Kern einfach geometrische Formen wie Rechtecke, Kreise, Linien usw., die in Kombination zu komplexen Formen zusammengesetzt werden können. Ein großer Vorteil eines SVG: Es kann verlustfrei skaliert werden, man kann es beliebig vergrößern.

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

Wir können nun das Rechteck einfach durch Angabe der Attribute zeichnen:
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

Als nächstes beginnen direkt die Vorbereitungen für die Grafik. Wie oben gezeigt brauchen wir zunächst ein SVG, um darauf zeichnen zu können, und dieses SVG braucht eine Breite und eine Höhe. Diese Werte kann man statisch setzen (auf z.B. 400 Pixel), damit vergibt man aber viel Flexibilität (wenn es nicht gewollt ist, eine statische Größe zu haben). Besser ist es, die Werte nach dem verfügbaren Platz festzulegen:
```javascript
<script>
var width = window.innerWidth;
var height = window.innerHeight;
</script>
```
Der Befehl `window.innerWidth` gibt die Breite des Fensters als Zahl zurück (siehe [hier](https://developer.mozilla.org/en/docs/Web/API/window/innerWidth), `window.innerHeight` macht das Gleiche für die Höhe. Zum Testen kann man in der Console diese Befehle eingeben, dann erhält man die entsprechenden Werte.

Das SVG würde nun das gesamte Browserfenster ausfüllen. Das kann (browserabhängig) dazu führen, dass Scrollbalken erscheinen und es minimal über den verfügbaren Platz hinausreicht. Daher reduzieren wir die Breite und Höhe um jeweils 20 Pixel und erzeugen das SVG:
```javascript
<script>
var width = window.innerWidth - 20;
var height = window.innerHeight - 20;

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);
</script>
```

Zunächst müssen wir festlegen, wo das SVG angehängt werden soll. Dafür nutzen wir den Befehl `d3.select`, der in d3.js zur Verfügung steht. Wir wählen damit den `<body>` aus und hängen mit `append` ein SVG an. Diesem geben wir die Breite `width` und die Höhe `height`.

Wir speichern das SVG gleich in der Variablen `svg`, um danach einfacher darauf zugreifen zu können. Alternativ könnte man es später auch so aufrufen: `d3.select("body").select("svg")`

Der Inspector im Browser sollte das SVG-Element bereits anzeigen. Am Bildschirm sieht man (noch) nichts).

Dieser Schritt zeigt gut, wie die Befehle in d3.js aufgebaut sind: Man wählt ein Element aus und hängt dann Befehle schrittweise jeweils mit . daran an. Das kann in einer Zeile passieren, aber auch untereinander geschrieben werden, letzteres ist meistens übersichtlicher. Wichtig ist nur, dass jeder neue Befehl mit . beginnt.

