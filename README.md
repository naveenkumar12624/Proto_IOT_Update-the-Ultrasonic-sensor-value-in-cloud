Update-the-Ultrasonic-sensor-value-in-Things Mate

# AIM:
To upload the Ultrasonic sensor value in the Things mate using Arduino controller.

# Apparatus required:
Arduino Controller  .
Indoor gateway.
LoRaWAN shield .
HC-SR04 Ultrasonic sensor module .
Power supply .
Connecting wires .
Bread board .

# PROCEDURE:
### Procedure for gateway setup
	Go to the link http://172.31.255.254:8000 .
	Type the user name password .
	Go to LoRa and set the frequency plan 865 Mhz .
	In LoRa WAN configurations enter the Gate EUI and server address .
	Enable the Internet .
	Select the Wifi access point .
	Type the ssid and password in wifi LAN setting .
	Select the internet and IoT service and provide the details.
	Check all the green colour tick marks in gate way and proceed
### Procedure for gateway registration in The thingsMate LoRaWAN Management .
	Login https://iot.saveetha.in:4433 and provide the user id and password .
	Go to overview and select application server .
	Go to gateway and add new gateway .
	Enter the gateway id, name, EUI and select frequency plan .
	Open Arduino IDE and type the program for the given application .
	Compile the program if no error uploads the program in the controller .
	Go to gateways in things mate and check the live data .
	Create channel by giving channel name and ID .
	Add end device and enter the frequency plan, DevEUI, AppEUI, APP key, Select LoRaWAN version .
	Enter payload formatters .
	Go to the option query and give the new query name .
	Go to the option Dashboard and verify the output..  

# THEORY:

### What is IoT?

Internet of Things (IoT) describes an emerging trend where a large number of embedded devices (things) are connected to the Internet. These connected devices communicate with people and other things and often provide sensor data to cloud storage and cloud computing resources where the data is processed and analyzed to gain important insights. Cheap cloud computing power and increased device connectivity is enabling this trend.IoT solutions are built for many vertical applications such as environmental monitoring and control, health monitoring, vehicle fleet monitoring, industrial monitoring and control, and home automation

![image](https://user-images.githubusercontent.com/71547910/235334044-c01d4261-d46f-4f62-b07f-72a7b6fce5d5.png)

### What is LoRaWAN
The LoRaWAN® specification is a Low Power, Wide Area (LPWA) networking protocol designed to wirelessly connect battery operated ‘things’ to the internet in regional, national or global networks, and targets key Internet of Things (IoT) requirements such as bi-directional communication, end-to-end security, mobility and localization services.LoRaWAN® network architecture is deployed in a star-of-stars topology in which gateways relay messages between end-devices and a central network server. The gateways are connected to the network server via standard IP connections and act as a transparent bridge, simply converting RF packets to IP packets and vice versa. The wireless communication takes advantage of the Long Range characteristics of the LoRaÒ physical layer, allowing a single-hop link between the end-device and one or many gateways. All modes are capable of bi-directional communication, and there is support for multicast addressing groups to make efficient use of spectrum during tasks such as Firmware Over-The-Air (FOTA) upgrades or other mass distribution messages.

The specification defines the device-to-infrastructure (LoRa®) physical layer parameters & (LoRaWAN®) protocol and so provides seamless interoperability between manufacturers, as demonstrated via the device certification program.While the specification defines the technical implementation, it does not define any commercial model or type of deployment (public, shared, private, enterprise) and so offers the industry the freedom to innovate and differentiate how it is used.The LoRaWAN® specification is developed and maintained by the LoRa Alliance®: an open association of collaborating members.
![image](https://github.com/anishkumar-Embedded/Update-the-Ultrasonic-sensor-value-in-cloud/assets/71547910/c63c4edd-3b95-4a69-b9c2-4862afb335c3)

### Characteristics of LoRaWAN technology
Long range communication up to 10 miles in line of sight.
Long battery duration of up to 10 years. For enhanced battery life, you can operate your devices in class A or class B mode, which requires increased downlink latency.
Low cost for devices and maintenance.
License-free radio spectrum but region-specific regulations apply.
Low power but has a limited payload size of 51 bytes to 241 bytes depending on the data rate. The data rate can be 0,3 Kbit/s – 27 Kbit/s data rate with a 222 maximal payload size.

# PROGRAM:
```c
#include <SoftwareSerial.h>
#include <Adafruit_Sensor.h>

#define triggerpin 8
#define echopin 9
int duration, inches, cm;
String inputString = "";
bool stringComplete = false;
long old_time = millis();
long new_time;
long uplink_interval = 30000;
bool time_to_at_recvb = false;
bool get_LA66_data_status = false;
bool network_joined_status = false;
SoftwareSerial ss(10, 11);
char rxbuff[128];
uint8_t rxbuff_index = 0;

void setup() {
  pinMode(triggerpin, OUTPUT);
  pinMode(echopin, INPUT);
  Serial.begin(9600);
  ss.begin(9600);
  ss.listen();
  inputString.reserve(200);
  dht.begin();
  sensor_t sensor;
  dht.temperature().getSensor(&sensor);
  dht.humidity().getSensor(&sensor);
  ss.println("ATZ"); //reset LA66
}

void loop() {
  new_time = millis();
  if ((new_time - old_time >= uplink_interval) && (network_joined_status == 1)) {
    old_time = new_time;
    get_LA66_data_status = false;
    HC04();
    char sensor_data_buff[128] = "\0";
    snprintf(sensor_data_buff, 128, "AT+SENDB=%d,%d,%d,%02X%02X", 0, 2, 2, (short)(inches), (short)(cm));
    ss.println(sensor_data_buff);
  }

  if (time_to_at_recvb == true) {
    time_to_at_recvb = false;
    get_LA66_data_status = true;
    delay(1000);
    ss.println("AT+CFG");
  }

  while (ss.available()) {
    char inChar = (char)ss.read();
    inputString += inChar;
    rxbuff[rxbuff_index++] = inChar;
    if (rxbuff_index > 128)
      break;

    if (inChar == '\n' || inChar == '\r') {
      stringComplete = true;
      rxbuff[rxbuff_index] = '\0';
      if (strncmp(rxbuff, "JOINED", 6) == 0) {
        network_joined_status = 1;
      }
      if (strncmp(rxbuff, "Dragino LA66 Device", 19) == 0) {
        network_joined_status = 0;
      }
      if (strncmp(rxbuff, "Run AT+RECVB=? to see detail", 28) == 0) {
        time_to_at_recvb = true;
        stringComplete = false;
        inputString = "\0";
      }
      if (strncmp(rxbuff, "AT+RECVB=", 9) == 0) {
        stringComplete = false;
        inputString = "\0";
        Serial.print("\r\nGet downlink data(FPort & Payload) ");
        Serial.println(&rxbuff[9]);
      }
      rxbuff_index = 0;

      if (get_LA66_data_status == true) {
        stringComplete = false;
        inputString = "\0";
      }
    }
  }

  while (Serial.available()) {
    char inChar = (char)Serial.read();
    inputString += inChar;
    if (inChar == '\n' || inChar == '\r') {
      ss.print(inputString);
      inputString = "\0";
    }
  }

  if (stringComplete) {
    Serial.print(inputString);
    inputString = "\0";
    stringComplete = false;
  }
}

void HC04() {
  digitalWrite(triggerpin, LOW);
  delayMicroseconds(2);
  digitalWrite(triggerpin, HIGH);
  delayMicroseconds(10);
  digitalWrite(triggerpin, LOW);
  duration = pulseIn(echopin, HIGH);
  inches = microsecondsToInches(duration);
  cm = microsecondsToCentimeters(duration);
  Serial.print(inches);
  Serial.print("in, ");
  Serial.print(cm);
  Serial.print("cm");
  Serial.println();
}

long microsecondsToInches(long microseconds) {
  return microseconds / 74 / 2;
}

long microsecondsToCentimeters(long microseconds) {
  return microseconds / 29 / 2;
}
```
# CIRCUIT DIAGRAM:
![image](https://github.com/VarshaAjith1110/Update-the-Ultrasonic-sensor-value-in-cloud/assets/94222288/5ab9f2ed-e35f-406d-bc48-a957d4a217e9)

# OUTPUT:
![image](https://github.com/VarshaAjith1110/Update-the-Ultrasonic-sensor-value-in-cloud/assets/94222288/6ca91d2a-283c-4d70-9b06-09f9bb3b6e9f)

# RESULT:
Thus the Ultrasonic sensor value is uploaded in the Things mate using Arduino controller.
