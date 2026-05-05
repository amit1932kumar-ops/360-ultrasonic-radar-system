#include <Adafruit_GFX.h>
#include <Adafruit_ST7735.h>

// TFT Setup
#define TFT_CS   4
#define TFT_RST  3 
#define TFT_DC   2
Adafruit_ST7735 tft(TFT_CS, TFT_DC, TFT_RST);

// Ultrasonic Pins
const int trigPin = 6;
const int echoPin = 7;

// Stepper Motor Pins
const int IN1 = 8;
const int IN2 = 9;
const int IN3 = 10;
const int IN4 = 5;

// Variables
float angle = 0.0;
bool forward = true;
int distance = 0;
unsigned long lastUpdateTime = 0;
const int updateInterval = 100; // ms

unsigned long lastMotorStepTime = 0;
const int motorStepDelay = 4; // ms per step

int stepNumber = 0;

// Full-step sequence (for 28BYJ-48)
const int stepSequence[4][4] = {
  {1, 0, 1, 0},
  {0, 1, 1, 0},
  {0, 1, 0, 1},
  {1, 0, 0, 1}
};

void setup() {
  tft.initR(INITR_BLACKTAB);
  tft.fillScreen(ST7735_BLACK);
  tft.setTextColor(ST7735_WHITE);
  tft.setTextSize(1);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  Serial.begin(115200);
  Serial.println("Radar Started");
}

void loop() {
  moveMotorCustom();

  if (millis() - lastUpdateTime > updateInterval) {
    distance = getDistance();
    updateDisplay(angle, distance);
    sendSerialData((int)angle, distance);
    lastUpdateTime = millis();
  }
}

void moveMotorCustom() {
  if (millis() - lastMotorStepTime >= motorStepDelay) {
    if (forward) {
      stepNumber++;
      if (stepNumber > 3) stepNumber = 0;
      angle += (360.0 / 2048.0); // 4096 steps per full rotation approx
      if (angle >= 360.0) {
        angle = 359.9;
        forward = false;
      }
    } else {
      stepNumber--;
      if (stepNumber < 0) stepNumber = 3;
      angle -= (360.0 / 2048.0);
      if (angle <= 0.0) {
        angle = 0.1;
        forward = true;
      }
    }

    setStep(stepNumber);
    lastMotorStepTime = millis();
  }
}

void setStep(int step) {
  digitalWrite(IN1, stepSequence[step][0]);
  digitalWrite(IN2, stepSequence[step][1]);
  digitalWrite(IN3, stepSequence[step][2]);
  digitalWrite(IN4, stepSequence[step][3]);
}

int getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000); 
  if (duration == 0) return 0;
  return duration * 0.034 / 2;
}

void updateDisplay(float angle, int distance) {
  tft.fillScreen(ST7735_BLACK);

  int centerX = tft.width() / 2;
  int centerY = tft.height() / 2;
  int radius = min(centerX, centerY) - 5;

  for (int r = radius; r > 0; r -= radius/4) {
    tft.drawCircle(centerX, centerY, r, ST7735_GREEN);
  }

  float radAngle = radians(angle - 90);
  int x = centerX + radius * cos(radAngle);
  int y = centerY + radius * sin(radAngle);
  tft.drawLine(centerX, centerY, x, y, ST7735_GREEN);

  if (distance > 0 && distance < 50) {
    int objRadius = map(distance, 0, 50, 0, radius);
    int objX = centerX + objRadius * cos(radAngle);
    int objY = centerY + objRadius * sin(radAngle);
    tft.fillCircle(objX, objY, 3, ST7735_RED);
  }

  tft.setCursor(5, 5);
  tft.print("Angle: ");
  tft.print((int)angle);
  tft.print((char)223);

  tft.setCursor(5, 20);
  tft.print("Dist: ");
  tft.print(distance);
  tft.print("cm");
}

void sendSerialData(int angle, int distance) {
  Serial.print(angle);
  Serial.print(",");
  Serial.print(distance);
  Serial.println(".");
}