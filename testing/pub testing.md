
We have interfaced our board with the sensor(publisher module) when the gas leak is detected we get an alert by call through twilio.</p>




https://github.com/Archanakattii/creative_minds/assets/160317292/18dee359-a3c2-4adb-affd-fe9cc0c11040


Here we used smoke instead of LPG gas later we would replace smoke by non-flammable gas which has similar threshold as LPG</p>

## Code used for publisher module

    #include <ESP8266WiFi.h>
    #include <ESP8266HTTPClient.h>
    #include <base64.h>
    #include <PubSubClient.h>
    
    // WiFi credentials
    const char* ssid = "abcd";
    const char* password = "*******";
    
    // MQTT Broker
    const char* mqtt_server = "broker.hivemq.com";
    const char* topic = "home/gasLeak";
    
    // Pins
    #define GAS_SENSOR_PIN A0
    
    WiFiClient espClient;
    PubSubClient client(espClient);
    
    // Twilio credentials and phone numbers
    const char* account_sid = "twilio id";
    const char* auth_token = "twilio token";
    const char* to_number = "to num";
    const char* from_number = "from num";
    const char* twilio_url = "https://api.twilio.com/2010-04-01/Accounts/twilio id/Calls.json";
    
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
