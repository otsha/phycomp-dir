---
title: "Week 4"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Serial Communication with Processing

The sensors that I tried out:

- Sparkfun Grid-Eye Infrared Array (AMG88 series)
- Adafruit Triple-Axis Magnetometer (TLV493D)

Essentially, the sketch displays what the infrared sensor detects. The color of the squares is determined by the values received from the magnetometer, while the infrared sensor's values are mapped to control the opacity (alpha) of each square.


![That's me!](images/ir-1.png)
![That's also me](images/ir-2.png)

## Arduino Code

```arduino
#include <SparkFun_GridEYE_Arduino_Library.h>
#include <Tlv493d.h>
#include <Wire.h>

GridEYE irGrid;
Tlv493d mag = Tlv493d();

int pixels[64];

void setup() {
  Serial.begin(9600);
  Wire.begin();
  irGrid.begin();
  mag.begin();
}

void loop() {
  readIrGrid();
  mag.updateData();

  outputSerialData();

  delay(100);
}

void readIrGrid() {
  for (int i = 0; i < 64; i++) {
    pixels[i] = irGrid.getPixelTemperature(i);
  }
}
/**
* Print out the data so that it can be accessed within Processing.
* The data is in the following format:
* p1, p2, ... p64,    mx, my, mz
*/
void outputSerialData() {
  for (int p : pixels) {
    Serial.print(p);
    Serial.print(",");
  }
  
  Serial.print("\t");

  Serial.print(mag.getX());
  Serial.print(",");
  Serial.print(mag.getY());
  Serial.print(",");
  Serial.print(mag.getZ());

  Serial.println("");
}
```

## Processing Code

```processing
import processing.serial.*;

Serial serial;

int[] temps; // Temperatures received from the IR sensor
PVector tempCol; // Heat color as determined by the magnetometer

void setup() {
  size(480, 480);
  serial = new Serial(this, Serial.list()[0], 9600);
  temps = new int[64]; // The IR sensor is 8x8 pixels in size
  tempCol = new PVector(0, 0, 0);
}

void draw() {
  background(0, 0, 50);
  
  readSerial();
  
  noStroke();
  
  // Draw what the IR sensor "sees" as squares
  for (int i = 0; i < 8; i += 1) {
    for (int j = 0; j < 8; j += 1) {
      
      // Determine the intensity of the color of the square (how warm is it?)
      float tempFac = map(temps[(i * 8) + j], 15, 37, 0.0, 1.0);
      
      // Draw the square
      fill(tempCol.x, tempCol.y, tempCol.z, 255 * tempFac);
      square(i * (width / 8), j * (height / 8), width / 8);
    }
  }
}

void readSerial() {
  if (serial.available() > 0) {
    
    // Read all serial data
    String rawData = serial.readStringUntil('\n');
    
    if (rawData != null) {
      String[] data = rawData.split("\t"); // Split the raw data into IR sensor and magnetometer data 
      String[] tempData = data[0].split(","); // Split the temperature data
      String[] magData = data[1].split(","); // Split the magnetometer data
      
      // Convert the temperature data into integers and chuck it in the array
      for (int i = 0; i < tempData.length; i++) {
        temps[i] = int(tempData[i]);
      }
      
      // Map the magnetometer values to RGB
      tempCol.x = map(abs(float(magData[0])), 0, 10, 0, 255);
      tempCol.y = map(abs(float(magData[1])), 0, 10, 0, 255);
      tempCol.z = map(abs(float(magData[2])), 0, 10, 0, 255);
    }
  }
}
```