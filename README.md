# creative_minds

# SMART GAS SAFETY SYSTEM : DETECTING GAS LEAKS FOR SAFER HOMES AND INDUSTRIES

# Introduction
The Smart Gas Safety System is designed to address the critical issue of detecting hazardous gas leaks, specifically carbon monoxide and methane, in both residential and industrial settings. Gas leaks pose significant health and safety risks as they are often undetectable by smell or sight, leading to potential poisoning, explosions, and fires. Our system employs advanced gas sensors (MQ-4 and MQ-7) and the ESP8266 microcontroller to ensure timely detection and real-time alerts through TWILIO API module. In addition to alerting residents, the system automatically activates ventilation systems using servo or stepper motors to mitigate immediate risks. For enhanced safety, a pneumatic solenoid valve is incorporated for master shutdown to stop further gas leakage. This comprehensive solution offers a modern approach to gas safety, protecting occupants and property from potential gas-related incidents.

# WORK PROGRESS
We have started working on our project and  have received all components, tested them on a breadboard, (for sensors and servomotor) and found one microcontroller is not working so we will have to order a new one,and we have ordered our solenoid valve and haven,t received it yet...

## Testing of components 

<p>https://github.com/Archanakattii/creative_minds/assets/160317297/80fc089a-efd6-4dbd-9a4d-f3db01a4527e</p>

here we have used smoke instead of gas for the detection from the sensor
</p> next, we'll connect the components to the cloud (broker) using the MQTT protocol, also interface the TWILIO Api module with the publisher module, and later solder everything onto a perforated board.

## Code used for components testing
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
