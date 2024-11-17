# Weather_management_system
## Introduction
The goal of the project is to put theoritical knowhow into practise by building a system to monitor basic weather elements

like Temperature & Humidity and thus provide a well functioning product at the end.

## Components 
### Hardware
1. ESP8266 Nodemcu
2. Breadboard/ PCB board
3. DHT11 Sensor
4. INA219 Current sensor
5. Battery(9V)
6. LM2596 DC-DC Converter module
7. Connectors

### Software
1. Arduino IDE
2. TinkerCAD
3. Draw io
4. EasyEDA PCB designer
5. Thingspeak Server

## Block Diagram
Using Draw.io, the following diagram was created to represented the general flow of the project from the power to visualization.

![Weather_Management_System_Block_Diagram drawio](https://github.com/user-attachments/assets/1f557720-a989-4b56-a6fb-d17b6af42969)

## Simulation
Using TinkerCAD, an instance of the hardware was captured and used to view how the logic would be implemented in the real device.

![image](https://github.com/user-attachments/assets/7d9ae1eb-64db-41b4-99f2-e21cee22098f)

![image](https://github.com/user-attachments/assets/64060d42-bae4-424e-874e-4280bc332941)

## PCB Design

Using EasyEDA, the design was implemented starting with the schematics, PCB routing and finally a 3D visualization.

![image](https://github.com/user-attachments/assets/5155e6d5-e068-4991-b2f8-f7a1fd256cb6)

![image](https://github.com/user-attachments/assets/dc532b63-9ccf-44cd-b396-16691a051070)

![image](https://github.com/user-attachments/assets/846b37f1-f3bc-487b-8e21-eb0c24671312)

Unfortunately, the above design could not be able to be printed due to financial constraints, hence 

a cheaper alternative was sought.

This involved combining & soldering already available modules on a perf board.




## Firmware

An Arduino IDE was used to implement the firmware and logic.

```
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <ThingSpeak.h>
#include <Adafruit_INA219.h>

// Define DHT11 sensor pins
#define DHTPIN 12       // DHT11 data pin connected to GPIO 4 (D2 on NodeMCU)
#define DHTTYPE DHT11   // DHT11 sensor

// Wi-Fi credentials for ThingSpeak connection
const char* ssid = "Vodafone-F43C";
const char* password = "Ronaldo943";
unsigned long myChannelNumber = 2716164;
const char* myWriteAPIKey = "PS81VPVUW69EO1NP";

// Instantiate DHT sensor and INA219 sensor
DHT dht(DHTPIN, DHTTYPE);
WiFiClient client;
Adafruit_INA219 ina219;

// Sleep duration in microseconds (20 seconds)
const uint32_t SLEEP_DURATION_US = 20 * 1000000;

void setup() {
  Serial.begin(115200);

  // Initialize DHT11
  dht.begin();

  // Initialize INA219 sensor
  if (!ina219.begin()) {
    Serial.println("Failed to find INA219 chip");
    while (1) { delay(10); }
  }

  // Wi-Fi setup for ThingSpeak
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi");

  // Initialize ThingSpeak
  ThingSpeak.begin(client);

  // Take sensor readings and send to ThingSpeak
  takeSensorReadings();

  // Enter deep sleep
  Serial.println("Entering deep sleep for 20 seconds...");
  ESP.deepSleep(SLEEP_DURATION_US);
}

void loop() {
  // Empty loop as ESP8266 will wake up, run setup(), and sleep again
}

void takeSensorReadings() {
  // Read temperature and humidity
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // Read from INA219
  float busVoltage = ina219.getBusVoltage_V();
  float current_mA = ina219.getCurrent_mA();
  float power_W = busVoltage * (current_mA / 1000); // Power in Watts

  // Check if any reads failed
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Print readings to Serial monitor
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" °C");

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.print("Bus Voltage: ");
  Serial.print(busVoltage);
  Serial.println(" V");

  Serial.print("Current: ");
  Serial.print(current_mA);
  Serial.println(" mA");

  Serial.print("Power: ");
  Serial.print(power_W);
  Serial.println(" W");

  // Send data to ThingSpeak
  ThingSpeak.setField(1, temperature);
  ThingSpeak.setField(2, humidity);
  ThingSpeak.setField(3, busVoltage);
  ThingSpeak.setField(4, current_mA);
  ThingSpeak.setField(5, power_W);

  int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
  if (x == 200) {
    Serial.println("Data sent to ThingSpeak successfully");
  } else {
    Serial.println("Error sending data to ThingSpeak: " + String(x));
  }
}

```
## Visualization
Thingspeak  server platform was utilized for receiving data collected by the sensors, after which it could enable different </br>
forms of visualizations

## Conclusion







