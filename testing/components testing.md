
    </p> #include <Servo.h>
    
    // Define the pins
    </p>const int gasSensorPin = A0;   // Analog pin for MQ-2 sensor
    </p>p>const int servoPin = D1;       // Digital pin for Servo motor
    
    </p>Servo myservo;  // Create servo object
    
    </p>void setup() {
     </p> Serial.begin(9600);     // Start serial communication
     </p> myservo.attach(servoPin);   // Attach servo to the pin
     </p> myservo.write(0);   // Initialize servo to 0 degrees
    </p>}
    
    </p>void loop() {
 </p> int sensorValue = analogRead(gasSensorPin);  // Read the MQ-2 sensor value

 </p> Serial.print("Gas Sensor Value: ");
 </p> Serial.println(sensorValue);   // Print the sensor value to the Serial Monitor

 </p> if (sensorValue > 300) 
 </p> {                     
  </p>  myservo.write(90);   // Turn the servo to 90 degrees
 </p>   delay(1000);         // Keep it there for a second
  </p> else  
  </p> {
  </p>  myservo.write(0);   // Otherwise, keep the servo at 0 degrees
 </p> }

 </p> delay(100);   // Short delay before the next loop iteration
</p>}
