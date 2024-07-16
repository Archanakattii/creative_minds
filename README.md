# creative_minds

# SMART GAS SAFETY SYSTEM : DETECTING GAS LEAKS FOR SAFER HOMES AND INDUSTRIES

# Introduction
The history of gas leak detection systems dates back to the early 20th century when rudimentary methods, such as canaries in coal mines, were used to detect toxic gases. Over time, technology advanced, leading to the development of electronic sensors capable of detecting various hazardous gases. The introduction of microcontrollers and wireless communication modules further enhanced these systems, allowing for real-time monitoring and automated responses. The Smart Gas Safety System is a modern iteration of these advancements, designed to address the critical issue of detecting hazardous gas leaks, specifically carbon monoxide and methane, in industrial and residential settings. Using advanced MQ-4 and MQ-7 gas sensors and the ESP8266 microcontroller, the system provides timely detection and real-time alerts via the TWILIO API module. It also automates safety measures by activating ventilation systems with servo or stepper motors and employs a pneumatic solenoid valve for master shutdown, ensuring comprehensive protection against gas-related incidents.</p>

# Overview
The Smart Gas Safety System is a comprehensive solution designed to detect hazardous gas leaks, such as carbon monoxide and methane, in residential and industrial environments. Utilizing advanced MQ-4 and MQ-7 gas sensors, along with the ESP8266 microcontroller, the system ensures timely detection of gas leaks. Real-time alerts are sent through the TWILIO API module, notifying residents and authorities immediately. The system also automates safety responses by activating ventilation systems via servo or stepper motors and employing a pneumatic solenoid valve for a master shutdown to stop further gas leakage. This modern approach enhances safety by protecting occupants and property from potential gas-related incidents.</p>

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
| solenoid valve  | 01 | Electromagnetic actuator |   |
| adapter         | 01 | power supply for solenoid valve| 

# TABLE FOR PIN CONECTIONS

## Publisher Module wiring connections

| ESP8266(1) | MQ-2|
|----------|----------|
| A0   | A0 |
| VCC   | vin(3.3V)   |
| GND | GND   |


## BLOCK DIAGRAM
![Swe 1  (2)](https://github.com/Archanakattii/creative_minds/assets/160317292/5773471e-3293-4fda-ad28-b1e901ddd590)


# Working Code
The system is divided into two parts:</p>
1.Publisher module</p>
2.Subscriber module</p>
Here MQTT acts as a broker</p>

**1.Publisher Module**

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

**2.Subscriber Module**

    #include <ESP8266WiFi.h>
    #include <ESP8266HTTPClient.h>
    #include <base64.h>
    #include <PubSubClient.h>
    #include <Servo.h>
    
    // WiFi credentials
    const char* ssid = "Swez";
    const char* password = "12345678910";
    
    // MQTT Broker
    const char* mqtt_server = "broker.hivemq.com";
    const char* topic = "home/gasLeak";
    
    // Pins
    #define SERVO_PIN D0
    #define RELAY_PIN D1
    
    // Twilio credentials and phone numbers
    const char* account_sid = "AC75879657656b99ec3e7f6881e050a8d2";
    const char* auth_token = "ac66b85e0aca9ecc689d5438539eab03";
    const char* to_number = "+918197204255";
    const char* from_number = "+12512783043";
    const char* twilio_url = "https://api.twilio.com/2010-04-01/Accounts/AC75879657656b99ec3e7f6881e050a8d2/Calls.json";
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
    
        
       
            
WiFi Connection:</p>
The setup_wifi() function connects the ESP8266 to the specified WiFi network.</p>
MQTT Client Setup:</p>
Sets up the MQTT client to connect to the HiveMQ broker and subscribes to the topic home/gasLeak.</p>
Servo and Relay Initialization:</p>
Initializes the servo motor and sets its initial position.</p>
Initializes the relay and sets its initial state to OFF.</p>
Callback Function:</p>
Processes incoming MQTT messages. If a "leak" message is received, it turns the relay off, sets the servo to 180 degrees, and makes a call via Twilio. If a "safe" message is received, it turns the relay on and sets the servo to 0 degrees.</p>
Twilio Call Function:</p>
Uses the Twilio API to make a call when a gas leak is detected.</p>

    
    
# WORK PROGRESS
We have interfaced our board with the sensor(publisher module) when the gas leak is detected we get an alert by call through twilio.</p>




https://github.com/Archanakattii/creative_minds/assets/160317292/18dee359-a3c2-4adb-affd-fe9cc0c11040




Here we used smoke instead of LPG gas later we would replace smoke by non-flammable gas which has similar threshold as LPG</p>

## Code used for publisher module

    #include <ESP8266WiFi.h>
    #include <ESP8266HTTPClient.h>
    #include <base64.h>
    #include <PubSubClient.h>
    
    // WiFi credentials
    const char* ssid = "Archanakatti";
    const char* password = "201401167";
    
    // MQTT Broker
    const char* mqtt_server = "broker.hivemq.com";
    const char* topic = "home/gasLeak";
    
    // Pins
    #define GAS_SENSOR_PIN A0
    
    WiFiClient espClient;
    PubSubClient client(espClient);
    
    // Twilio credentials and phone numbers
    const char* account_sid = "AC75879657656b99ec3e7f6881e050a8d2";
    const char* auth_token = "ac66b85e0aca9ecc689d5438539eab03";
    const char* to_number = "+918197204255";
    const char* from_number = "+12512783043";
    const char* twilio_url = "https://api.twilio.com/2010-04-01/Accounts/AC75879657656b99ec3e7f6881e050a8d2/Calls.json";
    
    // Twilio XML URL for call instructions (replace with your own)
    const char* twiml_url = "http://demo.twilio.com/docs/voice.xml";
    
    // Flag to track if a call has been made
    bool callMade = false;
    
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
        if (client.connect("ESP8266Client")) {
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
    
    if (gasValue > 400 && !callMade) { // Adjust threshold as necessary
      client.publish(topic, "leak");
      makeTwilioCall();
      callMade = true;
    } else {
      client.publish(topic, "safe");
      callMade = false;
    }
  
    delay(2000); // Publish data every 2 seconds
    }
    
    void makeTwilioCall() {
      WiFiClientSecure client;
      client.setInsecure();  
      HTTPClient http;
      String auth = String(account_sid) + ":" + String(auth_token);
      
    // Declare the char array with the correct size
    char auth_char_array[auth.length() + 1];
    auth.toCharArray(auth_char_array, auth.length() + 1);
    
    String auth_base64 = base64::encode(auth_char_array);
  
    http.begin(client, twilio_url);
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
    




## Pending Work
</p> 1.90 % work regaurding subscriber module is done (to be completed by 12.07.24)
</p> 2.Soldering all the components on the perforated board 
</p> 3.Model making (to be completed before 14.07.24)

