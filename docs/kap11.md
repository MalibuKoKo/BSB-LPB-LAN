[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  
[Zurück zu Kapitel 10](kap10.md)  
    

# 11. Nutzung von externen Programmen #

Da der Adapter lediglich eine Schnittstelle darstellt, mittels derer man
Zugriff auf den Heizungsregler via Computer erhält, können
selbstverständlich externe Programme zum Einsatz kommen. Somit kann die
Heizungssteuerung in komplexe Heimautomationssysteme eingebunden werden.
Auch das Erstellen von umfassenden Logdateien und deren grafische
Aufbereitung kann auf diese Weise erfolgen.

Da es hierfür verschiedene Softwarelösungen gibt, kann hier weder eine
umfangreiche Vorstellung der verschiedenen Systeme erfolgen, noch eine
grundsätzliche Anleitung zur Integration des Adapters oder einer
Heizungsregelung per se gegeben werden. Sollte jedoch bereits eines der
nachfolgend erwähnten Systeme zum Einsatz kommen, so sind die folgenden
Beispielkonfigurationen hoffentlich eine kleine Starthilfe, um die
eigenen Vorstellungen umzusetzen.

Sollten Fragen bzgl. der folgenden Beispiele oder anderer
Konfigurationsmöglichkeiten auftreten, so ist bitte von diesbezüglichen
Anfragen an mich abzusehen - die Fragen sollten mittels
entsprechender Literatur oder in spezifischen Foren geklärt werden.

***Hinweis:***  
*Selbstverständlich müssen stets sowohl die IP als auch -falls
aktiviert- die optionalen Sicherheitsfunktionen bei den folgenden
Beispielen entsprechend angepasst werden. Ebenso müssen Parameter, die
gesetzt werden sollen, schreibbar sein (s. Kap. [5](kap05.md)).*

*Solltest du ein anderes System als die im Folgenden aufgeführten
verwenden, so würde ich mich über die Zusendung eines Beispielscripts
zur Anbindung des Adapters freuen! Sende es mir dazu einfach als
.txt-Datei an `adapter [ät] quantentunnel.de` - danke!*

## 11.1 FHEM ##

### 11.1.1 Einbindung mittels BSB-LAN-Modul ###

*Derzeit wird vom FHEM-Forumsmitglied „justme1968" ein BSB-LAN-Modul für
FHEM entwickelt:
[https://forum.fhem.de/index.php/topic,84381.0.html](https://forum.fhem.de/index.php/topic,84381.0.html)  
Vielen Dank!*

*UPDATE:  
Eine erste (Test-)Version ist bereits verfügbar, ein stabiler
und problemloser Betrieb kann aber noch nicht garantiert werden!*

### 11.1.2 Einbindung mittels HTTPMOD-Modul ###

***Die FHEM-Beispielscripte stammen vom FHEM-Forumsmitglied „freetz".  
Vielen Dank!***

Um auf die Webschnittstelle des Adapters zuzugreifen, kann das Modul
HTTPMOD in FHEM genutzt werden.

***Beispielscript für eine Parameterabfrage und die Übermittlung einer
Raumtemperatur:***

Der Beispielcode fragt die Parameter 8700, 8743 und 8314 alle 300
Sekunden ab und weist diese dem Gerät \"THISION\" (Name des
Heizungssystems) und den Readings \"Aussentemperatur\",
\"Vorlauftemperatur\" und \"Ruecklauftemperatur\" zu.

Darüber hinaus stellt es ein Reading \"Istwert\" bereit, das per FHEM
gesetzt werden kann, um dem Heizungssystem die aktuelle Zimmertemperatur
mitzuteilen (Parameter 10000).

Zu guter Letzt berechnet es die Differenz zwischen \"Vorlauftemperatur\"
und \"Rücklauftemperatur\" und weist diese Differenz dem Reading
\"Spreizung\" zu.

***Bitte beachte:***  
*Die Regex-Bedingungen müssen vom Beginn des Strings an
(also der Parameternummer wie bspw. 8700) matchen und nicht erst ab
einem späteren Teil des Strings.*  
    
```
define THISION HTTPMOD http://192.168.178.88/8700/8743/8314 300
attr THISION userattr reading0Name reading0Regex reading1Name reading1Regex reading2Name reading2Regex readingOExpr set0Name set0URL
attr THISION event-on-change-reading .*
attr THISION reading0Name Aussentemperatur
attr THISION reading0Regex 8700 .*:[ \t]+([-]?[\d\.]+)
attr THISION reading1Name Vorlauftemperatur
attr THISION reading1Regex 8743 .*:[ \t]+([-]?[\d\.]+)
attr THISION reading2Name Ruecklauftemperatur
attr THISION reading2Regex 8314 .*:[ \t]+([-]?[\d\.]+)
attr THISION readingOExpr $val=~s/[\r\n]//g;;$val
attr THISION set0Name Istwert
attr THISION set0URL http://192.168.178.88/I10000=$val
attr THISION timeout 5
attr THISION userReadings Spreizung { sprintf("%.1f",ReadingsVal("THISION","Vorlauftemperatur",0)-ReadingsVal("THISION","Ruecklauftemperatur",0));; }
```  
    
    
***Beispielscript für die Abfrage und Ansteuerung eines Relaisboards:***

Das Folgende ist ein Beispiel für eine FHEM-Konfiguration, bei dem die
drei Relais-Ports namens \"Heater\", \"Fan\" und \"Bell\" abgefragt und
gesteuert werden, die an die entsprechenden GPIO-Pins 7, 6 und 5
angeschlossen sind.  
    
```
define EthRelais HTTPMOD http://192.168.178.88/G05/G06/G07 30
attr EthRelais userattr reading0Name reading0Regex reading1Name reading1Regex reading2Name reading2Regex readingOExpr readingOMap set0Name set0URL set1Name set1URL set2Name set2URL setIMap setParseResponse:0,1 setRegex
attr EthRelais event-on-change-reading .*
attr EthRelais reading0Name Heater
attr EthRelais reading0Regex GPIO7:[ \t](\d)
attr EthRelais reading1Name Fan
attr EthRelais reading1Regex GPIO6:[ \t](\d)
attr EthRelais reading2Name Bell
attr EthRelais reading2Regex GPIO5:[ \t](\d)
attr EthRelais room Heizung
attr EthRelais set0Name Heater
attr EthRelais set0URL http://192.168.178.88/G07=$val
attr EthRelais set1Name Fan
attr EthRelais set1URL http://192.168.178.88/G06=$val
attr EthRelais set2Name Bell
attr EthRelais set2URL http://192.168.178.88/G05=$val
attr EthRelais setParseResponse 1
attr EthRelais setRegex GPIO[0-9]+:[ \t](\d)
attr EthRelais timeout 5
```
    

## 11.2 openHAB ##

***Die openHAB-Beispielscripte stammen vom FHEM-Forumsmitglied
„acfischer42".  
Vielen Dank!***

Es existiert derzeit kein komplettes Binding fuer BSB-LAN. Mit dem HTTP
Binding[^25] und der Javascript Transformation[^26] ist es allerdings
durchaus möglich, Werte auszulesen und auch zu schreiben.

Ein Loggen der Daten kann bspw. mit InfluxDB erfolgen, die
Visualisierung mit Grafana.  
Selbstverständlich sind die notwendigen Addons wie bspw. die Javascript
Transformation vorhergehend zu installieren.

***Beispiel einer Item-Konfiguration:***  
    
```
Number hz_aussentemp "Aussentemperatur [%.1f °C]" <temperature> (Heizunglog) { http="<[http://192.168.178.88/8700:60000:JS(bsbinput.js)]" }
String hz_700 "Heizkreis 1 Betriebsart [%s]" <temperature> (Heizunglog){ http="<[http://192.168.178.88/700:1000:JS(bsbinput_string.js)]" }
```
    
Das folgende Javascript ist als *bsbinput.js* im Verzeichnis
*transformations* abzulegen.

***Beispielscript für Abfragen von Parametern, bei denen ein Wert
ausgegeben wird:***  
    
```javascript
(function(i) {
    var outputres;
	var results = [];
	value1 = i;
	// define regex to search for the line in the http response
	var regEx = '<input type=text id=\'value1\' VALUE=\'[-]*[0-9]+\.[0-9]+';
	var re = new RegExp(regEx, 'gim');
  
 do {
    match = re.exec(value1);
    if (match){
        results.push(match[0]);
    }
} while (match);
    
	outputres = results[0]
	//extract actual value from the output
	var output=outputres.substr(outputres.indexOf("VALUE='")+7,outputres.length);
	return output;
})(input)
```
    
***Beispielscript für direkte Abfragen von enum-Werten:***  
    
```javascript
(function(i) {
        var outputres;
	var results = [];
	value1 = i;
	// define regex to search for the line in the http response
	var regEx = '<option value=\'[0-9]+\' SELECTED>.*</option>';
	var re = new RegExp(regEx, 'gim');
  
 do {
    match = re.exec(value1);
    if (match){
        results.push(match[0]);
    }
} while (match);
    
	outputres = results[0]
	//extract actual value from the output
	var l=outputres.indexOf("</o")-outputres.indexOf(">")-1
	var output=outputres.substr(outputres.indexOf(">")+1,l);
		return output;
})(input)
```
    
Das Schreiben von Daten erfolgt über Rules:  
    
```
rule "RoomTemp"

when
	Item iSet_temp changed
then
	sendHttpGetRequest("http://192.168.178.88/I10000=" + iSet_temp.state.toString)
end
```
    
## 11.3 HomeMatic (EQ3) ##  
    
***Die folgenden HomeMatic-Beispielscripte stammen vom FHEM-Forumsmitglied „PaulM".  
Sie sind ebenfalls [hier](https://forum.fhem.de/index.php/topic,29762.msg769167.html#msg769167) im FHEM-Forum zu finden.  
Im Anschluss daran sind die Beispielskripte vom FHEM-Forumsmitglied "Bratmaxe" aufgeführt.  
Vielen Dank!***  
    
Zur Einbindung in HomeMatic bietet sich die Verwendung von CuxD und wget
an.

***Beispiel zur Abfrage des Parameters ‚8326 Brennermodulation' mittels
CuxD:***  
    
```
! Skriptanfang:
! BSB-Abfrage
string url='http://192.168.178.88/8326'; !IP anpassen
! WriteLine("URL: " # url); !nur Kontrolle
! --timeout=seconds  ! während der Dauer der Abfrage werden von der CCU 
! keine anderen Skripte ausgeführt !!! 
! siehe CUxD-Handbuch 5.8.2 System.Exec
! dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State
! ("wget -t 5 -T 20 -q -O - '"# url #"'");  ! abgekürzte Wget Syntax
dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget --tries=5 
--timeout=20 --quiet --output-document=- '"# url #"'");  
! ausführliche Wget Syntax
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State();
! WriteLine("Antwort: " # stdout);
! Beim stdout den ueberfluessigen Vorspann entfernen
integer laenge = stdout.Length();
integer pos = stdout.Find("body"); 
! ab hier kommen auswertbare Informationen
pos = pos + 10;
stdout = stdout.Substr(pos, (laenge - pos));
! wenn Rückmeldung mit allen Daten angezeigt werden soll, 
! Ausrufezeichen nächste Zeile entfernen
! WriteLine("Antwort ohne Vorspann: " # stdout);
string wort = "8326"; !Brennermodulation
integer laenge = wort.Length();
! WriteLine("laenge: " # laenge); zum Testen für andere Parameter 
! für Skripttest Ausrufezeichen entfernen
integer pos = stdout.Find(wort);
! WriteLine("pos:" #pos);
pos = pos + 39; !bei anderen Parametern meist zwischen 25 und 50
string daten = stdout.Substr((pos + laenge +1), 100);
! WriteLine("daten :"#daten);
integer pos = daten.Find(wort);
daten = daten.Substr(0, (pos));
integer zahl = daten.ToFloat();
! keine Prüfung, da auch 0 sein kann
dom.GetObject("H_Brennermodulation").State(zahl);
WriteLine("H_Brennermodulation: "#zahl);
! nur für Chart CUxD
dom.GetObject("CUxD.CUX2801001:1.LOGIT").State
("H_Brennermodulation"#";"#zahl);
WriteLine("Hallo Welt!");
! Skriptende:
```
    
***Beispiel zum Setzen der Betriebsart auf Komfort-Betrieb mit ‚S700=3'
mittels CuxD:***  
    
```
! Skriptanfang:
! Heizung KOMFORT (=AN - keine Nachtabsenkung)
! Anweisung senden
string urlset ='http://192.168.178.88/S700=3'; !IP anpassen
WriteLine("Befehl: " # urlset);
dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State
("wget -t 5 -T 20 -q -O - '"# urlset #"'");
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State();
! Kontrollabfrage
string url='http://192.168.178.88/700'; !IP anpassen
! WriteLine("URL: " # url);
dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State
("wget -t 5 -T 20 -q -O - '"# url #"'");
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State();
! WriteLine("Antwort: " # stdout);
! Beim stdout den ueberfluessigen Vorspann entfernen
integer laenge = stdout.Length();
integer pos = stdout.Find("body");
pos = pos + 10;
stdout = stdout.Substr(pos, (laenge - pos));
! wenn Rückmeldung mit allen Daten angezeigt werden soll, 
! Ausrufezeichen nächste Zeile entfernen
! WriteLine("Antwort ohne Vorspann: " # stdout);
string wort = "700"; !Heizbetrieb
integer laenge = wort.Length();
! WriteLine("laenge: " # laenge);
integer pos = stdout.Find(wort);
! WriteLine("pos:" #pos);
pos = pos + 28;
! WriteLine("pos: " # pos);	
string daten = stdout.Substr((pos + laenge +1), 100);
! WriteLine("daten :"#daten);
string substring = daten.Substr(0, 1);
integer zahl = substring.ToInteger();
WriteLine("aktueller Heizbetrieb (0 bis 3): " # zahl);
if (zahl == 0) {dom.GetObject('H_Heizbetrieb').State("Heizung AUS");}
if (zahl == 1) {dom.GetObject('H_Heizbetrieb').State("Heizung Automatik");}
if (zahl == 2) {dom.GetObject('H_Heizbetrieb').State("Heizung Nachtabsenkung");}
if (zahl == 3) {dom.GetObject('H_Heizbetrieb').State("Heizung Komfort");}
! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Heizbetrieb"#";"#zahl);	
! alle Werte aktualisieren
var programObj = dom.GetObject("Heizungswerte abfragen"); 
! Programmnamen ggf. anpassen
programObj.ProgramExecute();
WriteLine("Hallo Welt!");
! Skriptende:
```

***Heizung Abfrage und speichern als Systemvariable:***  
Ferienbetrieb von-bis / Absenkniveau (632/633/648)  
Heiz- und Warmwasserbetrieb (700/1600/8700/8326/8743/8314/8830)  
Übertrag der von einem One-Wire Sensor gemessenen Raumtemperatur zum BSB (I10000)  
Protokollieren (für Auswertungen mit CUxD Highcharts)  
    
```
!Heizung Abfrage V17 mit CUxD 2018-02-17

!wget mit ausführl. Syntax 2018-02-17
!inkl. Daten Ferienbetrieb und -niveau 2018-01-31

!Prüfung of Rückgabewerte ungleich Null
!auch senden an CUxD-Geräte Tempsens_H_*

!H_Trinkwassertemperatur korrigiert +1.3  2017-08-11
!mit Übertrag der Raumtemperatur an die Heizung Parameter 8740 2017-05-13
!mit OneWire Sensoren 2017-04-26

!als Systemvariablen sind in Homematic angelegt:
!H_Ferien_Beginn Zeichenkette
!H_Ferien_Ende Zeichenkette
!H_Ferienniveau Zeichenkette

!H_Aussentemperatur	Zahl
!H_Brennermodulation	Zahl
!H_Vorlauftemperatur	Zahl
!H_Kesselruecklauftemperatur	Zahl
!H_Trinkwassertemperatur	Zahl
!H_Heizbetrieb 	Zeichenkette
!H_Trinkwasserbetrieb	Zeichenkette

!632	Beginn Ferienbetrieb	TT.MM
!633	Ende Ferienbetrieb	TT.MM 
!648  Frostschutz / Reduziert

string url='http://192.168.10.13/632/633/648/700/1600/8700/8326/8743/8314/8830/T';

!WriteLine("URL: " # url);

!--timeout=seconds  ! während der Dauer der Abfrage werden von der CCU keine anderen Skripte ausgeführt 
! CUxD-Handbuch 5.8.2 System.Exec "Es ist zu beachten, dass die Verarbeitung des HM-Scripts erst fortgesetzt wird, nachdem das aufgerufene Programm beendet wurde. Während dieser Zeit werden keine anderen HM-Scripts ausgeführt!"

!dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# url #"'");  ! abgekürzte Wget Syntax
dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget --tries=5 --timeout=20 --quiet --output-document=- '"# url #"'");  ! ausführliche Wget Syntax
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
!WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	!WriteLine("Antwort ohne Vorspann: " # stdout);

string wort = "632"; !Ferien Heizkreis 1 Beginn TT.MM
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 48;
	!WriteLine("pos: " # pos);	
	string daten = stdout.Substr((pos + laenge +1), 13);
	daten = daten.Substr(2,5);
	!WriteLine("daten :"#daten);
	dom.GetObject('H_Ferien_Beginn').State("Ferien ab: ???"); ! setzt vorherigen Wert zurück
	dom.GetObject('H_Ferien_Beginn').State("Ferien ab: "# daten);
	var temp = dom.GetObject('H_Ferien_Beginn').Value();
	WriteLine("Ferien Heizkreis 1 Beginn TT.MM: " # temp);	
	
string wort = "633"; !Ferien Heizkreis 1 Ende TT.MM
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 48;
	!WriteLine("pos: " # pos);	
	string daten = stdout.Substr((pos + laenge +1), 15);
	daten = daten.Substr(0,5);
	!WriteLine("daten :"#daten);
	dom.GetObject('H_Ferien_Ende').State("Ferien bis: ???"); ! setzt vorherigen Wert zurück
	dom.GetObject('H_Ferien_Ende').State("Ferien bis: "# daten);
	var temp = dom.GetObject('H_Ferien_Ende').Value();
	WriteLine("Ferien Heizkreis 1 Ende TT.MM: " # temp);	
	
string wort = "648"; !Betriebsniveau Ferien
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 48;
	!WriteLine("pos: " # pos);	
	string daten = stdout.Substr((pos + laenge +1), 15);
	daten = daten.Substr(0,12);
	!WriteLine("daten :"#daten);
	dom.GetObject('H_Ferienniveau').State("Ferienniveau: ???"); ! setzt vorherigen Wert zurück
	dom.GetObject('H_Ferienniveau').State("Ferienniveau: "# daten);
	var temp = dom.GetObject('H_Ferienniveau').Value();
	WriteLine("Betriebsniveau Ferien: " # temp);
	
	
string wort = "700"; !Heizbetrieb
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 28;
	!WriteLine("pos: " # pos);	
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	string substring = daten.Substr(0, 1);
	integer zahl = substring.ToInteger();
	!WriteLine("zahl: " # zahl);
		if (zahl == 0) {dom.GetObject('H_Heizbetrieb').State("Heizung AUS");}
		if (zahl == 1) {dom.GetObject('H_Heizbetrieb').State("Heizung Automatik");}
		if (zahl == 2) {dom.GetObject('H_Heizbetrieb').State("Heizung Nachtabsenkung");}
		if (zahl == 3) {dom.GetObject('H_Heizbetrieb').State("Heizung Komfort");}
		
! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Heizbetrieb"#";"#zahl);


string wort = "1600"; !Trinkwasserbetrieb
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 35;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	string substring = daten.Substr(0, 1);
	integer zahl = substring.ToInteger();
	!WriteLine("zahl: " # zahl);
		if (zahl == 0) {dom.GetObject('H_Trinkwasserbetrieb').State("Warmwasserbetrieb - AUS");}
		if (zahl == 1) {dom.GetObject('H_Trinkwasserbetrieb').State("Warmwasserbetrieb - EIN");}

	! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Trinkwasserbetrieb"#";"#zahl);


string wort = "8700"; !Aussentemperatur
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 41;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	!Korrektur des angezeigten Wertes auf das Niveau der übrige Thermometer
	if (zahl<>0)
		{
		zahl = zahl - 3.5;
		dom.GetObject("H_Aussentemperatur").State(zahl);
		WriteLine("H_Aussentemperatur: "#zahl); 

		! nur für Chart CUxD 
		dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Aussentemperatur"#";"#zahl);

		! für CUxD Gerät Tempsens_H_Aussen
		dom.GetObject("CUxD.CUX9002012:1.SET_TEMPERATURE").State(zahl);	
		}
	
string wort = "8326"; !Brennermodulation
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 39;;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	!keine Prüfung, da auch 0 sein kann
	dom.GetObject("H_Brennermodulation").State(zahl);
	WriteLine("H_Brennermodulation: "#zahl); 

	! nur für Chart CUxD 
	dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Brennermodulation"#";"#zahl);

string wort = "8743"; !Vorlauftemperatur 1:
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 45;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	if (zahl<>0)
		{
		dom.GetObject("H_Vorlauftemperatur").State(zahl);
		WriteLine("H_Vorlauftemperatur: "#zahl); 

		! nur für Chart CUxD 
		dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Vorlauftemperatur"#";"#zahl);

		! für CUxD Gerät Tempsens_H_KesselVor
		dom.GetObject("CUxD.CUX9002014:1.SET_TEMPERATURE").State(zahl);	
		}
		
string wort = "8314"; !Kesselrücklauftemperatur
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 52;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	if (zahl<>0)
		{	
		dom.GetObject("H_Kesselruecklauftemperatur").State(zahl);
		WriteLine("H_Kesselruecklauftemperatur: "#zahl); 

		! nur für Chart CUxD 
		dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Kesselruecklauftemperatur"#";"#zahl);

		! für CUxD Gerät Tempsens_H_KesselRue
		dom.GetObject("CUxD.CUX9002013:1.SET_TEMPERATURE").State(zahl);	
		}
		
string wort = "8830"; !Trinkwassertemperatur
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 48;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	!Korrektur relativ zur Kesselvorlauftemperatur
	if (zahl<>0)
		{	
		zahl = zahl + 1.3;
		dom.GetObject("H_Trinkwassertemperatur").State(zahl);
		WriteLine("H_Trinkwassertemperatur "#zahl); 

		! nur für Chart CUxD 
		dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Trinkwassertemperatur"#";"#zahl);

		! für CUxD Gerät Tempsens_H_Trinkw
		dom.GetObject("CUxD.CUX9002015:1.SET_TEMPERATURE").State(zahl);	
		}

!Übertrag der Raumtemperatur Esszimmer  ...4d6 an die Heizung als Parameter 8740

	real temp = dom.GetObject("T_Innentemperatur_Esszimmer").Value();
	temp = temp.ToString();

	string urlset = 'http://192.168.10.13/I10000='# temp;
	!WriteLine("url " # urlset);

	dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# urlset #"'");   
	dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
	var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
	!WriteLine("stdout: " # stdout);
	
	!Abfrage, ob Setzen des Wertes ok  
	string url='http://192.168.10.13/8740';

	dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# url #"'");   
	dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
	var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
	!WriteLine("stdout: " # stdout);

	string wort = "8740"; !Raumtemperatur 1
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 42;
	string daten = stdout.Substr((pos + laenge +1), 5);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	!integer zahl = daten.ToFloat();
	WriteLine("an die Heizung übertragene Raumtemperatur (8740) "#daten #"°C"); 

WriteLine("Hallo Welt!");
```
    
***Anweisungen zum Ändern des Betriebs von Heizung und Trinkwasser:***  
Damit Syntax-sichere Anweisungen von CCU an BSB gegeben werden können (wichtig z.B. auch wenn via VPN kein direkter Zugang zum BSB-Adapter möglich ist).  

*Heizung AUS (= Frostschutzbetrieb):*  
    
```
!Heizung AUS (=Frostschutzbetrieb)  2017-03-09

string urlset ='http://192.168.10.13/S700=0';
WriteLine("Befehl: " # urlset);

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# urlset #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 

string url='http://192.168.10.13/700';

WriteLine("URL: " # url);

!--timeout=seconds

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# url #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	
!Werte aktualisieren
var programObj = dom.GetObject("Heizungswerte abfragen");
programObj.ProgramExecute();	
	
	WriteLine("Antwort ohne Vorspann: " # stdout);
WriteLine("Hallo Welt!");
```  
    
*Heizung Automatik (= AN - mit Nachtabsenkung):*  
```
!Heizung Automatik (=AN - mit Nachtabsenkung)  2017-03-09

!http://192.168.10.13/Passwort/S700=0 (Schutzbetrieb)
!http://192.168.10.13/Passwort/S700=1 (Automatik)
!http://192.168.10.13/Passwort/S700=2 (Reduziert)
!http://192.168.10.13/Passwort/S700=3 (Komfort)

string urlset ='http://192.168.10.13/S700=1';
WriteLine("Befehl: " # urlset);

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# urlset #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 

string url='http://192.168.10.13/700';

WriteLine("URL: " # url);

!--timeout=seconds

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# url #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	WriteLine("Antwort ohne Vorspann: " # stdout);

!Werte aktualisieren
var programObj = dom.GetObject("Heizungswerte abfragen");
programObj.ProgramExecute();	
	
WriteLine("Hallo Welt!");
```
    
*Heizung KOMFORT (= AN - keine Nachtabsenkung)*:  
```
!Heizung KOMFORT (=AN - keine Nachtabsenkung)  2017-03-09

!http://192.168.10.13/Passwort/S700=0 (Schutzbetrieb)
!http://192.168.10.13/Passwort/S700=1 (Automatik)
!http://192.168.10.13/Passwort/S700=2 (Reduziert)
!http://192.168.10.13/Passwort/S700=3 (Komfort)

! Anweisung senden
string urlset ='http://192.168.10.13/S700=3';
WriteLine("Befehl: " # urlset);

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# urlset #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 

! Kontrollabfrage
string url='http://192.168.10.13/700';

WriteLine("URL: " # url);

!--timeout=seconds

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# url #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	WriteLine("Antwort ohne Vorspann: " # stdout);
	
!alle Werte aktualisieren
var programObj = dom.GetObject("Heizungswerte abfragen");
programObj.ProgramExecute();	
	
WriteLine("Hallo Welt!");
```
    
*Heizung NACHTABSENKUNG (dauernd, d.h. auch tagsüber!):*  
```
!Heizung Nachtabsenkung (dauernd, d.h. auch tagsüber Nachtabsenkung)  2017-03-09

!http://192.168.10.13/Passwort/S700=0 (Schutzbetrieb)
!http://192.168.10.13/Passwort/S700=1 (Automatik)
!http://192.168.10.13/Passwort/S700=2 (Reduziert)
!http://192.168.10.13/Passwort/S700=3 (Komfort)


string urlset ='http://192.168.10.13/S700=2';
WriteLine("Befehl: " # urlset);

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# urlset #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 

string url='http://192.168.10.13/700';

WriteLine("URL: " # url);

!--timeout=seconds

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -q -O - '"# url #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	WriteLine("Antwort ohne Vorspann: " # stdout);
	
!Werte aktualisieren
var programObj = dom.GetObject("Heizungswerte abfragen");
programObj.ProgramExecute();	
	
WriteLine("Hallo Welt!");
```
    
***Abfrage der Tagesdurchschnitte /A bestimmter Parameter und speichern als Systemvariable:***  
    
```
!Heizung Abfrage Tagesdurchschnitte V5 2017-10-07

!Raumtemperatur Ist und Trinkwassertemperatur ergänzt
!Variablen reduziert

!als Systemvariablen sind in Homematic angelegt:
!H_Aussentemperatur_24h				Zahl	8700
!H_Brennermodulation_24h			Zahl	8326
!H_Vorlauftemperatur_24h			Zahl	8743
!H_Kesselruecklauftemperatur_24h	Zahl	8314

!H_Raumtemperatur_Ist_24h			Zahl	8740
!H_Raumtemperatur_Soll_24h  ???
!H_Trinkwassertemperatur_24h		Zahl	8830

string url='http://192.168.10.13/A';

WriteLine("URL: " # url);

!--timeout=seconds

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 -qO- '"# url #"'");   
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
!WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	WriteLine("Antwort ohne Vorspann: " # stdout);
	
!----------------------------------------------------------------------------

string wort = "8700"; !Aussentemperatur
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 20;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	!Korrektur des angezeigten Wertes auf das Niveau der übrige Thermometer
	zahl = zahl - 3.5;
	dom.GetObject("H_Aussentemperatur_24h").State(zahl);
	WriteLine("H_Aussentemperatur_24h: "#zahl); 

! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Aussentemperatur_24h"#";"#zahl);

!----------------------------------------------------------------------------

string wort = "8326"; !Brennermodulation
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 21;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	dom.GetObject("H_Brennermodulation_24h").State(zahl);
	WriteLine("H_Brennermodulation_24h: "#zahl); 

! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Brennermodulation_24h"#";"#zahl);

!----------------------------------------------------------------------------

string wort = "8743"; !Vorlauftemperatur 1:
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 23;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	!daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	dom.GetObject("H_Vorlauftemperatur_24h").State(zahl);
	WriteLine("H_Vorlauftemperatur_24h: "#zahl); 

! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Vorlauftemperatur_24h"#";"#zahl);

!----------------------------------------------------------------------------

string wort = "8314"; !Kesselrücklauftemperatur
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 33;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	dom.GetObject("H_Kesselruecklauftemperatur_24h").State(zahl);
	WriteLine("H_Kesselruecklauftemperatur_24h: "#zahl); 

! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Kesselruecklauftemperatur_24h"#";"#zahl);

!----------------------------------------------------------------------------

string wort = "8740"; !Raumtemperatur 1
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 21;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	dom.GetObject("H_Raumtemperatur_Ist_24h").State(zahl);
	WriteLine("H_Raumtemperatur_Ist_24h: "#zahl); 

! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Raumtemperatur_Ist_24h"#";"#zahl);

!----------------------------------------------------------------------------
string wort = "8830"; !Trinkwassertemperatur
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 28;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	dom.GetObject("H_Trinkwassertemperatur_24h").State(zahl);
	WriteLine("H_Trinkwassertemperatur_24h: "#zahl); 

! nur für Chart CUxD 
!dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("H_Trinkwassertemperatur_24h"#";"#zahl);


WriteLine("Hallo Welt!");
```
    
***Abfrage der angeschlossenen OneWire-Sensoren und speichern als Systemvariable:***  
    
```
!Temperatursensoren Abfrage CUxD 2017-10-13 (Auszug)


!als Systemvariablen sind in Homematic angelegt:
!28ff99e890160456 T_Aussentemperatur_Nord
!28ffff85901604d6 T_Innentemperatur_Esszimmer	

string url='http://192.168.10.13/ipwe.cgi';

!WriteLine("URL: " # url);

!--timeout=seconds

dom.GetObject("CUxD.CUX2801001:1.CMD_SETS").State("wget -t 30 --timeout=20 -q -O - '"# url #"'");
dom.GetObject("CUxD.CUX2801001:1.CMD_QUERY_RET").State(1);          
var stdout = dom.GetObject("CUxD.CUX2801001:1.CMD_RETS").State(); 
!WriteLine("Antwort: " # stdout);

!Beim stdout den ueberfluessigen Vorspann entfernen
	integer laenge = stdout.Length();
	integer pos = stdout.Find("body");
	pos = pos + 10;
	stdout = stdout.Substr(pos, (laenge - pos));
	!wenn Rückmeldung mit allen Daten angezeigt werden soll, Ausrufezeichen nächste Zeile entfernen
	!WriteLine("Antwort ohne Vorspann: " # stdout);

string wort = "28ff99e890160456"; !T_Aussentemperatur_Nord
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 11;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	if (zahl<>0)
		{
		WriteLine("T_Aussentemperatur_Nord gemessen: "#zahl); 
		dom.GetObject("T_Aussentemperatur_Nord").State(zahl);
		! nur für Chart CUxD 
		dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("T_Aussentemperatur_Nord"#";"#zahl);	
		}
	
string wort = "28ffff85901604d6"; !T_Innentemperatur_Esszimmer
	integer laenge = wort.Length();
	!WriteLine("laenge: " # laenge);
	integer pos = stdout.Find(wort);
	!WriteLine("pos:" #pos);
	pos = pos + 11;
	string daten = stdout.Substr((pos + laenge +1), 100);
	!WriteLine("daten :"#daten);
	integer pos = daten.Find(wort);
	daten = daten.Substr(0, (pos));
	integer zahl = daten.ToFloat();
	if (zahl<>0)
		{
		dom.GetObject("T_Innentemperatur_Esszimmer").State(zahl);
		WriteLine("T_Innentemperatur_Esszimmer: "#zahl); 	
		! nur für Chart CUxD 
		dom.GetObject("CUxD.CUX2801001:1.LOGIT").State("T_Innentemperatur_Esszimmer"#";"#zahl);
		}	

WriteLine("Hallo Welt!");
```
        
    
***Die folgenden HomeMatic-Beispielscripte stammen vom FHEM-Forumsmitglied „Bratmaxe".  
Sie sind samt einer genaueren Beschreibung ebenfalls [hier](https://forum.fhem.de/index.php/topic,29762.msg851779.html#msg851779) im FHEM-Forum zu finden (die hier eingefügten Beschreibungen wurden von dort größtenteils unverändert übernommen).  
Vielen Dank!***  
    
***Beispielscript für die Abfrage des Adapters:***  

Es müssen lediglich 6 Parameter eingegeben werden.  
CuxGeraetAbfrage = GeräteAdresse des CuxD Execute Gerätes, welches die Abfragen ausführt  
CuxGeraetLogging = GeräteAdresse des CuxD Execute Gerätes, welches das Logging ausführt (leer==deaktiviert)  
IPAdresseBSB = IP-Adresse des BSB-Adapters  
Wort = Parameternummer: Beispiel Außentemperatur = 8700 oder Betriebsmodus = 700  
Variablename = Name der Systemvariable in der CCU  
Durchschnitt24h = true == Durchschnittswert 24h holen, false == aktuellen Wert holen  

Es muss keine Variable vorher angelegt werden, das erledigt das Skript.  
Der Variabeltyp (Zahl, Bool, Werteliste) wird automatisch an den abgefragten Parameter angepasst.  
    
```
! BSB-Adapter Wert abfragen by Bratmaxe
! 29.10.2018 - V0.1 - Erste Version

string CuxGeraetAbfrage = "CUX2801001:1"; ! GeräteAdresse des CuxD Execute Gerätes, welches die Abfragen ausführt
string CuxGeraetLogging = "CUX2801001:1"; ! GeräteAdresse des CuxD Execute Gerätes, welches das Logging ausführt, Leer ("") lassen, wenn kein Cuxd-Highcharts Logging gewünscht
string IPAdresseBSB = "192.168.178.100"; !IP_Adresse des BSB-Adapters
string Wort = "8700"; !Parameternummer: Beispiel Außentemperatur = 8700, Betriebsmodus = 700
string Variablename = "Wetter_Temperatur_Heizung"; ! Name der Systemvariable
boolean Durchschnitt24h = false; ! true = Durchschnittswert holen, false = aktuellen Wert holen - diese muss vorher in der BSB_lan_config.h konfiguriert wurden sein!!!

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!Ab hier keine Anpassungen mehr notwendig!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

! URL Zusammenführen
string url="";
if (Durchschnitt24h) { url="http://" # IPAdresseBSB # "/A" # Wort; }
else { url="http://" # IPAdresseBSB # "/" # Wort; }
! Variable anlegen, wenn nicht vorhanden:
object svObject = dom.GetObject(Variablename);
object svObjectlist = dom.GetObject(ID_SYSTEM_VARIABLES);
if (!svObject)
{
    svObject = dom.CreateObject(OT_VARDP);
    svObjectlist.Add(svObject.ID());
	svObjectlist = dom.GetObject(ID_SYSTEM_VARIABLES);
    svObject.Name(Variablename);   
    svObject.Internal(false);
    svObject.Visible(true);
}

! Werte holen
dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_SETS").State("wget --tries=5 --timeout=20 --quiet --output-document=- '"# url #"'"); 
dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_QUERY_RET").State(1);
var stdout = dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_RETS").State();

! Prüfe, ob eine Ausgabe vorhanden ist, sonst z.B. IP-Adresse falsch, oder Netzwerkfehler
if (stdout != null && stdout != "")
{
	! Ausgabe filtern
	integer pos = (stdout.Find(Wort# " "));	
	if (pos == -1)
	{
		WriteLine("Position vom Wort '" # Wort # "' konnte nicht ermittelt werden");
	}
	
	stdout = stdout.Substr(pos, stdout.Length());
	pos = stdout.Find("/td");
	stdout = stdout.Substr(0, pos);
	
	! Sonderzeichen ersetzen
	if (stdout.Contains("&deg;")){ stdout = stdout.Replace("&deg;","°"); }
	if (stdout.Contains("&#037;")){ stdout = stdout.Replace("&#037;","%"); }
	stdout = stdout.ToLatin();
	!WriteLine("Nach Sonderzeichenumwandlung: " # stdout); !Debug: Welchen Wert hat stdout aktuell

	! Systemvariabel Info ermitteln
	string Info = stdout.Substr(0,stdout.Find(":"));
	!Info = Info.Substr(Wort.Length(), stdout.Length()); !Parameterzahl vor der Info entfernen
	!WriteLine("DPInfo = " # Info); !Debug: Welcher DPInfo-Wert wurde gefunden
	
	! Systemvariabel Wert ermitteln
	string Wert = stdout.Substr(stdout.Find(": ") + 2,stdout.Length());
	Wert = Wert.Substr(0,Wert.Find(" "));
	!WriteLine("Wert = " # Wert); !Debug: Welcher Wert wurde gefunden

	! Systemvariabel Einheit ermitteln
	string Einheit = stdout.Substr(stdout.Find(Info) + Info.Length() + 1, stdout.Length());
	Einheit = Einheit.Substr(Einheit.Find(Wert) + Wert.Length() + 1,Einheit.Length());
	Einheit = Einheit.RTrim();
	if (Einheit.Contains("- "))	{ Einheit = ""; }
	!WriteLine("Einheit = " # Einheit); !Debug: Welche Einheit wurde gefunden

	! Systemvariable Typ und Werte setzen
	svObject.DPInfo(Info);
	svObject.ValueUnit(Einheit);
	
	! Enums des Parameters ermitteln, wenn vorhanden
	url="http://" # IPAdresseBSB # "/E" # Wort;

	dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_SETS").State("wget --tries=5 --timeout=20 --quiet --output-document=- '"# url #"'"); 
	dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_QUERY_RET").State(1);
	stdout = dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_RETS").State();
	! Prüfe, ob es sich um einen Parameter mit Enum-Werten handelt.
	if (!stdout.Contains("FEHLER: Falscher Typ!"))
	{
		! Setzen des Systemvariabel Wertetyp und Ermitteln der Enum-Werte des Parameters
		stdout = (stdout.Substr(stdout.Find("0 - "), stdout.Length())).ToLatin();
		string value = "";
		string newvalues = "";
		integer inewvalues=0;
		foreach (value, stdout.Split("\r"))
		{
			if (value.Contains(" - "))
			{
				if (newvalues == "") { newvalues = newvalues # value.Substr(value.Find(" - ") + 3,value.Length()); }
				else { newvalues = newvalues # ";" # value.Substr(value.Find(" - ") + 3,value.Length()); }
				inewvalues = inewvalues + 1;
			}
		}
		
		svObject.ValueType(ivtInteger);
		svObject.ValueSubType(istEnum);
		svObject.ValueList(newvalues);
		!prüft, ob der ermittelte Wert innerhalbe der möglichen Werte liegt
		if (Wert < inewvalues) { if (Wert != svObject.Value()) { svObject.State(Wert); } }
		else { WriteLine("Der ermittelte Wert entspricht keinem gültigen Enum-Wert. Bitte Ausgabe prüfen!") }
	}
	elseif (Einheit.Contains("- Aus") || Einheit.Contains("- Ein"))
	{
		! Setzen des Systemvariabel Wertetyp
		svObject.ValueType(ivtBinary);
		svObject.ValueSubType(istBool);
		svObject.ValueName0("Aus");
		svObject.ValueName1("Ein");
		if (Wert != svObject.Value()) {	svObject.State(Wert); }
	}
	elseif (Einheit.Contains("°"))
	{
		! Setzen des Systemvariabel Wertetyp
		svObject.ValueType(ivtFloat);
		svObject.ValueSubType(istGeneric);
		svObject.ValueMin(-50);
		svObject.ValueMax(100);
		if (Wert != svObject.Value()) {	svObject.State(Wert); }
	}
	elseif (Einheit.Contains("%"))
	{
		! Setzen des Systemvariabel Wertetyp
		svObject.ValueType(ivtFloat);
		svObject.ValueSubType(istGeneric);
		svObject.ValueMin(0);
		svObject.ValueMax(100);
		if (Wert != svObject.Value()) {	svObject.State(Wert); }
	}
	else
	{
		! Setzen des Systemvariabel Wertetyp
		svObject.ValueType(ivtFloat);
		svObject.ValueSubType(istGeneric);
		if (Wert != svObject.Value()) {	svObject.State(Wert); }
	}
	dom.RTUpdate(0); ! Interne Aktualisierung der Systemvariabelen

	! Logging
	if (CuxGeraetLogging != "")	{ dom.GetObject("CUxD."#CuxGeraetLogging#".LOGIT").State(dom.GetObject(ID_SYSTEM_VARIABLES).Get(Variablename).Name()#";"#dom.GetObject(ID_SYSTEM_VARIABLES).Get(Variablename).Value());	}
}

```
    
***Skript zum Setzen von Parametern:***  

Ein Programm, wo alle Systemvariabeln die überwacht werden sollen mit ODER Verknüpft und größer oder gleich 0 und "bei Aktualisierung auslösen", anlegen.  
Beispiel: 
   
```
WENN Variablename größer oder gleich 0 "bei Aktualisierung auslösen"
DANN Dieses SKRIPT sofort ausführen
```
    
Die Variable muss in der Info zuerst den ParameterWert enthalten (wird vom obigen Auslese-Skript automatisch so benannt).   Beispiel: 700 Heizkreis 1 - Betriebsart  
Die Parameternummer wird dann automatisch aus der Systemvariable Info ermittelt.  
Wird die Variable geändert, so wird der geänderte Wert automatisch an den BSB-Adapter übermittelt und aktualisiert!  
    
```
! BSB-Adapter Wert setzen by Bratmaxe
! 29.10.2018 - V0.1 - Erste Version

! Funktionsbeschreibung:
! Ein Programm, wo alle Systemvariabeln die Überwacht werden sollen mit ODER Verknüpft und größer oder gleich 0 und  "bei Aktualisierung auslösen" , anlegen.
! Beispiel:
! WENN Variablename größer oder gleich 0 "bei Aktualisierung auslösen"
! DANN Dieses SKRIPT sofort ausführen
! die Variable muss in der Info zuerst den Parameter-Wert enthalten (wird von meinem Auslese Skript automatisch so benannt. Beispiel: 700 Heizkreis 1 - Betriebsart
! Die Parameternummer wird dann automatisch aus der Systemvariable Info ermittelt.
! Wird die Variable geändert, so wird der geänderte Wert automatisch an den BSB-Adapter übermittelt und aktualisiert!

string CuxGeraetSetzen = "CUX2801001:12"; ! GeräteAdresse des CuxD Execute Gerätes
string IPAdresseBSB = "192.168.2.200"; !IP_Adresse des BSB-Adapters

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!Ab hier keine Anpassungen mehr notwendig!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

! Hole Auslösende Variabel
var source = dom.GetObject("$src$"); !Funktioniert nur beim automatischen Aufruf
! Zum manuellen Aufruf/testen nächste Zeile einkommentieren
!source = dom.GetObject(ID_SYSTEM_VARIABLES).Get("VARIABLENAMEN");

if (source)
{
	! Wort ermitteln
	string Wort = source.DPInfo().ToString().Substr(0,source.DPInfo().Find(" "));
	!WriteLine("Wort: "#Wort);
	if (Wort != null && Wort != "")
	{		
		string Wert = source.Value().ToString();
		!WriteLine("Wert: "#Wert);
		if (Wert != null && Wert != "")
		{
			! Anweisung senden
			string urlset="http://" # IPAdresseBSB # "/S" # Wort # "=" # Wert;
			dom.GetObject("CUxD." # CuxGeraetSetzen # ".CMD_SETS").State("wget -t 5 -T 20 -q -O - '"# urlset #"'");
			dom.GetObject("CUxD." # CuxGeraetSetzen # ".CMD_QUERY_RET").State(1);
			var stdout = dom.GetObject("CUxD." # CuxGeraetSetzen # ".CMD_RETS").State();
			if (stdout != null && stdout != "")
			{
				if (stdout.Contains("FEHLER: "))
				{
					stdout = stdout.Substr(stdout.Find("FEHLER: "), stdout.Length());
					stdout = stdout.Substr(0, stdout.Find("/td"));
					WriteLine("Fehlermeldung: "# stdout);
					WriteLine("Wurde der BSB-Adapter zum Schreiben berechtigt? Handbuch Seite 26 beachten...");
				}
				else
				{
					! Kontrollabfrage
					string url="http://" # IPAdresseBSB # "/" # Wort;
					dom.GetObject("CUxD." # CuxGeraetSetzen # ".CMD_SETS").State("wget -t 5 -T 20 -q -O - '"# url #"'");
					dom.GetObject("CUxD." # CuxGeraetSetzen # ".CMD_QUERY_RET").State(1);
					stdout = dom.GetObject("CUxD." # CuxGeraetSetzen # ".CMD_RETS").State();

					! Ausgabe filtern
					integer pos = (stdout.Find("tr  td \r\n" # Wort # " ") + 9);
					stdout = stdout.Substr(pos, stdout.Length());
					pos = stdout.Find("/td");
					stdout = stdout.Substr(0, pos);
					
					! Sonderzeichen ersetzen
					if (stdout.Contains("&deg;")){ stdout = stdout.Replace("&deg;","°"); }
					if (stdout.Contains("&#037;")){ stdout = stdout.Replace("&#037;","%"); }
					!WriteLine("Nach Sonderzeichenumwandlung: " # stdout); !Debug: Welchen Wert hat stdout aktuell
					
					! Systemvariabel oldWert ermitteln
					string oldWert = stdout.Substr(stdout.Find(": ") + 2,stdout.Length());
					oldWert = oldWert.Substr(0,oldWert.Find(" "));
					!WriteLine("oldWert = " # oldWert.ToFloat()); !Debug: Welcher oldWert wurde gefunden
					!WriteLine("newWert = " # Wert.ToFloat()); !Debug: Welcher oldWert wurde gefunden
					
					if (Wert.ToFloat() != oldWert.ToFloat()) { WriteLine("Fehler: Werte stimmen nach setzen nicht überein!"); }	
					else { WriteLine("Wert wurde erfolgreich gesetzt");	}
				}
			}
			else { WriteLine("Keine Ausgabe gefunden. IP-Adresse und Verkabelung prüfen.");	}
		}
		else { WriteLine("Der neue Wert konnte nicht ermittelt werden.");	}
	}
	else { WriteLine("Wort konnte nicht ermittelt werden, Steht der Wert in der SystemvariableInfo am Anfang gefolgt von einem Leerzeichen?");	}
}
else { WriteLine("Auslösende Variable nicht erkannt! - Skript wird nicht ausgeführt.");	}

```
    
***Abfrage von heizungsseitigen Fehlermeldungen zwecks Benachrichtigung im Störungsfall:***  
    
```
! BSB-Adapter Wert abfragen Fehlercodes by Bratmaxe
! 05.11.2018 - V0.1 - Erste Version

string CuxGeraetAbfrage = "CUX2801001:1"; ! GeräteAdresse des CuxD Execute Gerätes, welches die Abfragen ausführt
string IPAdresseBSB = "192.168.178.100"; !IP_Adresse des BSB-Adapters
string Variablename = "Heizung_Fehlercodes"; ! Name der Systemvariable
integer AnzahlFehler = 10;

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!Ab hier keine Anpassungen mehr notwendig!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

! Parameter Zusammenbauen
integer i =0;
string Woerter ="";
while (i < AnzahlFehler)
{
	if (Woerter != "")
	{
		Woerter = Woerter + "," + ((6801) + (10 * i)).ToString();
	}
	else { Woerter = Woerter + ((6801) + (10 * i)).ToString(); }
	i = i + 1;
}

! URL Zusammenführen
string Ergebnis = "";
string Wort = "";

foreach(Wort, Woerter.Split(","))
{
	string url="http://" # IPAdresseBSB # "/" # ((Wort.ToInteger() - 1).ToString());
	
	! Werte holen
	dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_SETS").State("wget --tries=5 --timeout=20 --quiet --output-document=- '"# url #"'"); 
	dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_QUERY_RET").State(1);
	var stdout = dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_RETS").State();
	
	! Prüfe, ob eine Ausgabe vorhanden ist, sonst z.B. IP-Adresse falsch, oder Netzwerkfehler
	if (stdout != null && stdout != "")
	{
		! Ausgabe filtern
		integer pos = (stdout.Find((Wort.ToInteger() - 1).ToString() #  " "));	
		stdout = stdout.Substr(pos, stdout.Length());
		pos = stdout.Find("/td");
		stdout = stdout.Substr(0, pos);
		
		! Sonderzeichen ersetzen
		if (stdout.Contains("°")){ stdout = stdout.Replace("°","°"); }
		if (stdout.Contains("%")){ stdout = stdout.Replace("%","%"); }
		stdout = stdout.ToLatin();
		Ergebnis = Ergebnis # stdout.RTrim() # "\n\r";
	}

	url="http://" # IPAdresseBSB # "/" # Wort;	
	dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_SETS").State("wget --tries=5 --timeout=20 --quiet --output-document=- '"# url #"'"); 
	dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_QUERY_RET").State(1);
	stdout = dom.GetObject("CUxD." # CuxGeraetAbfrage # ".CMD_RETS").State();

	! Prüfe, ob eine Ausgabe vorhanden ist, sonst z.B. IP-Adresse falsch, oder Netzwerkfehler
	if (stdout != null && stdout != "")
	{
		! Ausgabe filtern
		integer pos = (stdout.Find(Wort# " "));	
		stdout = stdout.Substr(pos, stdout.Length());
		pos = stdout.Find("/td");
		stdout = stdout.Substr(0, pos);
		
		! Sonderzeichen ersetzen
		if (stdout.Contains("°")){ stdout = stdout.Replace("°","°"); }
		if (stdout.Contains("%")){ stdout = stdout.Replace("%","%"); }
		stdout = stdout.ToLatin();
		Ergebnis = Ergebnis # stdout.RTrim() # "\n\r\n\r";
	}
}

!Wenn noch keine Systemvarible vorhanden, diese anlegen
object svObject = dom.GetObject(Variablename);
object svObjectlist = dom.GetObject(ID_SYSTEM_VARIABLES);
if (!svObject)
{   
    svObjectlist = dom.GetObject(ID_SYSTEM_VARIABLES);
    svObject = dom.CreateObject(OT_VARDP);
    svObjectlist.Add(svObject.ID());
    svObject.Name(Variablename);   
    svObject.ValueType(ivtString);
    svObject.ValueSubType(istChar8859);
    svObject.DPInfo("Die letzen 20 Fehlercodes der Heizung");
    svObject.Internal(false);
    svObject.Visible(true);
    dom.RTUpdate(0);
}

if (Ergebnis.ToLatin() != svObject.Value().ToLatin()) { svObject.State(Ergebnis); }
```
    

## 11.4 ioBroker ##  

*Die ioBroker-Beispiele stammen vom FHEM-Forumsmitglied „Thomas\_B".  
Vielen Dank!*

***Werte abrufen und anzeigen (exemplarisch ‚Warmwasser
Solltemperatur'):***  
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro1.jpg">
        
Unter ‚ioBroker Admin → Adapter' eine ‚Parser'-Instanz hinzufügen:
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro2.jpg">
    
Danach unter ‚ioBroker Admin → Instanzen' die hinzugefügten
Adapterinstanz ‚parser.0' zum Konfigurieren öffnen:
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro3.jpg">
    
    
Dort auf das ‚+' klicken, danach unter ‚Namen' den Namen
‚TWW\_Nennsollwert' vergeben. Unter ‚URL oder Dateiname' die IP des
BSB-LAN-Adapters samt Parameternummer angeben. Anschließend auf das
‚Bearbeiten'-Icon klicken.

<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro4.jpg">
    
Es öffnet sich die Eingabemaske, in der unter ‚RegEx' folgende
Zeichenfolge eingegeben werden muß:

`asser\s+-\s+TWW Nennsollwert\:\s+(\d{2}.\d)`
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro5.jpg">
    
    
Danach kann die Eingabemaske mit ‚Speichern' geschlossen werden.
    
Das Abfrageintervall kann man nach Bedarf einstellen, danach auf
‚Speichern und Schließen' klicken. Damit ist die Adapterkonfiguration
abgeschlossen.
    
Unter ‚ioBroker Admin → Objekte' findet sich nun der Ordner ‚parser.0'
und die unter der Instanz ‚parser.0' angelegten Datennamen und deren
Werte:

    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro6.jpg">
    

Die Werte können unter VIS mittels eines ‚Basic-Number'-Widgets mit
folgenden Einstellungen angezeigt werden:

    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro7.jpg">
    

Widgetcode zum Importieren:
    
```
[{"tpl":"tplValueFloat","data":{"oid":"parser.0.TWW_Nennsollwert","visibility-cond":"==","visibility-val":1,"is_comma":"true","factor":"1","html_append_singular":" ºC","html_append_plural":" ºC","gestures-offsetX":0,"gestures-offsetY":0,"is_tdp":false,"signals-cond-0":"==","signals-val-0":true,"signals-icon-0":"/vis.0/bluefox_ehome/lowbattery.png","signals-icon-size-0":0,"signals-blink-0":false,"signals-horz-0":0,"signals-vert-0":0,"signals-hide-edit-0":false,"signals-cond-1":"==","signals-val-1":true,"signals-icon-1":"/vis.0/bluefox_ehome/lowbattery.png","signals-icon-size-1":0,"signals-blink-1":false,"signals-horz-1":0,"signals-vert-1":0,"signals-hide-edit-1":false,"signals-cond-2":"==","signals-val-2":true,"signals-icon-2":"/vis.0/bluefox_ehome/lowbattery.png","signals-icon-size-2":0,"signals-blink-2":false,"signals-horz-2":0,"signals-vert-2":0,"signals-hide-edit-2":false,"visibility-groups-action":"hide","lc-type":"last-change","lc-is-interval":true,"lc-is-moment":false,"lc-format":"","lc-position-vert":"top","lc-position-horz":"right","lc-offset-vert":0,"lc-offset-horz":0,"lc-font-size":"12px","lc-font-family":"","lc-font-style":"","lc-bkg-color":"","lc-color":"","lc-border-width":"0","lc-border-style":"","lc-border-color":"","lc-border-radius":10,"lc-zindex":0,"digits":"1"},"style":{"left":"398px","top":"428px","width":"59px","height":"18px","color":"white","text-align":"right","font-family":"Arial, Helvetica, sans-serif","font-style":"normal","font-variant":"normal","font-weight":"bold","font-size":"","z-index":"20"},"widgetSet":"basic"}]
```
    

***Schalter anlegen (exemplarisch ‚Heizung Automatik (AN)'-Schalter):***

    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro8.jpg">
    

Zunächst ein leeres Script ‚Heizung Automatik Schalter' anlegen:
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro9.jpg">
    

Dann ein Blocky-Script ‚Heizung Automatik script' mit folgendem Inhalt
anlegen (der nachfolgende Code kann in Blocky importiert werden):

    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro10.jpg">
    
    
```
on({id: "javascript.0.scriptEnabled.Heizung_Automatik_Schalter", change: "ne"}, function (obj) {
  var value = obj.state.val;
  var oldValue = obj.oldState.val;
  for (var count = 0; count < 5; count++) {
    try {
      require("request")('http://<IP des BSB-LAN Adapters>/S700=1').on("error", function (e) {console.error(e);});
    } catch (e) { console.error(e); }
    console.log("request: " + 'http:// ://<IP des BSB-LAN Adapters>//S700=1');
  }
});
```
    
Dann ein ‚jqui -- Button State'-Widget in VIS anlegen:
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro11.jpg">
    
In den Eigenschaften unter ‚Schalter' das anfangs angelegte leere Script
angeben:  
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro12.jpg">
    
Damit lässt sich die Betriebsart ‚Heizung Automatik' einschalten.

Damit der Schalterzustand durch <img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro_button_red.jpg">
 oder 
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro_button_green.jpg">
 entsprechend visualisiert wird, müssen
noch folgende Signalbilder in dem Widget hinzugefügt werden, wobei die
Bilder „on.png" und „off.png" in dem genannten Verzeichnispfad
abgespeichert werden müssen.

    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro13.jpg">
    
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro14.jpg">
    

Widgetcode zum Importieren:
    
```
[{"tpl":"tplJquiButtonState","data":{"oid":"javascript.0.scriptEnabled.Heizung_Automatik_Schalter(2)","g_fixed":false,"g_visibility":false,"g_css_font_text":true,"g_css_background":true,"g_css_shadow_padding":false,"g_css_border":false,"g_gestures":false,"g_signals":true,"g_last_change":false,"visibility-cond":"==","visibility-val":1,"visibility-groups-action":"hide","buttontext":"Heizung Automatik (AN)","signals-cond-0":"==","signals-val-0":"Betriebsart: 1 - Automatik","signals-icon-0":"/vis.0/main/img/on.png","signals-icon-size-0":"33","signals-blink-0":false,"signals-horz-0":"-1","signals-vert-0":"6","signals-hide-edit-0":false,"signals-cond-1":"!=","signals-val-1":"Betriebsart: 1 – Automatik","signals-icon-1":"/vis.0/main/img/off.png","signals-icon-size-1":"20","signals-blink-1":false,"signals-horz-1":"2","signals-vert-1":"17","signals-hide-edit-1":false,"signals-cond-2":"==","signals-val-2":true,"signals-icon-2":"/vis/signals/lowbattery.png","signals-icon-size-2":0,"signals-blink-2":false,"signals-horz-2":0,"signals-vert-2":0,"signals-hide-edit-2":false,"lc-type":"last-change","lc-is-interval":true,"lc-is-moment":false,"lc-format":"","lc-position-vert":"top","lc-position-horz":"right","lc-offset-vert":0,"lc-offset-horz":0,"lc-font-size":"12px","lc-font-family":"","lc-font-style":"","lc-bkg-color":"","lc-color":"","lc-border-width":"0","lc-border-style":"","lc-border-color":"","lc-border-radius":10,"lc-zindex":0,"value":"","signals-oid-1":"parser.0.Betriebsart","signals-oid-0":"parser.0.Betriebsart"},"style":{"left":"14px","top":"426px","width":"219px","height":"27px","z-index":"1","font-size":"x-small"},"widgetSet":"jqui"}]

```
    
Die Einbindung der jeweiligen Werte bei ‚Objekt ID \[0\]' und ‚Objekt ID
\[1\]' (‚parser.0.Betriebsart') wird nachfolgend erklärt.

***Abfrage für die Betriebsart (für die On/Off-Visualisierung des
Heizungsschalters):***

Bei der Adapterkonfiguration für ‚parser.0' eine Regel mit der
Bezeichung ‚Betriebsart' erstellen, dann die IP (samt Parameternummer)
des BSB-LAN-Adapters angeben und zum Bearbeiten öffnen.
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro15.jpg">
    

Unter ‚RegEx' folgende Syntax angeben:

`(\w+:\s+\d\s+-\s+\w+)`

    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/ioBro16.jpg">
    
    
    
     
     
[Weiter zu Kapitel 12](kap12.md)      
[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  
