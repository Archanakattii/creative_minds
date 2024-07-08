Here we have used smoke instead of gas for the detection from the sensor
    
    #include <Servo.h>
    
    // Define the servo motor
    Servo myservo;
    
    // Define the analog pin for the MQ-2 sensor
    const int mq2Pin = A0;
    
    // Threshold value for gas detection (adjust as necessary)
    const int gasThreshold = 300;
    
    void setup() {
      // Attach the servo motor to pin D4
      myservo.attach(D4);
    
      // Begin serial communication
      Serial.begin(115200);
    
      // Move the servo to the initial position
      myservo.write(0);
    }
    
    void loop() {
      // Read the analog value from the MQ-2 sensor
      int gasValue = analogRead(mq2Pin);
      Serial.print("Gas Value: ");
      Serial.println(gasValue);
    
      // Check if the gas value exceeds the threshold
      if (gasValue > gasThreshold) {
        // Move the servo to 90 degrees if gas is detected
        myservo.write(90);
        Serial.println("Gas detected! Servo moved to 90 degrees.");
      } else {
        // Move the servo to 0 degrees if no gas is detected
        myservo.write(0);
        Serial.println("No gas detected. Servo at 0 degrees.");
      }
    
      // Delay for a while before the next reading
      delay(1000);
    }
