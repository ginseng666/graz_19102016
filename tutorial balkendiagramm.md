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

Dieser Code umreißt die Struktur der Seite: Neben dem `<html>`-Tag am Anfang steht im `<head>` ein Titel und der Zeichensatz wird angegeben (`<meta...>`). Danach wird die d3.js-library eingebunden, damit später die entsprechenden Befehle verfügbar sind. Das `</head>`-Tag wird geschlossen, der `<body>` beginnt und enthält noch einen leeren `<script>`-Block. Hier wird der Javascript-Code eingefügt. Anschließend werden der `</body>` und das `</html>` wieder geschlossen.

Diesen Teil kann man standardmäßig als Ausgangsbasis für eine Seite verwenden.

Visualisierungen mit d3.js zeichnen so genannte SVGs

...rect, definiert sich über die breite und höhe
