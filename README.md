#include <Servo.h>

Servo servo1;
const int trigPin = 12;
const int echoPin = 11;
const int potPin = A0;

long duration;
int distance = 0;
int soil = 0;
int Ultra_Distance = 15;   // ultrasonic trigger distance (cm)
int threshold = 980;       // soil moisture threshold (Wet < 980)

// Servo positions
const int NEUTRAL_POS = 60;
const int WET_POS = 30;     // Left side for wet waste
const int DRY_POS = 150;    // Right side for dry waste

void setup() {
  Serial.begin(9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  
  servo1.attach(8);
  servo1.write(NEUTRAL_POS);   // Start at neutral
  delay(1000);
  
  Serial.println("🗑 Smart Waste Segregation System Ready!");
  Serial.println("Wet threshold: < " + String(threshold));
  Serial.println("Detection range: " + String(Ultra_Distance) + " cm");
  Serial.println("========================");
}

void loop() {
  // 🔹 Measure distance using ultrasonic sensor
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.034 / 2;
  
  // 🔹 Check if object is detected within range
  if (distance > 1 && distance < Ultra_Distance) {
    Serial.println("\n🔍 Object detected at " + String(distance) + " cm");
    Serial.println("📊 Reading moisture sensor...");
    
    delay(1000);  // Wait for stable reading
    
    // Read moisture sensor value FIRST
    soil = analogRead(potPin);
    Serial.println("📊 Moisture reading: " + String(soil));
    
    // 🔹 NOW decide where servo should move based on moisture reading
    if (soil < threshold) {
      // WET WASTE detected - move servo to WET position
      Serial.println("💧 WET WASTE DETECTED!");
      Serial.println("🔄 Moving servo to WET position...");
      moveServoTo(WET_POS, "WET");
    } else {
      // DRY WASTE detected - move servo to DRY position
      Serial.println("📄 DRY WASTE DETECTED!");
      Serial.println("🔄 Moving servo to DRY position...");
      moveServoTo(DRY_POS, "DRY");
    }
    
    // Keep position for sorting
    delay(3000);  // Hold position for 3 seconds to allow waste to fall through
    
    // Return to neutral position
    Serial.println("🔄 Returning to neutral position...");
    moveServoTo(NEUTRAL_POS, "NEUTRAL");
    
    Serial.println("✅ Segregation complete!\n");
    delay(2000);  // Wait before next detection
  }
  
  delay(200);  // Main loop delay
}

// Function to move servo smoothly
void moveServoTo(int targetPos, String wasteType) {
  Serial.println("➡ Moving to " + wasteType + " position (" + String(targetPos) + "°)");
  
  int currentPos = servo1.read();
  
  // Smooth movement
  if (currentPos < targetPos) {
    for (int pos = currentPos; pos <= targetPos; pos += 3) {
      servo1.write(pos);
      delay(20);
    }
  } else {
    for (int pos = currentPos; pos >= targetPos; pos -= 3) {
      servo1.write(pos);
      delay(20);
    }
  }
  
  servo1.write(targetPos);  // Ensure exact position
  Serial.println("✅ Reached " + wasteType + " position!");
}
