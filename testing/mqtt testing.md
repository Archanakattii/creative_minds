## AT PUBLISHER MODULE</p>

    #include <ESP8266WiFi.h>
    #include <PubSubClient.h>
    
    const char* ssid = "your_SSID";
    const char* password = "your_PASSWORD";
    const char* mqtt_server = "broker_ip_address";
    
    WiFiClient espClient;
    PubSubClient client(espClient);
    
    const int ledPin = D1;
    
    void setup() {
      pinMode(ledPin, OUTPUT);
      Serial.begin(115200);
      setup_wifi();
      client.setServer(mqtt_server, 1883);
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
        if (client.connect("ESP8266Publisher")) {
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
    
      digitalWrite(ledPin, HIGH);
      client.publish("home/ledStatus", "ON");
      delay(2000);
    
      digitalWrite(ledPin, LOW);
      client.publish("home/ledStatus", "OFF");
      delay(2000);
    }


##AT SUBSCRIBER MODULE</p>


    #include <ESP8266WiFi.h>
    #include <PubSubClient.h>
    
    const char* ssid = "your_SSID";
    const char* password = "your_PASSWORD";
    const char* mqtt_server = "broker_ip_address";
    
    WiFiClient espClient;
    PubSubClient client(espClient);
    
    const int ledPin = D1;
    
    void setup() {
      pinMode(ledPin, OUTPUT);
      Serial.begin(115200);
      setup_wifi();
      client.setServer(mqtt_server, 1883);
      client.setCallback(callback);
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
    
    void callback(char* topic, byte* payload, unsigned int length) {
      payload[length] = '\0';
      String message = String((char*)payload);
    
      if (String(topic) == "home/ledStatus") {
        if (message == "ON") {
          digitalWrite(ledPin, HIGH);
          Serial.println("ON");
        } else if (message == "OFF") {
          digitalWrite(ledPin, LOW);
          Serial.println("OFF");
        }
      }
    }
    
    void reconnect() {
      while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");
        if (client.connect("ESP8266Subscriber")) {
          Serial.println("connected");
          client.subscribe("home/ledStatus");
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
    }


https://github.com/Archanakattii/creative_minds/assets/160317292/fee36191-360a-461c-b8ec-53d878c3053d



