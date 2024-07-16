# creative_minds

# SMART GAS SAFETY SYSTEM : DETECTING GAS LEAKS FOR SAFER HOMES AND INDUSTRIES

# Introduction
The history of gas leak detection systems dates back to the early 20th century when rudimentary methods, such as canaries in coal mines, were used to detect toxic gases. Over time, technology advanced, leading to the development of electronic sensors capable of detecting various hazardous gases. The introduction of microcontrollers and wireless communication modules further enhanced these systems, allowing for real-time monitoring and automated responses. The Smart Gas Safety System is a modern iteration of these advancements, designed to address the critical issue of detecting hazardous gas leaks, specifically carbon monoxide and methane, in industrial and residential settings. Using advanced MQ-4 and MQ-7 gas sensor and the ESP8266 microcontroller, the system provides timely detection and real-time alerts via the TWILIO API module. It also automates safety measures by activating ventilation systems with servo or stepper motors and employs a pneumatic solenoid valve for master shutdown, ensuring comprehensive protection against gas-related incidents.</p>

# Overview
The Smart Gas Safety System is a comprehensive solution designed to detect hazardous gas leaks, such as carbon monoxide and methane, in residential and industrial environments. Utilizing advanced MQ-4 and MQ-7 gas sensor, along with the ESP8266 microcontroller, the system ensures timely detection of gas leaks. Real-time alerts are sent through the TWILIO API module, notifying residents and authorities immediately. The system also automates safety responses by activating ventilation systems via servo or stepper motors and employing a pneumatic solenoid valve for a master shutdown to stop further gas leakage. This modern approach enhances safety by protecting occupants and property from potential gas-related incidents.</p>

# Components required with Bill of Materials
| ITEM | QUANTITY | DESCRIPTION | LINK |
|----------|----------|----------|----------|
| ESP8266  | 02  | NodeMCU wifi wireless module |  https://amzn.in/d/05MyWk6n   |
| V8 cable | 02 | cable for ESP8266 for powersupply | https://amzn.in/d/09nCeXKZ   |
| MQ-2 sensor | 01  | smoke detection sensor  |  https://amzn.in/d/0eJd0Ud8  |
| Battery | 02  | 9V lithium ion battry  |  https://amzn.in/d/0hs6br2Q  |
| jumper wires   | 1 set each | male to male & male to female & female to female   |  https://amzn.in/d/09yav9Pw  |
| Berg strip  | 04  | IC  base for ESP8266  |  https://amzn.in/d/06I7wmXf  |
| general purpose PCB | 01   | ___  |  https://amzn.in/d/00qFvVEb  |
| Servo Motor  | 01   | standard Servo motor |  https://amzn.in/d/05cKaQQ5  |
| Relay    | 01   | Electromagnetic switch |  https://amzn.in/d/05jAhE0O   |
| solenoid valve  | 01 | Electromagnetic actuator | https://amzn.in/d/07uVnz72  |
| adapter         | 01 | power supply for solenoid valve|  https://amzn.in/d/01LiqYgm

# TABLE FOR PIN CONECTIONS

## Publisher Module wiring 

| ESP8266(1) | MQ-2|
|----------|----------|
| A0   | A0 |
| VCC   | vin(3.3V)   |
| GND | GND   |

## Subscriber Module wiring 

| ESP8266(2) | Relay| Solenoid valve| Servomotor|
|----------|----------|----------|----------|
| Vin or 3.3V    | VCC  |     | VCC  |
| GND   | GND   |      | GND  |
| D1  | IN  |  |    |
|     | COM | External Power Supply  |    |
|    |  | +ve of Solenoid Valve  |   |
|D2  | NO       |          | Servosignal|

## BLOCK DIAGRAM
![Swe 1  (2)](https://github.com/Archanakattii/creative_minds/assets/160317292/5773471e-3293-4fda-ad28-b1e901ddd590)


# Working Code
The system is divided into two parts:</p>
1.Publisher module</p>
2.Subscriber module</p>
**Here MQTT acts as a broker</p>**

# **1.Publisher Module**

    #include <ESP8266WiFi.h>
    #include <PubSubClient.h>
    
    // WiFi credentials
    const char* ssid = "abcd";
    const char* password = "********";   //dummy wifi credientials
    
    // MQTT Broker
    const char* mqtt_server = "broker.hivemq.com";
    const char* topic = "home/gasLeak";
    
    // Pins
    #define GAS_SENSOR_PIN A0
    
    WiFiClient espClient;
    PubSubClient client(espClient);
    
    void setup() {
      Serial.begin(115200);
    
      // Connect to WiFi
      setup_wifi();
      
      // Initialize MQTT
      client.setServer(mqtt_server, 1883);
    
      // Wait for connection
      while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
      }
      Serial.println("WiFi connected");
    
      delay(100);
    }
    
    void setup_wifi() {
      delay(10);
      Serial.println();
      Serial.print("Connecting to ");
      Serial.println(ssid);
    
      WiFi.begin(ssid, password);
    
      while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
      }
    
      Serial.println("");
      Serial.println("WiFi connected");
      Serial.println("IP address: ");
      Serial.println(WiFi.localIP());
    }
    
    void reconnect() {
      while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");
        if (client.connect("ESP8266ClientPublisher")) {
          Serial.println("connected");
        } else {
          Serial.print("failed, rc=");
          Serial.print(client.state());
          Serial.println(" try again in 5 seconds");
          delay(5000);
        }
      }
    }
    
    void loop() {
      if (!client.connected()) {
        reconnect();
      }
      client.loop();
    
      int gasValue = analogRead(GAS_SENSOR_PIN);
      Serial.print("Gas Value: ");
      Serial.println(gasValue);
      
      if (gasValue > 350) { // Adjust threshold as necessary
        client.publish(topic, "leak");
      } else {
        client.publish(topic, "safe");
      }
    
      delay(2000); // Publish data every 2 seconds
    }
This code runs on an ESP8266 microcontroller to detect gas leaks and publish the sensor data to an MQTT broker. It connects to a specified WiFi network using the provided SSID and password, and sets up communication with an MQTT broker at "broker.hivemq.com" on the topic "home/gasLeak." The gas sensor is connected to the analog pin A0. </p>

In the `setup()` function, the code initializes serial communication, connects to the WiFi network, and sets the MQTT server. It waits for the WiFi connection to be established, indicated by printing dots in the serial monitor. The `setup_wifi()` function handles the WiFi connection process and prints the connection status and IP address once connected. </p>

The `reconnect()` function ensures that the ESP8266 reconnects to the MQTT broker if the connection is lost, printing the connection status to the serial monitor. In the `loop()` function, the code continuously checks the MQTT connection and reconnects if necessary. It reads the gas sensor value from the analog pin and prints it to the serial monitor. Based on the gas sensor value, it publishes a message to the MQTT topic indicating either "leak" if the value exceeds a threshold of 350, or "safe" if it does not. The data is published every 2 seconds.</p>



https://github.com/user-attachments/assets/9a082c30-6fcc-4c34-96f3-4564fb267448

# **2.Subscriber Module**

    #include <ESP8266WiFi.h>
    #include <ESP8266HTTPClient.h>
    #include <base64.h>
    #include <PubSubClient.h>
    #include <Servo.h>
    
    // WiFi credentials
    const char* ssid = "your id";
    const char* password = "your password";
    
    // MQTT Broker
    const char* mqtt_server = "broker.hivemq.com";
    const char* topic = "home/gasLeak";
    
    // Pins
    #define SERVO_PIN D0
    #define RELAY_PIN D1
    
    // Twilio credentials and phone numbers
    const char* account_sid = "twilio acc num";
    const char* auth_token = "twilio token num";
    const char* to_number = "your num";
    const char* from_number = "from num";
    const char* twilio_url = "https://api.twilio.com/2010-04-01/Accounts/Calls.json";
    const char* twiml_url = "http://demo.twilio.com/docs/voice.xml";
    
    WiFiClient espClient;
    WiFiClientSecure secureClient;
    PubSubClient client(espClient);
    Servo servo;
    
    void setup() {
      Serial.begin(115200);
    
      // Connect to WiFi
      setup_wifi();
      
      // Initialize MQTT
      client.setServer(mqtt_server, 1883);
      client.setCallback(callback);
    
      // Initialize Servo
      servo.attach(SERVO_PIN);
      servo.write(0); // Initial position
      
      // Initialize Relay
      pinMode(RELAY_PIN, OUTPUT);
      digitalWrite(RELAY_PIN, LOW); // Relay initially OFF
    
      delay(100);
    }
    
    void setup_wifi() {
      Serial.println();
      Serial.print("Connecting to ");
      Serial.println(ssid);
    
      WiFi.begin(ssid, password);
    
      while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
      }
    
      Serial.println("");
      Serial.println("WiFi connected");
      Serial.println("IP address: ");
      Serial.println(WiFi.localIP());
    }
    
    void reconnect() {
      while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");
        if (client.connect("ESP8266ClientSubscriber")) {
          Serial.println("connected");
          client.subscribe(topic);
        } else {
          Serial.print("failed, rc=");
          Serial.print(client.state());
          Serial.println(" try again in 5 seconds");
          delay(5000);
        }
      }
    }
    
    void callback(char* topic, byte* payload, unsigned int length) {
      String message;
      for (unsigned int i = 0; i < length; i++) {
        message += (char)payload[i];
      }
      
      Serial.print("Message arrived [");
      Serial.print(topic);
      Serial.print("]: ");
      Serial.println(message);
      
      if (message == "leak") {
        digitalWrite(RELAY_PIN, LOW); // Turn relay OFF
        servo.write(180); // Turn servo back to 0 degrees
        Serial.println("Gas leak detected! Relay OFF, Servo at 180 degrees.");
        makeTwilioCall();
      } else if (message == "safe") {
        digitalWrite(RELAY_PIN, HIGH); // Turn relay ON
        servo.write(0); // Turn servo to 180 degrees
        Serial.println("Gas leak safe! Relay ON, Servo at 0 degrees.");
      } else {
        Serial.println("Unknown message received.");
      }
    }
    
    void loop() {
      if (!client.connected()) {
        reconnect();
      }
      client.loop();
    }
    
    void makeTwilioCall() {
      secureClient.setInsecure();  // Use this for testing purposes only. In production, you should validate the server certificate.
      HTTPClient http;
      String auth = String(account_sid) + ":" + String(auth_token);
      
      // Declare the char array with the correct size
      char auth_char_array[auth.length() + 1];
      auth.toCharArray(auth_char_array, auth.length() + 1);
      
      String auth_base64 = base64::encode(auth_char_array);
    
      http.begin(secureClient, twilio_url);
      http.addHeader("Authorization", "Basic " + auth_base64);
      http.addHeader("Content-Type", "application/x-www-form-urlencoded");
    
      String postData = "Url=" + String(twiml_url) + "&To=" + String(to_number) + "&From=" + String(from_number);
      
      int httpCode = http.POST(postData);
      
      if (httpCode > 0) {
        String payload = http.getString();
        Serial.println("Response: " + payload);
      } else {
        Serial.println("Error on HTTP request: " + String(httpCode));
      }
    
      http.end();
    }
    
This code runs on an ESP8266 microcontroller to detect gas leaks, control a servo motor and relay based on the leak status, and make a phone call using the Twilio API when a leak is detected. It connects to a WiFi network using provided credentials and communicates with an MQTT broker at "broker.hivemq.com" on the topic "home/gasLeak." The servo motor is connected to pin D0, and the relay is connected to pin D1. The servo starts at 0 degrees, and the relay is initially off. </p>

The `setup()` function initializes serial communication, connects to WiFi using `setup_wifi()`, sets the MQTT server and callback function, and initializes the servo and relay. The `setup_wifi()` function handles the WiFi connection process and prints the connection status and IP address. The `reconnect()` function ensures that the ESP8266 reconnects to the MQTT broker if the connection is lost and subscribes to the topic.</p>

In the `loop()` function, the code continuously checks the MQTT connection and reconnects if necessary. The `callback()` function handles incoming MQTT messages. If a "leak" message is received, it turns off the relay, sets the servo to 180 degrees, and calls the `makeTwilioCall()` function to make a phone call using Twilio. If a "safe" message is received, it turns on the relay and sets the servo to 0 degrees. The `makeTwilioCall()` function makes an HTTP POST request to the Twilio API to initiate a phone call, using base64-encoded Twilio credentials for authentication.</p>      
       
            

https://github.com/user-attachments/assets/26c738e8-c740-447a-9639-aaf61b5604d2




    
# Demo Images
**Publisher module</p>**

Here we have used Smoke instead of LPG in the publisher module as LPG is hazardous and flammable,using approriate threshold LPG can be detected and for this detection we are using MQ-2 sensor for the prototype.</p>

![Screenshot 2024-07-16 201728](https://github.com/user-attachments/assets/7400a67f-c807-4549-89e5-a1e2017452b1)



![WhatsApp Image 2024-07-16 at 7 04 34 PM](https://github.com/user-attachments/assets/82695a3a-9632-4b24-baae-17721e2815ec)

**Subscriber module</p>**


![Screenshot 2024-07-16 201811](https://github.com/user-attachments/assets/5463c5a5-141f-4c8a-80a1-9cad722e8e10)


![WhatsApp Image 2024-07-16 at 7 05 19 PM](https://github.com/user-attachments/assets/3f154899-3b66-4e74-b88c-af6a5cd1168d)

![WhatsApp Image 2024-07-16 at 7 05 44 PM](https://github.com/user-attachments/assets/615ebd14-7fce-4900-8d91-0d814f69e6ac)

![WhatsApp Image 2024-07-16 at 7 06 01 PM](https://github.com/user-attachments/assets/462438a4-4b33-468d-a06d-97dac9285558)

# Application Video




https://github.com/user-attachments/assets/7d1e9e9c-80b6-4cb6-9896-724df87b0492



https://github.com/user-attachments/assets/b9ead745-7654-416f-88e1-5a23be8cafbd



The gas leak detection system prototype functions through a series of coordinated steps to ensure safety. The MQ-2 gas sensor continuously monitors the environment for hazardous gases, sending data to the ESP8266 microcontroller. Upon detecting a gas concentration above a set threshold, the ESP8266 processes this data and uses its Wi-Fi connectivity to publish a "leak" status message via the MQTT protocol to a broker. Automated safety measures are then triggered: a pneumatic solenoid valve shuts off the gas supply to prevent further leakage, while a servo motor opens windows for ventilation. The ESP8266 also makes HTTP requests to Twilio's API, triggering calls to alert occupants about the gas leak. For safety during testing, smoke is used instead of LPG gas. The system maintains continuous monitoring, adjusting the safety mechanisms and maintaining MQTT connectivity for real-time updates, thereby providing a comprehensive and proactive approach to gas leak detection and response.

## TWILIO API 
Twilio is used to automate the process of sending alerts when a gas leak is detected. This integration allows the system to immediately make a phone call to a specified number, ensuring that the relevant people are quickly informed of the potential danger. This real-time notification is crucial for safety, as it ensures prompt attention and action to address the gas leak. Twilio's APIs make it straightforward to implement this functionality, enabling seamless communication without the need for manual intervention.

# Prototype

![WhatsApp Image 2024-07-16 at 7 06 27 PM](https://github.com/user-attachments/assets/569475a2-f6b8-41d4-8e78-805ebd3edb83)

# Conclusion

This project introduces a groundbreaking solution for detecting and responding to gas leaks in homes and industries, integrating advanced sensors for precise detection and automated ventilation activation using motors. Utilizing TWILIO api communication ensures rapid alerts to occupants in emergencies.The inclusion of a Pneumatic solenoid valve for master shutdown enhances safety measures, preventing further gas leakage effectively. This innovative approach not only prioritizes safety but also offers a cost-effective solution, setting new standards in proactive gas leak detection and mitigation.

# Acknowledgement
We would like to extend our sincere thanks to everyone who supported us during the ELCIA hackathon.

Firstly, we thank Mr. Kunal Ghosh  and IIIT Bangalore  organizing this incredible event and providing us with the platform to bring our ideas to life. 

We extend our heartfelt thanks to our mentors, Mr. Raymond and Mr. Manjunath Ganjipete, for their invaluable guidance and insights. Their expertise and encouragement were instrumental in shaping our project and overcoming challenges along the way.

We also acknowledge the generous support from our HOD, Dr. K M Sadyojatha, whose encouragement and support greatly enhanced our development process. Their contributions made it possible for us to focus on innovation and creativity.

Lastly, we appreciate one another for the cooperation and support extended in making this implementation a reality.

Thank you all for making this journey a memorable and enriching experience.


