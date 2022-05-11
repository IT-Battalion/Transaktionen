# JDBC Transaktions

## Aufgabe

1. **Generieren einer id fuer neue Bestellungen** Beim Anlegen einer Bestellungen wird zunaechst die hoechste vergebene Order-Id gesucht und dann eine neue Bestellung entsprechend erhoeht eingefuegt. Demonstriere, wieso das zu Problemen fuehren kann. Fasse danach diese beiden Operationen in eine Transaktion zusammen. Laesst sich das Problem durch Setzen eines Isolation Levels loesen? Beseitige das Problem nun endgueltig, indem du vor Auslesen der maximalen ID die Tabelle fuer alle anderen Teilnehmer lockst. Welche Art von Lock ist  hier am sinnvollsten?
2. **Atomizitaet bei Bestellungen** Tritt waehrend dem Speichern der einzelnen Positionen einer Bestellung ein  Fehler auf, so koennen unvollstaendige Bestellungen im System verbleiben oder auch Warenstaende faelschlich verringert werden, was sicher  ungewuenscht ist. Demonstriere und dokumentiere dieses Verhalten. Fuehre diese Schritte daher nun in einer Transaktion durch, sodass  sichergestellt ist, dass eine Bestellung ganz oder gar nicht erfasst  wird. Nachdem das Anlegen in der Tabelle `orders` selbst nach Aufgabe 1 eine eigene Transaktion ist, wird dir im Fehlerfall eine  "leere" Bestellung uebrigbleiben - dies stellt hier kein Problem dar. (*Zusatzfrage: Wie muesste man die Anwendung bzw. die Datenbank aendern, um auch das  Anlegen leerer Bestellungen zu vermeiden ohne gleichzeitig die  Performance stark zu beeintraechtigen?*)
3. **Gleichzeitiges Abschicken zweier Bestellungen** Werden zwei Bestellungen fuer den gleichen Artikel gleichzeitig abgeschickt,  koennen mehr Artikel bestellt werden, als vorhanden (und der Lagerstand  dabei ins Negative gehen). Demonstriere dieses Verhalten und verwende  danach sinnvolle Locks auf Zeilenebene um solche Fehler zu Verhindern.
4. **Anzeige von Statistiken** Der Aufruf von http://127.0.0.1:8000/stats liefert Statistiken ueber Bestellungen nach Laendern aufgeschluesselt. Wird  waehrend dem Erstellen der Statistik eine Bestellung abgeschickt, so  kann diese Statistik inkonsistent werden (zB. in der Uebersicht weniger  Bestellungen anfuehren, als spaeter in der Detailsansicht). Demonstriere dieses Verhalten stelle danach sicher, dass solche Phaenomene  ausgeschlossen werden koennen. Um die Performance nicht zu  beeintraechtigen soll deine Loesung aber keine Locks verwenden, sondern  durch Setzen eines entsprechenden Isolation Levels realisiert werden.

## Lösung

### Aufgabe 1:

Sollte das erstellen einer Order länger dauern, und während dessen eine neue Bestellung erstellt werden. Versucht die API, zwei Orders mit der selben ID zu erstellen. Da jede ID jedoch nur einmal existieren darf, führt dies zu Problemen.

Um das Problem zu begheben wurden Transactions implementiert und ein exclusive lock vor dem finden der ID gesetzt.

Getestet habe ich das dann mittels Postman. Ich habe zwei fast idente Anfragen zum erstellen eines Produktes gesendet. Aufgrund des eingebauten Delays konnte man das ganz einfach machen. Als Resultat habe ich zwei Orders mit unterschiedlichen IDs bekommen.

![Bildschirmfoto 2022-03-23 um 13.10.19](/Users/pelias/Desktop/Bildschirmfoto 2022-03-23 um 13.10.19.png)

![Bildschirmfoto 2022-03-23 um 13.10.27](/Users/pelias/Desktop/Bildschirmfoto 2022-03-23 um 13.10.27.png)

### Aufgabe 2:

Hier wurden beide Transactions zu einer zusammengefasst, dass habe ich aber von hausaus bei der ersten Aufgabe gemacht.

Getestet habe ich dass dann so:

![Bildschirmfoto 2022-03-23 um 13.18.04](/Users/pelias/Desktop/Bildschirmfoto 2022-03-23 um 13.18.04.png)

### Aufgabe 3:

Hier wurde eine einzelne Zeile gelockt um zu verhindern, dass eine zweite Query auf diese zugreifen kann.

Getestet habe ich, dass dann mit zwei verschiedenen Requests die zusammen mehr Produkte bestellen würden als vorhanden.

![Bildschirmfoto 2022-03-23 um 14.07.02](/Users/pelias/Desktop/Bildschirmfoto 2022-03-23 um 14.07.02.png)

![Bildschirmfoto 2022-03-23 um 14.07.10](/Users/pelias/Desktop/Bildschirmfoto 2022-03-23 um 14.07.10.png)

### Aufgabe 4:

Hier wurde das Isolation Level auf TRANSACTION_READ_COMMITTED gesetzt. Somit wird das Problem der Inkonsistenz behoben.

![Bildschirmfoto 2022-03-23 um 14.10.32](/Users/pelias/Desktop/Bildschirmfoto 2022-03-23 um 14.10.32.png)

## Zusatzfragen

### *Wie muesste man die Anwendung  bzw. die Datenbank aendern, um auch das Anlegen leerer Bestellungen zu vermeiden ohne gleichzeitig die Performance stark zu beeintraechtigen?*

Ich würde die Columns als NOT NULL columns defenieren, da es so zu einer Exception kommt, sobald die Werte null sind.

### *Wie wuerdest du in einem Webservice einen API Endpoint fuer Bestellungen besser realisieren?*

Ich würde jedenfalls mehrere Requests machen und nicht alles in eine schreiben. Außerdem würde ich die ID nicht manuell aus der Datenbank auslesen sondern automatisiert erstellen lassen.