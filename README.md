# SMART IRRIGATION CONTROLLER
mini project for irrigation system
CODE:
// Corrected Arduino Soil Moisture Sensor with Variable Pump Speed
// Motor ON when DRY, Motor OFF when WET (~70% or above)

const int sensorPin = A0;
const int sensorPower = 8;
const int LED_Dry = 2;       // Red LED - Dry + Pump ON
const int LED_Optimal = 3;   // Green LED - Optimal + Pump OFF
const int pumpPin = 11;      // PWM pin for pump

// Calibration values - MUST be calibrated!
int dryValue = 850;   // Raw reading in completely DRY soil
int wetValue = 350;   // Raw reading in fully WET soil

const int optimalPercent = 70;   // Target moisture level

int sensorReading;
int moisturePercent;
int pumpSpeed = 0;

const int minPumpSpeed = 80;    // Minimum speed near optimal
const int maxPumpSpeed = 255;   // Maximum speed when very dry

void setup() {
  pinMode(LED_Dry, OUTPUT);
  pinMode(LED_Optimal, OUTPUT);
  pinMode(sensorPower, OUTPUT);
  pinMode(pumpPin, OUTPUT);
  
  digitalWrite(sensorPower, LOW);
  digitalWrite(pumpPin, LOW);
  
  Serial.begin(9600);
  Serial.println("Soil Moisture System - FIXED");
  Serial.println("Pump ON when dry | OFF when >= 70%");
}

void loop() {
  // Read sensor
  digitalWrite(sensorPower, HIGH);
  delay(10);
  sensorReading = analogRead(sensorPin);
  digitalWrite(sensorPower, LOW);

  // Convert to moisture percentage (0% = dry, 100% = wet)
  moisturePercent = map(sensorReading, dryValue, wetValue, 0, 100);
  moisturePercent = constrain(moisturePercent, 0, 100);

  // === CORRECTED CONTROL LOGIC ===
  if (moisturePercent >= optimalPercent) {
    // Soil is wet enough → TURN PUMP OFF
    pumpSpeed = 0;
    digitalWrite(LED_Dry, LOW);
    digitalWrite(LED_Optimal, HIGH);
  } 
  else {
    // Soil is dry → TURN PUMP ON with variable speed
    // Drier soil = higher speed
    pumpSpeed = map(moisturePercent, 0, optimalPercent-1, maxPumpSpeed, minPumpSpeed);
    
    digitalWrite(LED_Dry, HIGH);
    digitalWrite(LED_Optimal, LOW);
  }

  // Apply speed to pump (PWM)
  analogWrite(pumpPin, pumpSpeed);

  // Very fast response
  delay(50);  

  // Monitoring output
  Serial.print("Raw: ");
  Serial.print(sensorReading);
  Serial.print(" | Moisture: ");
  Serial.print(moisturePercent);
  
  Serial.print("% | Pump Speed: ");
  Serial.println(pumpSpeed);
}
