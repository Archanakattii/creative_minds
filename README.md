# creative_minds

# SMART GAS SAFETY SYSTEM : DETECTING GAS LEAKS FOR SAFER HOMES AND INDUSTRIES

# Introduction
The Smart Gas Safety System is designed to address the critical issue of detecting hazardous gas leaks, specifically carbon monoxide and methane, in both residential and industrial settings. Gas leaks pose significant health and safety risks as they are often undetectable by smell or sight, leading to potential poisoning, explosions, and fires. Our system employs advanced gas sensors (MQ-4 and MQ-7) and the ESP8266 microcontroller to ensure timely detection and real-time alerts through TWILIO API module. In addition to alerting residents, the system automatically activates ventilation systems using servo or stepper motors to mitigate immediate risks. For enhanced safety, a pneumatic solenoid valve is incorporated for master shutdown to stop further gas leakage. This comprehensive solution offers a modern approach to gas safety, protecting occupants and property from potential gas-related incidents.</p>

## BLOCK DIAGRAM
![Swe 1  (2)](https://github.com/Archanakattii/creative_minds/assets/160317292/5773471e-3293-4fda-ad28-b1e901ddd590)


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
    }



## Pending Work
</p> 1.90 % work regaurding subscriber module is done (to be completed by 12.07.24)
</p> 2.Soldering all the components on the perforated board 
</p> 3.Model making (to be completed before 14.07.24)

