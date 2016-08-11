
Wetterdöschen
===================
----------
Unsere senseBox enthält die mitgelieferten Sensoren für Temperatur/Luftfeuchtigkeit und UV-Strahlung sowieso einen Sensor für ein Gemisch aus diversen Gasen, der nicht im Lieferumfang enthalten war.

Materialien
-----------
----------
**Aus der senseBox:edu**
-	Genuino UNO
-	Ethernet-Shield
-	HDC1000 (Temperatur&Luftfeuchtigkeitssensor)
-	VEML6070 (UV-Lichtsensor)

**Zusätzliche Hardware**
-	MQ 5 (Gas-Sensor)
-	diverse Kabel
-	Ethernet-Kabel
-	Power-over-Ethernet-Adapter
-	Ventilator

Setup Beschreibung
-----------
----------
**Hardwarekonfiguration**

Die Sensoren befinden sich „lose“ in der Box. Da auf dem senseBox-Shield nicht genug Steckplätze für alle drei Sensoren vorhanden waren, haben wir mehrere Kabel zusammengelötet. 
Die Stromversorung erfolgt mithilfe eines Power-over-Ethernet-Adapters über das LAN-Kabel.
Die Sensoren liegen bei uns alle innerhalb der lichtdurchlässigen Box. Da diese ohnehin nur schattig platziert werden kann, sind die Werte des UV-Sensors zu vernachlässigen. Da der Ventilator die Umgebungsluft in die Box leitet, finden die Sensoren für Temperatur/Luftfeuchtigkeit und Gas sehr ähnliche Bedingungen vor wie außerhalb der Box.

**Softwaresketch**

Der Softwaresketch basiert auf einer von OpenSenseMap bereitgestellten Schablone.
Die Sensoren liefern nicht alle direkt den gewünschten Messwert, daher muss im Code eine Umrechnung stattfinden. 
Den Großteil an Code konnte man jedoch von den jeweiligen Websites der Sensoren kopieren und unter Umständen mit kleinen Anpassungen übernehmen.



```
#include <SPI.h>
#include <Ethernet.h>
/*
 * Zusätzliche Sensorbibliotheken, -Variablen etc im Folgenden einfügen.
 */
 
//SenseBox ID
#define SENSEBOX_ID "573d7649566b8d3c11114ac2"

//Sensor IDs
#define TEMPSENSOR_ID "573d7649566b8d3c11114ac6"
#define SENSOR1_ID "573d7649566b8d3c11114ac5" // Luftfeuchtigkeit 
#define UVSENSOR_ID "573d7649566b8d3c11114ac4"

//Ethernet-Parameter
char server[] = "www.opensensemap.org";
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
// Diese IP Adresse nutzen falls DHCP nicht möglich
IPAddress myIP(192, 168, 0, 42);
EthernetClient client;

//Messparameter
int postInterval = 10000; //Uploadintervall in Millisekunden
long oldTime = 0;


void setup()
{
  Serial.begin(9600); 
  Serial.print("Starting network...");
  //Ethernet Verbindung mit DHCP ausführen..
  if (Ethernet.begin(mac) == 0) 
  {
    Serial.println("DHCP failed!");
    //Falls DHCP fehltschlägt, mit manueller IP versuchen
    Ethernet.begin(mac, myIP);
  }
  Serial.println("done!");
  delay(1000);
  Serial.println("Starting loop.");
}

void loop()
{
  //Upload der Daten mit konstanter Frequenz
  if (millis() - oldTime >= postInterval)
  {
    oldTime = millis();
    /*
     * Hier Sensoren auslesen und nacheinerander über postFloatValue(...) hochladen. Beispiel:
     * 
     * float temperature = sensor.readTemperature();
     * postFloatValue(temperature, 1, temperatureSensorID);
     */ 
  }
}

void postFloatValue(float measurement, int digits, String sensorId)
{ 
  //Float zu String konvertieren
  char obs[10]; 
  dtostrf(measurement, 5, digits, obs);
  //Json erstellen
  String jsonValue = "{\"value\":"; 
  jsonValue += obs; 
  jsonValue += "}";  
  //Mit OSeM Server verbinden und POST Operation durchführen
  Serial.println("-------------------------------------"); 
  Serial.print("Connectingto OSeM Server..."); 
  if (client.connect(server, 8000)) 
  {
    Serial.println("connected!");
    Serial.println("-------------------------------------");     
    //HTTP Header aufbauen
    client.print("POST /boxes/");client.print(SENSEBOX_ID);client.print("/");client.print(sensorId);client.println(" HTTP/1.1");
    client.println("Host: www.opensensemap.org"); 
    client.println("Content-Type: application/json"); 
    client.println("Connection: close");  
    client.print("Content-Length: ");client.println(jsonValue.length()); 
    client.println(); 
    //Daten senden
    client.println(jsonValue);
  }else 
  {
    Serial.println("failed!");
    Serial.println("-------------------------------------"); 
  }
  //Antwort von Server im seriellen Monitor anzeigen
  waitForServerResponse();
}

void waitForServerResponse()
{ 
  //Ankommende Bytes ausgeben
  boolean repeat = true; 
  do{ 
    if (client.available()) 
    { 
      char c = client.read();
      Serial.print(c); 
    } 
    //Verbindung beenden 
    if (!client.connected()) 
    {
      Serial.println();
      Serial.println("--------------"); 
      Serial.println("Disconnecting.");
      Serial.println("--------------"); 
      client.stop(); 
      repeat = false; 
    } 
  }while (repeat);
}
```




OpenSenseMap Registrierung
-----------
----------
Unsere Box wurde bei OpenSenseMap unter dem Namen „Wetterdöschen“ registriert. Dort haben wir 4 Sensoren angegeben (da einer der Sensoren zwei unterschiedliche Werte misst):
•  UV-Intensität(µW/cm²) 
•  Temperatur(°C)
•  Luftfeuchtigkeit(%)
•  Gas (H2, LPG, CH4, CO, Alkohol)

Stationsaufbau
-----------
----------
Die Station steht auf einem schattigen Balkon in MS-Gievenbeck.
![enter image description here](https://pp.vk.me/c631526/v631526290/40dd1/DW9qx2RqYhk.jpg)

Kontakt
-----------
----------
[Daniela Heines](daniela.heines@uni-muenster.de)
 [Anna Formaniuk](a_form03@uni-muenster.de)



