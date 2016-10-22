#Ergebnis-Landkarte mit Javascript

Das folgende Tutorial versucht eine Schritt-für-Schritt-Einführung in die Visualisierung von Wahlergebnissen mittels Javascript/d3.js zu geben. Ziel ist, eine [Ergebnis-Landkarte](http://bl.ocks.org/ginseng666/1b950542b04bdf91d55a846b633bb77b) zu erstellen. Dazu werden Daten als `json` und `csv` geladen, ausgewertet, verknüpft und dargestellt. Als Grundlage wird das Ergebnis des ersten Wahlgangs der Bundespräsidentenwahl 2016 verwendet.

Das Tutorial baut auf dem Beispiel zum [Balken-Diagramm](tutorial balkendiagramm.md) auf und setzt die dort behandelten Punkte voraus. Der Code verwendet Version 4 von d3.js. 

Als Vorwarnung: Einige der Befehle mögen kryptisch wirken, man versteht sie vielleicht erst im Lauf der Zeit. Sie sind aber so formuliert, dass man sie (zumeist) unverändert übernehmen kann.


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

Wir definieren zunächst eine Variable `projection` mit der in d3.js vorgegebenen Mercator-Projektion. Danach erzeugen wir eine Variable `gemeinden`, in der die `geoPath()`-Funktion zusammen mit der Projektion gespeichert wird. Abschließend stellen wir eine Art Zoomstufe mit `scale` ein, vorerst auf 1, und legen fest, dass die Karte vorerst nicht verschoben wird (`translate`). Die letzten beiden Punkte setzen die Projektion quasi auf 0, bevor sie unten dann an die jeweilige Karte angepasst wird.

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
Nachdem wir die Kartendatei geladen haben, müssen wir sie richtig positionieren und richtig skalieren. Erscheint eine Landkarte trotz fehlerfreiem Code manchmal nicht am Bildschirm, dann liegt der Fehler oft bei diesen Punkten. 

Die Skalierung erfolgt im Verhältnis zu den Variablen `width` und `height`, damit die Darstellung an den verfügbaren Platz angepasst ist. Um diese Werte abzustimmen, brauchen wir zunächst die Abmessungen der Karte selbst:
```javascript
var b = gemeinden.bounds(map);	
var box = d3.geoBounds(map);	
```

Der Befehl `gemeinden.bounds(map)` gibt uns die Eckpunkte eines Rechtecks, das um die Karte gelegt wird. `d3.geoBounds(map)` macht dasselbe für die Geo-Koordinaten.

Nun berechnen wir die Skalierung und verändern die Projektion entsprechend:
```javascript
var s = .95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height);
projection.scale(s).center([(box[0][0]+box[1][0])/2,(box[0][1]+box[1][1])/2]).translate([width / 2, height / 2]);
```

Auch dieser Teil mag zunächst überfordernd wirken, im Kern berechnet man aber nur die maximal mögliche Vergrößerung der Karte angesichts ihrer Abmessungen und des verfügbaren Platzes (Ausgangspunkt für `scale` ist immer 1). Dann sagt man der Projektion, wie stark sie die Karte vergrößern/verkleinern soll, zentriert sie horizontal und vertikal und verschiebt die Darstellung noch in die Mitte des Bildschirms.

Nach diesen eher komplexen Vorarbeiten ist die Darstellung der Karte vergleichsweise simpel. Wir verwenden dazu das [path](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths)-Element, laden die Karte mit `data(map.features)` und zeichnen für jeden Eintrag in diesem Array - für jede Gemeinde - die Grenzen in schwarz (`stroke`). Die Flächen bleiben leer (`fill`). Einmal mehr der Hinweis: Wenn man sich Variable in der Console anschaut, dann hilft das, die Abläufe zu verstehen.

Hier der komplette Block, vom Laden der Datei bis zur Darstellung:

```javascript
d3.json("gemeinden.json", function(grenzen)
  {
  var map = topojson.feature(grenzen, grenzen.objects.gemeinden);

  var b = gemeinden.bounds(map);	
  var box = d3.geoBounds(map);	
  var s = .95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height);
  projection.scale(s).center([(box[0][0]+box[1][0])/2,(box[0][1]+box[1][1])/2]).translate([width / 2, height / 2]);
	
  svg.selectAll("path")
    .data(map.features)
    .enter()
    .append("path")
    .attr("d", gemeinden)
    .style("stroke", "black")
    .style("fill", "none");
  });
```

Wie angesprochen steht der gesamte Code innerhalb der `{}` des `d3.json`, damit er erst ausgeführt wird, wenn die Datei fertig geladen wurde.

![Screenshot Karte 1](https://github.com/ginseng666/graz_19102016/blob/master/img/karte_1.jpg)


###Ergebnis-csv laden und bearbeiten
Damit ist die erste Hälfte des Projekts abgeschlossen, wir haben eine Karte. Jetzt fehlen nur noch die Ergebnisse. Anders als beim Balken-Diagramm haben wir nicht sechs Werte, sondern sechs Werte für alle Gemeinden in Österreich. Daraus händisch eine Variable zu schaffen sollte auf der ToDo-Liste nicht ganz oben stehen.

Wir müssen das auch nicht machen, wir können die [Ergebnis-Datei](http://www.bmi.gv.at/cms/BMI_wahlen/bundespraes/bpw_2016/FILES/Endgueltiges_Gesamtergebnis_BPW16_1WG.xlsx) des BMI verwenden. Das ist eine xlsx-Datei, die alle relevanten Felder enthält. 

![Screenshot Beispiel BMI](https://github.com/ginseng666/graz_19102016/blob/master/img/beispiel_bmi.jpg)

Bevor wir sie zu einer csv-Datei weiterverarbeiten entwirren wir die verbundenen Zellen in den ersten beiden Zeilen, beschriften die Spalten und löschen die Prozenteinträge. Ebenso entfernen wir alle Tausender-Trennzeichen.

![Screenshot Beispiel BMI2](https://github.com/ginseng666/graz_19102016/blob/master/img/beispiel_bmi2.jpg)

Jetzt können wir die Datei speichern. Um aus dem deutschen Excel eine gut verwendbare csv-Datei zu bekommen, muss man ein paar Umwege gehen:

* Wir speichern die Excel-Datei mit "Speichern als" als "Unicode Text"
* Diese Datei öffnen wir mit einem Text-Editor
* Mittels Suchen/Ersetzen alle "," durch "." ersetzen (ersetzt alle Komma-Zeichen)
* Das Trennzeichen zwischen den Spalten - wahrscheinlich Tabulator - kopieren wir und via Suchen/Ersetzen ersetzen wir es mit einem ","
* Die Datei speichern
* Die Dateiendung auf ".csv" ändern


Eine fertige Datei kann [hier](gemeindeergebnisse.csv) heruntergeladen werden.

Nach dieser Tour de force ist das Laden der Datei in Javascript simpel:

```javascript
d3.csv("gemeindeergebnisse.csv", function(ergebnisse)
  {
  console.log(ergebnisse);
  });
```

In der Console sehen wir nun einen Array von Objects, der für jede Zeile im Excel einen Eintrag enthält. Vom Aufbau her ist er identisch mit der Ergebnis-Variablen im Balken-Diagramm-Beispiel. Was uns auch auffällt: Alle Zahlen stehen unter "". Das ist ein Nachteil des Arbeitens mit CSV, alle Werte werden standardmäßig als String geladen.

Damit wir mit den Zahlen weiterarbeiten können, müssen wir sie wieder in Zahlen umwandeln. Dafür verwenden wir zwei Schleifen:
```javascript
d3.csv("gemeindeergebnisse.csv", function(ergebnisse)
  {
  for (var i = 0; i < ergebnisse.length; i++)
    {
    for (var x in ergebnisse[i])
      {
      if (x != "GKZ" && x != "Gebietsname") ergebnisse[i][x] = +ergebnisse[i][x];
      }
    }
  console.log(ergebnisse);
});
```

Die erste for-Schleife kennen wir schon, wir durchlaufen einfach jeden Eintrag im Array. Jeder Eintrag wird dann einer zweiten Schleife unterworfen: Mit `for (var x in ergebnisse[i])` durchlaufen wir alle Eigenschaften des Objects `ergebnisse[i]`. Der Name der jeweiligen Eigenschaft wird in der Variablen `x` gespeichert.

Dann folgt ein Ausschlussverfahren: Die Eigenschaften `GKZ` und `Gebietsname` sind Strings, müssen also nicht geändert werden. Alle anderen Eigenschaften werden aber in eine Zahl umgewandelt - das passiert schlicht mit dem `+`-Zeichen.


###Karte und Ergebnis verknüpfen
Soweit gerüstet geht es nun nur mehr darum, die Karte und die Ergebnisse zu verknüpfen. Dafür brauchen wir einen Wert, der in beiden Datensätzen enthalten ist, und das ist die Gemeindekennziffer.
