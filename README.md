#include <Wire.h>
#include <Adafruit_ADS1X15.h>

Adafruit_ADS1115 ads;

// MUX Select Pins
const int mux1_S0 = 2;
const int mux1_S1 = 3;
const int mux1_S2 = 4;
const int mux1_S3 = 5;

const int mux2_S0 = 6;
const int mux2_S1 = 7;
const int mux2_S2 = 8;
const int mux2_S3 = 9;

// Reference resistor in ohms
const float REF_RESISTOR = 1000.0;  // 1kΩ known resistor

// MUX channel combinations for 2x2 electrode matrix
int inputPairs[4][2] = {
  {0, 1},  // Z11
  {0, 2},  // Z12
  {3, 1},  // Z21
  {3, 2}   // Z22
};

void setup() {
  Serial.begin(9600);
  Wire.begin();
  ads.begin();
  ads.setGain(GAIN_ONE);  // +/-4.096V range

  // Set MUX select pins as output
  for (int pin = 2; pin <= 9; pin++) {
    pinMode(pin, OUTPUT);
  }

  Serial.println("Bio-Impedance Matrix Measurement Started");
}

void loop() {
  float impedance[2][2];

  for (int i = 0; i < 4; i++) {
    int A = inputPairs[i][0];
    int B = inputPairs[i][1];

    // Set multiplexer inputs
    selectMUXChannel(1, A);  // Current injection channel
    selectMUXChannel(2, B);  // Voltage measurement channel

    delay(50);  // Allow MUX to settle

    int16_t adcReading = ads.readADC_Differential_0_1();  // AIN0 - AIN1
    float voltage = ads.computeVolts(adcReading);         // in volts

    float current = 1.0 / REF_RESISTOR;  // Assuming 1V across known resistor = 1 mA
    float Z = voltage / current;

    if (i == 0) impedance[0][0] = Z;
    else if (i == 1) impedance[0][1] = Z;
    else if (i == 2) impedance[1][0] = Z;
    else if (i == 3) impedance[1][1] = Z;
  }

  // Print 2x2 Impedance Matrix
  Serial.println("Impedance Matrix (Ohms):");
  Serial.print("[ ");
  Serial.print(impedance[0][0], 2); Serial.print("\t");
  Serial.print(impedance[0][1], 2); Serial.println(" ]");
  Serial.print("[ ");
  Serial.print(impedance[1][0], 2); Serial.print("\t");
  Serial.print(impedance[1][1], 2); Serial.println(" ]");
  Serial.println("---------------------------");

  delay(1000);  // Update every second
}

// Function to select channel on MUX1 or MUX2
void selectMUXChannel(int muxNum, int channel) {
  bool s0 = bitRead(channel, 0);
  bool s1 = bitRead(channel, 1);
  bool s2 = bitRead(channel, 2);
  bool s3 = bitRead(channel, 3);

  if (muxNum == 1) {
    digitalWrite(mux1_S0, s0);
    digitalWrite(mux1_S1, s1);
    digitalWrite(mux1_S2, s2);
    digitalWrite(mux1_S3, s3);
  } else if (muxNum == 2) {
    digitalWrite(mux2_S0, s0);
    digitalWrite(mux2_S1, s1);
    digitalWrite(mux2_S2, s2);
    digitalWrite(mux2_S3, s3);
  }
}
