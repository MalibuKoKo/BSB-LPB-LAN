[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  
[Zurück zu Kapitel 7](kap07.md)  
    

# 8. URL-Befehle und Spezialfunktionen #

Da das Webinterface prinzipiell nur ‚aufgesetzt' ist, um eine Bedienung
ohne weitere Programme wie bspw. FHEM zu ermöglichen, ist ein direkter
Zugriff auf die einzelnen Funktionen und Parameter mittels anderer
Programme grundsätzlich möglich.

## 8.1 Auflistung und Beschreibung der URL-Befehle ##

-   **Alle Kategorien auflisten:**

    `http://<IP-Adresse>/K`
    Bei diesem Befehl kommuniziert der Adapter nicht mit dem
    Heizungssystem.  
    Es ist eine softwareseitige, interne Funktion.

-   **Alle enum-Werte für Parameter \<x\> auflisten:**

    `http://<IP-Adresse>/E<x>`  
    Bei diesem Befehl kommuniziert der Adapter nicht mit dem
    Heizungssystem.  
    Es ist eine softwareseitige, interne Funktion. Dieser Befehl ist nur
    für Parameter des Typs VT\_ENUM verfügbar.

-   **Alle Werte von Kategorie \<x\> abfragen:**

    `http://<IP-Adresse>/K<x>`

-   **Alle Werte von Parameter \<x\> abfragen:**

    `http://<IP-Adresse>/<x>`

-   **Alle Werte eines Parameterbereichs (von-bis) abfragen:**

    `http://<IP-Adresse>/<x>-<y>`

-   **Mehrere Abfragen können miteinander verkettet werden, z.B.:**

    `http://<IP-Adresse>/K11/8000/8003/8005/8300/8301/8730-8732/8820`

-   **Frage den Reset-Wert für Parameter \<x\> ab:**

    `http://<IP-Adresse>/R<x>`
    Im Display der integrierten Heizungssteuerung gibt es für einige
    Parameter eine Reset-Option. Ein Reset wird vorgenommen, indem das
    System nach dem Reset-Wert gefragt wird und dieser anschließend
    gesetzt wird.

-   **Setze Wert \<v\> (value) für den Parameter \<x\> mit optionalem
    Ziel \<z\>:**

    `http://<IP-Adresse>/S<x>=<v!z>`
    Die gewünschte Gerätezieladresse ist als \<z\> einzufügen, wenn
    \<!z\> nicht eingegeben wird, wird die Standardzieladresse
    verwendet.

    Um einen Parameter auf \'abgeschaltet/deaktiviert\' zu setzen, muss
    lediglich ein leerer Wert eingefügt werden:
    `http://<IP-Adresse>/S<x>=`
        
    ***Hinweis:***  
    *Voreingestellt ist nur der Lesezugriff erlaubt, ein
    Verändern von Einstellungen ist per default nicht möglich. Um dies
    zu ändern, ist Schreibzugriff für den entsprechenden Parameter zu
    gewähren. Siehe hierzu den entsprechenden Punkt in Kap. [5](kap05.md).*

    ***ACHTUNG:***  
    *Diese Funktion ist nicht ausgiebig getestet! Bitte sei
    vorsichtig mit dieser Funktion und nutze sie ausschließlich auf dein
    eigenes Risiko hin. Das Format des Wertes hängt von seinem Typ ab.
    Einige Parameter können abgeschaltet werden.*  

-   **Sende eine INF-Nachricht für den Parameter \<x\> mit dem Wert
    \<v\>:**

    `http://<IP-Adresse>/I<x>=<v>`
    Einige Werte können nicht direkt gesetzt werden. Das Heizungssystem
    wird mit einer TYPE\_INF-Nachricht informiert, bspw. bei der
    Raumtemperatur:  
    `http://<IP-Adresse>/I10000=19.5` → Raumtemperatur beträgt 19.5°C

-   **Bus-Typ (BSB, LPB oder PPS) vorübergehend ändern:**

    `http://<IP-Adresse>/P<x>`  
    Wechselt zwischen BSB (\<x\>=0), LPB (\<x\>=1) und PPS (\<x\>=2).  
    Nach einem Reset/Neustart des Arduino wird die Einstellung aus der
    Datei *BSB\_lan\_config.h* verwendet. Um den Bus-Typ dauerhaft
    festzulegen, sollte die Option setBusType config in der Datei
    *BSB\_lan\_config.h* entsprechend angepasst werden.

-   **Zusätzlich die eigene oder die Zieladresse mittels URL-Befehl
    wechseln:**

    Dazu muss der Befehl  
    `http://<IP-Adresse>/P<x,y,z>`
    genutzt werden, wobei  
    \<x\> = Bus (0 = BSB, 1 = LPB, 2 = PPS),  
    \<y\> = eigene Adresse (default 0x06 = RGT1) und  
    \<z\> = Zieladresse (default 0x00 = Geräteadresse 1) sind.  
    Leerwerte bei den Adressen belassen den bisherigen Wert (= Adresse).  
    ***ACHTUNG:*** *Diese Funktion wurde noch nicht ausgiebig getestet!*  

-   **Setze den Verbositäts-Level auf \<n\>:**

    `http://<IP-Adresse>/V<n>`
    Der voreingestellte Verbositäts-Level ist 1. Somit wird
    standardmäßig der Bus überwacht und alle Daten werden zusätzlich im
    Raw-Hex-Format dargestellt.  
    Soll der Modus deaktivert werden, so ist \<n\> auf 0 zu setzen
    (URL-Befehl: `/V0`).  
        
    Der Verbositäts-Level betrifft sowohl die serielle Konsole des
    Arduino als auch (optional) das Loggen der Bus-Daten auf die
    microSD-Karte, so dass die Speicherkarte u.U. sehr schnell voll
    wird! Im Fall des Loggens auf die interne microSD-Karte ist es daher
    empfehlenswert, den Verbostitätsmodus bereits in der Datei
    *BSB\_lan\_config.h* zu deaktivieren (byte verbose = 0).  
    Die html-Ausgabe bleibt mit /V1 unverändert.

-   **Bus-Monitor aktivieren:**

    `http://<IP-Adresse>/M<n>`
    Standardmäßig ist der Monitor-Modus deaktiviert (\<n\>=0).  
    Wenn \<n\> auf 1 gesetzt wird, werden alle Bytes auf dem Bus
    überwacht. Telegramme werden durch Umbruchzeichen als solche
    erkannt. Jedes Telegramm wird im Hex-Format auf der seriellen
    Konsole mit einem Zeitstempel in Millisekunden dargestellt.  
    Die Ausgabe der Überwachung betrifft nur die serielle Konsole des
    Arduino, die html-Ausgabe bleibt unverändert.  
    Zum Deaktivieren des Monitor-Modus ist \<n\> wieder auf 0 zu setzen
    (URL-Befehl: `/M0`).

-   **Setzen/Abfragen der GPIO-Pins (GPIO wird als OUTPUT genutzt):**

    `http://<IP-Adresse>/G<xx>[=<y>]`  
    Gibt den momentanen Status von GPIO Pin \<xx\> an, wobei \<y\>=0 LOW
    und \<y\>=1 HIGH ist. Kann ebenfalls benutzt werden, um den Pin auf
    \<y\>=0 (LOW) oder \<y\>=1 (HIGH) zu setzen (bspw. bei einem
    optional angeschlossenen Relaisboard).  
    Reservierte Pins, die nicht gesetzt werden dürfen, können in der
    *BSB\_lan\_config.h* unter dem Parameter GPIO\_exclude gesperrt
    werden.

-   **Abfragen der GPIO-Pins (GPIO wird als INPUT genutzt):**

    `http://<IP-Adresse>/G<xx>,I`
    Für die reine Abfrage eines externes Gerätes, das an einen GPIO
    angeschlossen ist (z.B. ein einfaches Koppelrelais), da die Pins per
    default auf ‚output' gesetzt sind. Der Pin bleibt nach diesem Befehl
    so lange auf ‚input', bis das nächste Mal mit `/G<xx>=<y>` ein
    Wert geschrieben wird - ab da ist er dann bis zum nächsten „I"
    wieder auf ‚output'.

-   **24h-Durchschnittswerte von ausgewählten Parametern anzeigen:**

    `http://<IP-Adresse>/A[=parameter1,...,parameter20]`  
    Zeigt rollierende 24h-Durchschnittswerte ausgewählter Parameter an.
    Die Initiale Festlegung dieser Parameter erfolgt in der Datei
    *BSB\_lan\_config.h* in der Variable `avg_parameters`.
    Während der Laufzeit kann `/A=[parameter1],...,[parameter20]` 
    verwendet werden, um (bis zu 20) neue Parameter zu definieren.

-   **Abfrage von DS18B20-Temperatursensoren:**

    `http://<IP-Adresse>/T`  
    Gibt die Temperaturwerte von optional angeschlossenen
    DS18B20-Sensoren aus.

-   **Abfrage von DHT22-Feuchtigkeits-/Temperatursensoren:**

    `http://<IP-Adresse>/H`  
    Gibt die Temperatur- & Feuchtigkeitswerte von optional
    angeschlossenen DHT22-Sensoren aus.

-   **Akkumulierte Brennerlaufzeit anzeigen:**

    `http://<IP-Adresse>/B`  
    Fragt sowohl die akkumulierte Brennerlaufzeit (in Sekunden) und die
    Brennerstarts/-takte als auch die Anzahl und die Dauer der Ladungen
    (in Sekunden) des Trinkwasserspeichers ab, die anhand von
    Broadcast-Nachrichten ermittelt wurden.

    `http://<IP-Adresse>/B0` setzt den Zähler zurück.

    Bei zweistufigen Ölbrennern wird zudem *eventuell* zwischen Stufe 1
    und 2 differenziert und die jeweiligen Starts und Laufzeiten werden
    angezeigt.

    ***Hinweis:***  
    *Diese Unterscheidung der beiden Brennerstufen findet aufgrund von
    spezifischen Broadcasts (BC) statt, also der vom Regler automatisch
    gesendeten Meldungen. Diese stufen-spezifischen BCs werden jedoch
    nicht von allen Reglern gesendet, die bei zweistufigen Ölbrennern
    verbaut wurden. Derzeit scheint es, als wenn die entsprechenden BCs
    nur bei dem Reglertyp RVS43.325 gesendet werden. Bei anderen Reglern
    werden die Brennerstarts und -laufzeiten kumuliert bei Stufe 1
    angezeigt (beinhalten also auch Stufe 2!).  
    Mittels Abfrage der Parameter 8330-8333 können die vom Regler
    gezählten Starts und Betriebsstunden der beiden Stufen nach wie vor
    abgerufen werden.*  
        
    Sollen die Brennerstarts und -laufzeiten von zweistufigen
    (Öl-)Brennern geloggt werden, beachte bitte den entsprechenden
    Hinweis in Kap. [5](kap05.md).*  

-   **Aktivieren/Deaktivieren des Loggens auf die microSD-Karte:**

    Prinzipiell erfolgt das Aktivieren/Deaktivieren der Log-Funktion
    durch das entsprechende Definement in der Datei *BSB\_lan\_config.h*
    vor dem Flashen. Während des Betriebes kann jedoch das Loggen
    deaktiviert werden, indem man folgende Parameter definiert:  
    `http://<IP-Adresse>/L=0,0`
    Zum Aktivieren werden dann wieder das Intervall und die gewünschten
    Parameter eingetragen. Bei einem Reset/Neustart des Arduino werden
    die Einstellungen aus der Datei *BSB\_lan\_config.h* verwendet --
    eine dauerhafte Umstellung der Logging-Parameter sollte also dort
    erfolgen.

    ***Hinweis:***  
    *Der per default aktivierte Verbositäts-Modus sollte im
    Fall eines dauerhaften Loggens auf die microSD-Karte in der
    Konfiguration deaktiviert werden (s.o.).*

-   **Konfiguration des Logfiles:**

    `http://<IP-Adresse>/L=<x>[,<parameter1>,<...>,<parameter20>]`  
    Setzt während der Laufzeit das Logging-Intervall auf \<x\> Sekunden
    und (optional) die Logging-Parameter auf \[parameter1\],
    \[parameter2\] etc.  
        
    Das Logging muss durch das Definement \#define LOGGING in der Datei
    *BSB\_lan\_config.h* aktiviert werden und kann initial anhand der
    Variablen log\_parameters und log\_interval konfiguriert werden
    (s.o.).

-   **Konfiguration des Loggens von Bus-Telegrammen:**

    `http://<IP-Adresse>/LU=<x>`
    Wenn Bus-Telegramme geloggt werden (Parameter 30000 als einzigen
    Parameter loggen), logge nur die unbekannten Command IDs (\<x\>=1)
    oder alle (\<x\>=0) Telegramme.

    `http://<IP-Adresse>/LB=<x>`
    Wenn Bus-Telegramme geloggt werden (Parameter 30000 als einzigen
    Parameter loggen), logge nur die Broadcasts (\<x\>=1) oder alle
    (\<x\>=0) Telegramme.

-   **Darstellung des Logfiles:**

    `http://<IP-Adresse>/D`
    Zeigt den Inhalt der Datei *datalog.txt*, die sich auf der
    microSD-Karte im Slot des Ethernet-Shields befindet.

    Mittels  
    `http://<IP-Adresse>/D0`
    kann die Datei *datalog.txt* zurückgesetzt werden, gleichzeitig wird
    eine korrekte CSV-Header-Datei generiert (*dieser Schritt wird zudem
    für die erste Benutzung empfohlen, bevor das Loggen gestartet wird*).

    Wer Parameter auf SD-Karte loggt, hat neben der reinen Textform auch
    die Möglichkeit, unter  
    `http://<IP-Adresse>/DG`  
    einen Graphen angezeigt zu bekommen.  
        
    ***Hinweis:***  
    *Für /DG muss bei Javascript-Blockern die Domain d3js.org freigegeben
    werden, da der Arduino weiterhin nur die CSV-Datei in den Browser
    lädt und diese dann mit dem D3-Framework grafisch aufbereitet wird.  
    Wird die Log-Datei via Webinterface mittels Klick auf „Anzeige
    Logdatei" aufgerufen, erfolgt standardmäßig zuerst die grafische
    Darstellung.*

-   **Resetten/Restarten des Arduino:**

    `http://<IP-Adresse>/N`
    Reset und anschließender Restart des Arduino nach einem Pausieren
    für ca. 15 Sekunden (Voraussetzung: `#define RESET` in
    *BSB\_lan\_config.h*, s.o.).  
      
    ***Hinweis:***    
    *Hierbei wird gleichzeitig auch noch das EEPROM des
    Arduino mit Nullen überschrieben. Dies hat z.Z. jedoch nur
    Auswirkung für PPS bzw. MAX!-Nutzer: Bei PPS werden damit die
    zwischengespeicherten Werte gelöscht, bei MAX! die registrierten
    Geräte (diese müssen sich dann durch einen Druck auf die Anlerntaste
    gegenüber BSB-LAN neu identifizieren).*

-   **Abfrage von MAX!-Thermostaten:**

    `http://<IP-Adresse>/X`  
    Gibt die Temperaturen von optional angeschlossenen MAX!-Thermostaten
    wieder. Diese sind zuvor in der Datei *BSB\_lan\_config.h* zu
    definieren.  
        
    ***Hinweis:***    
    *Bei MAX!-Geräten, die in BSB-LAN aufgenommen werden
    sollen, muss jeweils einmal die Anlern-Taste gedrückt werden (zu
    Erkennen an dem anschließenden 30-Sekunden-Countdown). Ein
    bestehendes Pairing zwischen den Geräten und einem Max!Cube bzw. CUL
    wird dabei nicht gestört und kann parallel betrieben werden (s. Kap. [12.2](kap12.md#122-max-komponenten)).*

-   **Ausgabe einer Parameterabfrage im JSON-Format:**

    Siehe hierzu den entsprechenden Punkt unter 8.2 Spezialfunktionen.

-   **Überprüfen von BSB-LAN auf nicht-freigegebene reglerspezifische
    CommandIDs:**

    Siehe hierzu den entsprechenden Punkt unter 8.2 Spezialfunktionen.

## 8.2 Spezialfunktionen ##

Der Übersichtlichkeit halber sind im Folgenden nochmals einige
ausgewählte Spezialfunktionen bzw. spezielle URL-Befehle gesondert
aufgeführt und beschrieben.

### 8.2.1 Raumtemperatur übermitteln ###

Mittels einer INF-Nachricht kann eine Raumtemperatur an den Regler
gesendet werden, um einen Raumeinfluss bei der Berechnung der
VL-Temperatur geltend zu machen.

Für die Raumtemperatur HK1[^20] ist der Spezialparameter 10000, für den
HK2[^21] der Parameter 10001 zu nutzen.

***Beispiel:***  
*Der URL-Befehl für den HK1, um eine Raumtemperatur von
19.5°C zu übermitteln, lautet: `http://<IP-Adresse>/I10000=19.5`*

***Hinweis:***  
*Um diese Funktion zu nutzen, muss die Funktion ‚Raumeinfluss' vorher im
Regler aktiviert und der Einflussfaktor prozentual festgelegt werden
(Parameter 750 für HK1, Parameter 1050 für HK2).  
Wird nur ein Temperaturwert als Einflussfaktor gemessen und übermittelt,
ist die Temperaturmessung in einem Führungs- / Referenzraum zu
empfehlen, in dem sich keinerlei weitere Wärmequelle (bspw. Kaminofen,
große Fenster in Südlage etc.) befindet.*

### 8.2.2 Präsenztaste simulieren ###

Die Funktion der Präsenztaste ist mit dem Spezialparameter 701 (für HK1)
und 1001 (für HK2) implementiert und als SET-Befehl auszuführen. Die
genannten Parameter müssen schreibbar sein (s. Kap. [5](kap05.md)).

Bei aktivem Automatikprogramm ist dabei `http://<IP-Adresse>/S701=1` für
den Wechsel auf ‚Betriebsart Reduziert' und `http://<IP-Adresse>/S701=0`
für den Wechsel auf ‚Betriebsart Komfort' zu setzen.  
Der jeweilige Wechsel ist bis zur nächsten Betriebsart-Umschaltung laut
Zeitprogramm gültig. Die Präsenztaste wirkt nur im Automatikbetrieb!

Es ist jedoch bisher noch nicht umfangreich getestet, ob diese Funktion
bei jedem Regler Wirkung zeigt oder nicht.

### 8.2.3 Manuellen TWW-Push ausführen ###

Bei einigen Reglern ist die (nahezu undokumentierte) Funktion eines
manuellen Trinkwasser-Pushs[^22] verfügbar. Um einen manuellen TWW-Push
auszulösen, muss dazu die TWW-Taste an der ISR-Bedieneinheit gedrückt
und für etwa drei Sekunden gehalten werden, bis im Display eine
entsprechende Meldung erscheint.

Bei einigen Reglern kann diese Funktion mittels eines SET-Befehls
erfolgen. Dieser lautet `http://<IP-Adresse>/S1601=1` - der
Spezialparameter 1601 muss dazu schreibbar sein (s. Kap. [5](kap05.md)).

### 8.2.4 Abrufen und Steuern mittels JSON ###

***Hinweis:***    
*Diese Funktion ist derzeit noch in der (Weiter-)Entwicklung,
es kann also noch Veränderungen hinsichtlich der Befehle und/oder
Funktionen geben!*

Parameterabfragen sowie das Setzen von Werten kann ebenfalls mittels
JSON erfolgen.

-   **Abfrage von Kategorien:**

    `http://<IP-Adresse>/JK=<xx>`
    Abfrage einer spezifischen Kategorie (\<xx\>=Kategorienummer)

    `http://<IP-Adresse>/JK=ALL`  
    Abfrage aller Kategorien (samt Min. und Max.)

-   **Abfragen und Setzen von Parametern per HTTP POST:**

    Hierbei ist der Aufruf der URL\
    `http://<IP-Adresse>/JQ` für eine Abfrage und  
    `http://<IP-Adresse>/JS` für das Setzen von Parametern zu verwenden.

    Folgende Parameter sind dabei möglich:
    
    ```
    http://<IP-Adresse>/JQ
    Senden: "Parameter"
    Empfangen: "Parameter", "Value", "Unit", "DataType" (0 = Zahl, 1 = ENUM, 2 = Wochentag, 3 = Stunde/Minute, 4 = Datum/Uhrzeit, 5 = Tag/Monat, 6 = String)  
    
    http://<IP-Adresse>/JS  
    Senden: "Parameter", "Value" (nur numerisch), "Type" (0 = INF, 1 = SET)  
    Empfangen: "Parameter", "Status" (0 = Fehler, 1 = OK, 2 = Parameter read-only)  
    ```

### 8.2.5 Überprüfen auf nicht-freigegebene reglerspezifische Command IDs ###

*Hinweis: Es ist empfehlenswert, diese Abfrage einmalig auszuführen.*

Diese Funktion ist nur bei aktiviertem Debug-Definement `#define DEBUG`
in der Datei *BSB\_lan\_config.h* verfügbar!

`http://<IP-Adresse>/Q`

Diese Funktion geht alle Command IDs durch, die in der Datei
*BSB\_lan\_defs.h* hinterlegt sind und schickt diejenigen, die nicht für
den eigenen Reglertyp hinterlegt sind, als Anfrage-Parameter (Typ QUR,
0x06) an den Regler.  
Das passiert bei Parametern, bei denen bisher nur eine Command ID
bekannt ist, ständig und erzeugt die bekannten „error 7 (parameter not
supported)"-Fehlermeldungen.

Sobald aber mehr als eine Command ID bekannt ist, bleibt die bisherige
Command ID i.d.R. auf \"DEV\_ALL\", ist also für alle Regler der
Standard, und die neue Command ID wird erst einmal nur für die Therme
freigeschaltet, die diese Command ID gemeldet hat.

Da es aber auch genauso gut umgekehrt sein kann, dass die \"neue\"
Command ID der Standard ist, und die \"alte\" Command ID der Sonderfall,
geht /Q nun die Command IDs durch, die nicht dem eigenen Regler
zugewiesen sind. Häufig können dort noch eine Reihe \"neuer\" Parameter
freigeschaltet werden.

***Zur Info:***  
*Es wird hierbei immer nur eine Anfrage mit einer Command ID an den
Regler geschickt!  
Der Regler beantwortet diese entweder mit einer Fehlermeldung (Typ ERR,
0x08) oder einer Antwort mit einem Datenpaket (Typ ANS, 0x07).  
In keinem Fall werden dabei Werte gesetzt oder Reglereinstellungen
verändert! Dafür müsste ein ganz anderer Telegramm-Typ gesetzt werden
(entweder Typ SET, 0x03 oder Typ INF, 0x02) - das macht /Q explizit
nicht!*  

Wenn bereits alle Parameter für den Reglertyp bekannt und freigegeben
sind, sieht die auf `http://<IP-Adresse>/Q`
folgende Webausgabe exemplarisch so aus:
    
```
Gerätefamilie: 90  
Gerätevariante: 100  
Start Test...  
Test Ende.  
```
    
Eine entsprechende Webausgabe bei bisher nicht-freigegebenen Parametern
für den spezifischen Regler hingegen sieht exemplarisch so aus:
    
```
Gerätefamilie: 90  
Gerätevariante: 100  
Start Test...  
2214  
DC 86 00 0B 06 3D 0D 08 EB E7 3A  
DC 80 06 0E 07 0D 3D 08 EB 00 0F 00 1B 94 5024  
5024 Trinkwasserspeicher - TWW Schaltdifferenz 1 ein: error 7 (parameter not supported)  
DC 86 00 0B 06 3D 31 07 1D C8 19  
DC 80 06 0E 07 31 3D 07 1D 00 01 40 A6 94 6031  
6031 Konfiguration - Relaisausgang QX22 Modul 1: error 7 (parameter not supported)  
DC 86 00 0B 06 3D 05 07 86 E3 AE  
DC 80 06 0D 07 05 3D 07 86 00 00 2C 55 8314  
8314 Diagnose Erzeuger - Kesselrücklauftemperatur Ist: error 7 (parameter not supported)  
DC 86 00 0B 06 3D 11 05 1A 58 5A  
DC 80 06 0E 07 11 3D 05 1A 01 00 00 91 09  
Test Ende.  
```  
    
In diesem Fall sollte die Webausgabe bitte kopiert und gemeldet werden,
damit eine entsprechende Anpassung vorgenommen werden kann.  
    
     
     
[Weiter zu Kapitel 9](kap09.md)      
[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  

