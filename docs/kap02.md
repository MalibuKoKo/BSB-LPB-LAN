[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  
[Zurück zu Kapitel 1](kap01.md)  
    

# 2. Grundsätzliches zum BSB, LPB und zur PPS-Schnittstelle #

## 2.1 BSB und LPB

BSB (Boiler System Bus) und LPB (Local Process Bus) sind zwei
verschiedene Bus-Typen, die sich vereinfacht in zwei Nutzungsklassen
unterscheiden lassen:

1. BSB ist ein ‚lokaler' Bus zur Nutzung des lokal angeschlossenen
Reglers. Angeschlossene Geräte wie bspw. die Bedieneinheit, ein
Raumgerät / Fernbedienung oder auch der (via BSB) angeschlossene Adapter
können nur auf den Regler zugreifen, an dem sie angeschlossen sind.

2. LPB ist ein ‚übergreifender' Bus zur Nutzung mehrerer angeschlossener
Regler in einem komunikationsfähigen Verbund.
Über den LPB können mehrere Regler miteinander verbunden werden und bei
korrekter Parametrierung (Stichwort Geräte- und Segmentadressen) gewisse
Werte/Einstellungen/Parameter miteinander teilen bzw. sich gegenseitig
beeinflussen.
Auf diese Weise kann bspw. eine Kaskadenschaltung von mehreren Brennern
realisiert werden oder eine Gas- oder Öl-Heizung mit einer Solaranlage
und einem Feststoffkessel regelungstechnisch \'verbunden\' werden.\
Der korrekte Anschluss der einzelnen Komponenten sowie die korrekte
Parametrierung der jeweiligen Regler sollte im Normalfall bereits bei
der Installation der Anlage durch den Heizungsinstallateur erfolgt
sein.

***Beispiel:***  
*Vorhanden sind eine Öl- oder Gasheizung, ein nachgerüsteter
wasserführender Kamin und eine thermische Solaranlage zur Unterstützung
des Heizkreises oder der Warmwasserbereitung.
Alle drei Wärmeerzeuger sind hydraulisch an einem Pufferspeicher
angeschlossen.
Die Wärme für den Heizkreis soll vom Pufferspeicher bezogen werden.*

*Die Regelung der Solaranlage und des Feststoffkessels übernimmt ein
Solarsystemregler (SSR), die Kesselsteuerung der Heizung übernimmt in
diesem Beispiel der interne Heizungsregler. Alle Sensoren, Pumpen,
Mischer etc. sind am SSR angeschlossen, welcher jedoch via LPB mit dem
Heizungsregler verbunden ist. Durch diese Verbindung der beiden Regler
kann somit bspw. eine Pufferspeicherladung geregelt werden, bei der die
Heizung nur aktiv wird, wenn weder Solar noch Feststoffkessel den Puffer
laden / geladen haben.*

*Wenn ein Adapter via BSB an einem der beiden Regler aus oben genanntem
Beispiel angeschlossen ist, kann er folglich nur auf den jeweiligen
Regler \'lokal\' zugreifen, an dem er angeschlossen ist (also bspw.
Heizungsregler oder SSR). Ebenso verhält es sich mit den jeweiligen
Bedieneinheiten der Regler, die über den ‚Bus BE' (Bus Bedieneinheit)
angeschlossen sind.
Wenn ein Adapter via LPB an einem der beiden Regler aus oben genanntem
Beispiel angeschlossen ist, müssen
1. die Geräte- und Segmentadressen entsprechend der
LPB-Konfigurationsanforderungen eingestellt werden, und
2. beim Adapter eine Zieladresse eingestellt werden, an die die
jeweiligen Anfragen des Adapters geschickt werden.*    

Die spezifischen technischen Daten, Leistungsmerkmale und Anforderungen
an entsprechende Installationen und Parametrierungen hinsichtlich der
Geräte- und Segmentadressen sind den jeweiligen technischen
Dokumentationen der Hersteller zu entnehmen. Hinsichtlich des LPB seien
insbesondere die Dokumentationen „LPB Systemgrundlagen"⁷ und „LPB
Projektierungsgrundlagen"⁸ empfohlen.

Bei einigen Reglern sind die entsprechenden Anschlüsse teilweise
unterschiedlich gekennzeichnet:

-   Der BSB ist nicht auf allen Reglern als solcher bezeichnet, weitere
    Bezeichnungen sind „FB" (Fernbedienung) sowie „CL+" und „CL-".

    Der zusätzliche Anschluss „G+" führt 12V und ist für die
    Hintergrundbeleuchtung der entsprechenden Raumgeräte vorgesehen.
    Dieser ist für den Anschluss des Adapters NICHT zu verwenden!\
    (Sollte der Adapter irrtümlicherweise an G+ statt an CL+
    angeschlossen werden, so leuchtet zwar die LED, allerdings ist
    keinerlei Funktion gegeben.)

-   Der LPB ist bei einigen Reglern mit „DB"(+) und „MB"(-)
    gekennzeichnet.

Die folgenden Abbildungen zeigen exemplarisch die verschiedenen
Anschlüsse.  
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/bsb-lpb-anschluss.jpg">

BSB & FB (CL+/CL-) und LPB (DB/MB) bei einem Brötje ISR-RVS43.222-Regler.  
    
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/bsb-lpb-anschluss-2.jpg">
    
b = BSB (CL+/CL-) und a = LPB (DB/MB) bei einem Siemens RVS63.283-Regler.  
    
    
<img src="https://raw.githubusercontent.com/1coderookie/BSB-LPB-LAN/master/docs/pics/bsb-servicebuchse.jpg">
    
BSB (CL+/CL-) an der vierpoligen Servicebuchse vorne in der Bedieneinheit eines ISR Plus
→ Die (dauerhafte) Verwendung dieses Anschlusses ist jedoch nicht zu empfehlen.  
    
    

***Bei Anschluss des Adapters sollte der betreffende Regler stets
ausgeschaltet sein, ebenso bei einem Entfernen des Adapters.***

***Es ist unbedingt darauf zu achten, dass der Regler polrichtig
angeschlossen wird!
Ein verkehrter Anschluss kann eine Beschädigung des Reglers und/oder
Adapters zur Folge haben!***

Die entsprechende Polung bzw. Bezeichnung der Anschlüsse ist auf der
Adapterplatine gekennzeichnet. Bei einem Nachbau ist der Schaltplan zu
beachten.

Wenn mehrere Regler (bspw. wie im obigen Beispiel) vorhanden sind,
bietet es sich derzeit noch an, pro Regler jeweils einen Adapter via BSB
anzuschließen, um den jeweiligen Zugriff zu realisieren. Eine
übergreifende Abfrage von Werten oder Parametern zweier oder mehrerer
Regler im LPB-Verbund via Adapter kann mittlerweile zwar erfolgen, doch
ist diese Funktion noch nicht ausgiebig getestet worden. Um mit einem
Adapter via LPB auf verschiedene Regler zuzugreifen, ist die jeweilige
Angabe der reglerspezifischen Geräteadresse als Zieladresse nötig (siehe
Kapitel 8. URL-Befehle und Spezialfunktionen). Alle Geräte (Regler und
Adapter) müssen sich dabei im selben Segment befinden und grundsätzlich
gemäß den LPB-Projektierungsgrundlagen konfiguriert sein.

Gewisse Funktionen scheinen im Übrigen nicht via LPB unterstützt zu
werden, bspw. das Senden einer Raumtemperatur an einen Regler, da diese
Information vom Regler auf dem BSB erwartet wird. Da die Software
ursprünglich für die Nutzung des Adapters via BSB geschrieben und erst
nachträglich von Frederik um die LPB-Funktionalität erweitert wurde, ist
es außerdem möglich, dass via LPB nicht alle Parameter verfügbar sind,
die via BSB verfügbar wären. Die Software wird zwar stetig weiter
entwickelt, derzeit bietet es sich jedoch noch an, als Anschluss den BSB
zu präferieren, wenn dieser vorhanden ist.

***Tipps:***  

Um vor Störeinflüssen möglichst geschützt zu sein, sollten die
Anschlusskabel für den LPB-Anschluss gemäß
LPB-Projektierungsgrundlagen einen Querschnitt von 1,5mm² aufweisen,
zweiadrig verdrillt und geschirmt sein (Leitungslänge max. 250m pro
Busteilnehmer, max. Gesamtlänge 1000m).  
Für den BSB-Anschluss sind Cu-Leitungen mit mindestens 0,8mm² (bis 20m)
Querschnitt zu wählen, bspw. LIYY oder LiYCY 2 x 0,8. Bei Leitungslängen
bis 80m sollte 1mm², bis 120m sollten 1,5mm² Querschnitt gewählt
werden⁹.  
Generell ist eine parallele Verlegung mit Netzleitungen zu vermeiden
(Störsignale), geschirmte Leitungen sind ungeschirmten Leitungen immer
vorzuziehen.

Der Anschluss der Leitungen an die jeweiligen Kontakte sollte
prinzipiell immer mit den spezifischen Steckern erfolgen.  
Sollten diese nicht unmittelbar erhältlich oder verfügbar sein, können
auch isolierte 6,3mm-Kabelschuhe verwendet werden.  

---
⁷ Siemens Building Technologies - Landis & Staefa Division: CE1N2030D  
⁸ Siemens Building Technologies - Landis & Staefa Division: CE1N2032D  
⁹ Siehe „Brötje Montageanleitung für Raumgerät RGT/RGTK"  

---

## 2.2 PPS-Schnittstelle ##

Die PPS-Schnittstelle findet sich bei älteren Reglern und stellt eine
Punkt-zu-Punkt-Schnittstelle dar, mittels derer
Bedieneinheiten/Raumgeräte wie das QAA70[^11] angeschlossen werden
können. An demjenigen Anschluss wird analog zum QAA auch der Adapter
angeschlossen.

PPS scheint bei folgenden Reglern zum Einsatz gekommen zu sein[^12]: RVP
digital Serie D, RVP54..., ALBATROS RVA..., LGM11...; bzw. u.a. bei
folgenden Heizungen: Brötje WGB 15 / WGB 20, Weishaupt WRD 0.2 / 1.1,
Sieger TG11 (mit Siegermatic S42DB), Olymp THR 5-25C, Schäfer Interdomo
(mit DomoCommand DC 225).

Die beiden Geräte (Raumgerät und Regler) sprechen nur bedingt
miteinander. Der Regler sendet Infos, schickt dann später mit einem
einzigen Byte (0x17) eine Anforderung an das Raumgerät, das dann
teilweise auf vorhergehende Regler-Infos reagiert, andererseits aber
auch nach eigenem Rhythmus seine Infos sendet. Und das teilweise in
unterschiedlicher Häufigkeit.
Der Bus kommt so kaum zur Ruhe, i.d.R. werden bis zu zwei Telegramme pro
Sekunde ausgetauscht, entsprechend schnell muss die Software dann auch
antworten. Kommt auf bestimmte Anfragen des Reglers keine oder nicht die
richtige Antwort, wird angenommen, dass es kein Raumgerät mehr gibt und
der Regler verfällt wieder in einen Suchmodus.

Der Funktionsumfang ist hierbei nur rudimentär und beschränkt sich
derzeit mittels BSB-LAN auf etwa ein Dutzend Parameter, die man
lesen/schreiben kann:

- Raumtemperatur Ist  
- Raumtemperatur Soll  
- Außentemperatur (read-only)  
- Außentemperatur gemischt (read-only)  
- Position Drehknopf  
- Kesselvorlauftemperatur (read-only)  
- Mischervorlauftemperatur (read-only)  
- Status Trinkwasserbetrieb (read-only)  
- Trinkwassertemperatur Ist (read-only)  
- Trinkwassertemperatur Soll  
- Betriebsart  
- Anwesenheit

Immerhin lassen sich damit aber die wichtigsten Funktionen einer
intelligenten Heizungssteuerung umsetzen, indem man z.B. gewichtete
Raumtemperaturen sendet und die Solltemperaturen nach vielfältigeren
Kriterien steuern kann.

***Hinweis:***  
Noch offen ist, ob wegen des Punkt-zu-Punkt-Designs ein QAA mit dem
Adapter friedlich koexistieren kann oder ob man sich am Ende für eine
Variante entscheiden muss.
    
     
     
[Weiter zu Kapitel 3](kap03.md)      
[Zurück zum Inhaltsverzeichnis](inhaltsverzeichnis.md)  

