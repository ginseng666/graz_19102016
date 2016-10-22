#Ergebnis-Landkarte mit Javascript

Das folgende Tutorial versucht eine Schritt-für-Schritt-Einführung in die Visualisierung von Wahlergebnissen mittels Javascript/d3.js zu geben. Ziel ist, eine [Ergebnis-Landkarte](http://bl.ocks.org/ginseng666/1b950542b04bdf91d55a846b633bb77b) zu erstellen. Dazu werden Daten als `json` und `csv` geladen, ausgewertet, verknüpft und dargestellt. Als Grundlage wird das Ergebnis des ersten Wahlgangs der Bundespräsidentenwahl 2016 verwendet.

Das Tutorial baut auf dem Beispiel zum [Balken-Diagramm](tutorial balkendiagramm.md) auf und setzt die dort behandelten Punkte voraus. Der Code verwendet Version 4 von d3.js.


###zu Landkarten generell
Ergebnis-Landkarten zu Wahlen gehören wohl zum Standard-Repertoire der politischen Berichterstattung. Von der Darstellung der [Sieger](http://driven-by-data.net/2016/06/23/brexit-map.html) bis zu [Heat-Maps](http://orf.at/wahl/bp16/#ersterwahlgang/globus/hun) der Stimmenverteilung sind verschiedene Formen möglich und üblich. 

Um eine Landkarte zu erzeugen benötigt man zunächst eine Datei mit den geometrischen Formen der Gemeinden, Bezirke oder Länder. Es gibt unterschiedliche Formate, in d3.js verwendet man üblicherweise das `topojson`-Format, das eine vergleichsweise kleine Dateigröße erlaubt. Der grundsätzliche Weg zu einer Karte ist [hier](https://bost.ocks.org/mike/map/) erläutert.

Geographische Grenzen für Österreich gibt es u.a. hier:
* [Gemeindegrenzen als shp](https://www.data.gv.at/katalog/dataset/c33d36b0-f184-4f2a-89cc-839ca7fcf88a)
* [Gemeindegrenzen als geojson/topojson](https://github.com/ginseng666/GeoJSON-TopoJSON-Austria)


Im politischen Kontext ist zu bedenken, dass die geographische Fläche einer Einheit nicht gleichbedeutend mit ihrer Bedeutung ist - oder anders ausgedrückt, die meisten Wahlberechtigten wohnen in Österreich in geographisch kleinräumigen Städten, wie das Projekt [austromorph](https://austromorph.space/) schön demonstriert.


###Das Grundgerüst
Wir können wieder das gleiche Grundgerüst wie beim Balken-Diagramm verwenden und auch gleich ein SVG erzeugen. Achtung: Neben d3.js müssen wir nun auch topojson.js als library laden:
```javascript
<!DOCTYPE html>
<head>
<title>ein Balkendiagramm</title>
<meta charset="utf-8">
<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/topojson/1.6.20/topojson.js" charset="utf-8"></script>
</head>
<body>

<script>
var width = window.innerWidth * 0.95;
var height = window.innerHeight * 0.95;

var svg = d3.select("body").append("svg")
	.attr("width", width)
	.attr("height", height);
</script>

</body>
</html>
```


Zusätzlich bauen wir einen Array mit den Nachnamen der KandidatInnen ein, sowie ein Object mit Farben zu den einzelnen Personen:
```javascript
var kandidaten = ["Griss", "Hofer", "Hundstorfer", "Khol", "Lugner", "Van der Bellen"];
var farben = {"Griss":"#8E88A7", "Hofer":"#2657A8", "Hundstorfer":"#C83D44",
"Khol":"#191919", "Lugner":"#E7B500", "Van der Bellen":"#89A04F"};
```


###Vorbereitungen für die Karte
Wie erwähnt benötigt man zur Visualisierung einer Landkarte eine Datei mit geographischen Grenzen. Diese sind oft (müssen aber nicht) als Koordinaten bzw. in Länge/Breite angegeben, d.h. man muss die Angaben in Grad und Minuten in Bildschirm-Koordinaten umwandeln. Dafür braucht man eine Projektion, z.B. Mercator. Man kann im Übrigen nicht eine beliebige Projektion verwenden, sondern muss sich an die vorgegebene Version halten, da sonst die Karte verzerrt oder gar nicht dargestellt wird.

Im Code sieht dieser Schritt so aus:
```javascript
var projection = d3.geoMercator();
var gemeinden = d3.geoPath().projection(projection);
projection.scale(1).translate([0, 0]);
```

Wir definieren zunächst eine Variable `projection` mit der in d3.js vorgegebenen Mercator-Projektion. Danach erzeugen wir eine Variable `gemeinden`, in der die `geoPath()`-Funktion zusammen mit der Projektion gespeichert wird. Abschließend stellen wir eine Art Zoomstufe mit `scale` ein, vorerst auf 1, und legen fest, dass die Karte vorerst nicht verschoben wird (`translate`). Der letzte Punkt wird weiter unten noch wichtig.

Auf den ersten Blick mag das relativ unverständlich wirken. Dieser Ablauf ist für viele Kartendarstellungen identisch, man kann ihn daher einfach auch kopieren und verwenden. Das Verständnis stellt sich meistens im Lauf der Zeit ein.

Als nächstes laden wir die Kartendatei ([hier](gemeinden.json)), die als `json` vorliegt, mit dem entsprechenden Befehl und speichern sie in die Variable `map`:
```javascript
d3.json("gemeinden.json", function(grenzen)
	{
	var map = topojson.feature(grenzen, grenzen.objects.gemeinden);
  });
```

Der Befehl `d3.json()` lädt die angegebene Datei, anschließend wird sie in der Funktion mit dem im Klammer angegebenen Namen weiterverarbeitet. Die Code-Zeile 
`var map = topojson.feature(grenzen, grenzen.objects.gemeinden);`
liest das topojson-Format unserer Datei ein und speichert sie als Object in die Variable `map`. Ergänzt man darunter noch `console.log(map);`, so kann man sich das Ergebnis dieser Operation genauer anschauen.

Ein wichtiger Hinweis: Das Laden von Dateien in dieser Form geschieht in d3.js [asynchron](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Synchronous_and_Asynchronous_Requests), das heißt, dass während dem Ladevorgang das Programm "außerhalb" dieses Befehls weiterläuft. Das führt zu Problemen und Fehlern, wenn man dann bereits auf die Daten zugreifen will.

Eine Lösung ist, alle weiteren Befehle innerhalb der geschwungenen Klammern (also vor dem `});`) auszuführen, was aber bei längerem Code unübersichtlich wird. Besser ist es daher, am Ende des Ladevorgangs eine Funktion mit dem weiteren Programm aufzurufen.


###Landkarte positionieren, skalieren und zeichnen
