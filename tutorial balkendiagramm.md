#Balkendiagramm mit Javascript

Das folgende Tutorial versucht eine Schritt-für-Schritt-Einführung in die Visualisierung von Wahlergebnissen mittels Javascript/d3js zu geben.

###Voraussetzungen

*eine grobe Vorstellung, wie eine HTML-Seite aussieht
*eine ungefähre Vorstellung von Javascript (link Folien Peter?)

Vorab kann man auch einen Blick in dieses Tutorial werfen, das einige grundlegende Punkte zum Thema Visualisierung von Wahldaten anschneidet.


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

_Ein wichtiger Hinweis an der Stelle: Alle Positionsangaben auf dem Bildschirm, wie eben die x/y-Koordinate, werden am Computer immer von der linken oberen Ecke des Bildschirms aus berechnet (dort liegt der Punkt x=0, y=0)._

Wir können nun das Rechteck einfach durch Angabe der Attribute zeichnen:
```
<svg width="200" height="200">
<rect x="0" y="0" width="50" height="80"></rect>
</svg>
```

Dieser Code erzeugt ein schwarzes Rechteck in der linken oberen Ecke des Bildschirms, das 50 Pixel breit und 80 Pixel hoch ist. Es gibt jede Menge weiterer Attribute, die verwendet werden können, um das Rechteck zu verändern, der Blick in die [Dokumentation](https://developer.mozilla.org/en-US/docs/Web/SVG) lohnt sich.
