[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  
[Zurück zu Anhang A2](anhang_a2.md)  
    

# Anhang B: Cheatsheet URL-Befehle #

| URL-Befehl            | Auswirkung                                                                    |
|:----------------------|:------------------------------------------------------------------------------|
|  /\<x\>               | Wert/Einstellung von Parameter \<x\> anzeigen
|  /\<x\>,\<y\>,\<z\>   | Werte/Einstellungen der Parameter \<x\>, \<y\> und \<z\> anzeigen  
|  /\<x\>-\<y\>         | Werte/Einstellungen der Parameter \<x\> bis \<y\> anzeigen  
|  /A                   | Anzeigen der 24h-Durchschnittswerte  
|  /A=\<x\>,\<y\>       | Ändern der 24h-Durchschnittswertberechnung in Parameter \<x\>, \<y\>  
|  /B                   | Anzeige akkumulierter Brennerlaufzeiten (in Sek.) & -takte (plus TWW)  
|  /B0                  | Zurücksetzen des Zählers Brennerlaufzeiten & -takte  
|  /C                   | Anzeige der Konfiguration von BSB-LAN  
|  /D                   | Anzeige der Logdatei der microSD-Karte  
|  /DG                  | Grafische Anzeige der Logdatei der microSD-Karte  
|  /D0                  | Zurücksetzen der Logdatei & neue Generierung des Headers  
|  /E\<x\>              | ENUM-Werte für Parameter \<x\> anzeigen  
|  /G\<x\>              | GPIO: Abfragen des Pins \<x\>  
|  /G\<x\>,\<y\>        | GPIO: Setzen des Pins \<x\> auf high (\<y\> = 1) oder low (\<y\> = 0)  
|  /G\<x\>,I            | GPIO: Abfragen des Pins \<x\> mit gleichzeitigem Setzen auf INPUT  
|  /H                   | Abfrage optional angeschlossener DHT22-Sensoren  
|  /I\<x\>=\<y\>        | INF-Nachricht an Parameter \<x\> mit Wert \<y\> senden  
|  /K                   | Alle Regler-Kategorien auflisten  
|  /K\<x\>              | Alle Parameter und Werte von Regler-Kategorie \<x\> abfragen  
|  /L=0,0               | Loggen auf microSD-Karte deaktivieren  
|  /L=\<x\>,\<y\>       | Log-Intervall auf \<x\> Sekunden setzen, mit (optional) Log-Parameter \<y\>  
|  /LB=\<x\>            | Loggen von Bus-Telegrammen: Nur Broadcasts (\<x\>=1) oder alle (\<x\>=0)  
|  /LU=\<x\>            | Loggen von Bus-Telegrammen: Nur unbekannte (\<x\>=1) oder alle (\<x\>=0)  
|  /M\<x\>              | Monitor-Modus aktivieren (\<x\> = 1) oder deaktivieren (\<x\> = 0)  
|  /N                   | Reset und Neustart des Arduino (Dauer ca. 15Sek)  
|  /O                   | Übersicht der URL-Befehle  
|  /P\<x\>              | Busprotokoll / Bustyp setzen: \<x\> = 0 → BSB \| 1 → LPB \| 2 → PPS  
|  /P\<x\>,\<s\>,\<d\>  | Busprotokoll/-typ \<x\>, eigene Adresse \<s\>, Zieladresse \<d\> setzen  
|  /Q                   | Test auf nicht-freigegebene reglerspezifische Parameter  
|  /R\<x\>              | Abfrage des Reset-Werts für Parameter \<x\>  
|  /S\<x\>=\<y\>        | Wert \<y\> für Parameter \<x\> setzen  
|  /T                   | Abfrage optional angeschlossener DS18B20-Sensoren  
|  /V\<x\>              | Verbositäts-Modus aktivieren (\<x\> = 1) oder deaktivieren (\<x\> = 0)  
|  /X                   | Abfrage optional eingebundener MAX!-Thermostate  

     
     
[Weiter zu Anhang C](anhang_c.md)      
[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  

