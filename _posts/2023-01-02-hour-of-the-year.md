---
title: Hour of the Year 7-Segment Display + PCB Design
date: 2023-01-02 
categories: [Arduino]
tags: [programming, pcb]     # TAG names should always be lowercase
author: Jaron
image:
  path: /assetsweb/hourofyear/houry.jpg
  lqip: 
  alt: A picture of the working clock 46 hours into the year
---

## Introduction
One day in class, I found myself wondering about the number of hours in a month. This thought lead me down a rabbit hole that went to hours per year and seconds per decade, and then attempting to calculate the number in  my head. This gave me the admittitly usless idea of creating a clock which counts the number of hours that have passed since January 1st at 12 AM.

## Parts Required
To make this, I used an Arduino to process the time and send information to each component; a Real Time Clck Module (RTC) to give the current time to the Arduino; a 7-segment display which will display the hour (only 4 digits are necessary as there are 8760 hours per year); and a 74HC595 shift register which uses a Serial IN, Parrallel OUT protocol to tell the display to show a certain value.

## Schematics and PCB design
In order to make an outline in how the wiring would work, I made a schematic in Altium which could be followed when wiring the components together with a breadboard.

![](/assetsweb/hourofyear/schem.png)

This schematic was then used to create a PCB in altium. In the future, by using this condenced version of the same thing, it would make this project more durable, smaller, and more professional. Despite this, the PCB at hand has some flaws, it is a bit messy, and there is some empty space which could be removed. However in this state, it is still a large improvement over a plain breadboard.

![](/assetsweb/hourofyear/PCB2d.png)

![](/assetsweb/hourofyear/PCB.png)

## The Code
Although the fully put together system is fully capable of displaying any 4 digit number, the system needs to have a program to convert the time to hours and actually tell it to display that number on the screen. Below is the whole code segment used.

```arduino
#include <Wire.h>
#include <DS1307RTC.h>
#include <TimeLib.h>

int daysInMonth(int month, int year);

// define the pin connections:
#define DATA_PIN 10
#define LATCH_PIN 9
#define CLOCK_PIN 8
#define DIGIT_1_PIN 7
#define DIGIT_2_PIN 6
#define DIGIT_3_PIN 5
#define DIGIT_4_PIN 4

// define the pin state constants:
const int ON = HIGH;
const int OFF = LOW;

// define the digit segments:
const byte DIGIT_A = 0;
const byte DIGIT_B = 1;
const byte DIGIT_C = 2;
const byte DIGIT_D = 3;
const byte DIGIT_E = 4;
const byte DIGIT_F = 5;
const byte DIGIT_G = 6;
const byte DIGIT_DP = 7;

int brightness = 70; 

void setup() {
  // set the digit pins to output:
  pinMode(DIGIT_1_PIN, OUTPUT);
  pinMode(DIGIT_2_PIN, OUTPUT);
  pinMode(DIGIT_3_PIN, OUTPUT);
  pinMode(DIGIT_4_PIN, OUTPUT);

  // set the data, latch, and clock pins to output:
  pinMode(DATA_PIN, OUTPUT);
  pinMode(LATCH_PIN, OUTPUT);
  pinMode(CLOCK_PIN, OUTPUT);

  Serial.begin(9600);
  while (!Serial) ; // wait until Arduino Serial Monitor opens
  setSyncProvider(RTC.get);   // the function to get the time from the RTC
  if(timeStatus()!= timeSet) 
     Serial.println("Unable to sync with the RTC");
  else
     Serial.println("RTC has set the system time"); 
}
int daysInMonth(int month, int year) {
  if (month == 2) {
    // Check for leap year
    if ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0) {
      return 29;
    } else {
      return 28;
    }
  } else if (month == 4 || month == 6 || month == 9 || month == 11) {
    return 30;
  } else {
    return 31;
  }
}
void loop() {
  time_t currentTime = now() + 590;

  int hoursRemaining = 24 - hour(currentTime);
  int currentYear = year(currentTime);
  int currentMonth = month(currentTime);
  int currentDay = day(currentTime);

  // Calculate the number of days that have passed since the beginning of the year
  int daysSinceStartOfYear = currentDay - 1;
  for (int i = 1; i < currentMonth; i++) {
    daysSinceStartOfYear += daysInMonth(i, currentYear);
  }

  // Calculate the number of hours that have passed since the beginning of the year
  int hoursSinceStartOfYear = daysSinceStartOfYear * 24;
  
  displayNumber(hoursSinceStartOfYear - hoursRemaining + 8);
  
  delayMicroseconds(1638*((100-brightness)/10));
 
}

void displayNumber(int number) {
  // turn all digits off:
  digitalWrite(DIGIT_1_PIN, OFF);
  digitalWrite(DIGIT_2_PIN, OFF);
  digitalWrite(DIGIT_3_PIN, OFF);
  digitalWrite(DIGIT_4_PIN, OFF);

  // split the number into its digits:
  int digit1 = number % 10;
  int digit2 = (number / 10) % 10;
  int digit3 = (number / 100) % 10;
  int digit4 = (number / 1000) % 10;

  // display the digits:
  displayDigit(digit1, DIGIT_1_PIN);
  displayDigit(digit2, DIGIT_2_PIN);
  displayDigit(digit3, DIGIT_3_PIN);
  displayDigit(digit4, DIGIT_4_PIN);
}

void displayDigit(int digit, int digitPin) {
  // turn on the specified digit:
  digitalWrite(digitPin, ON);

  // send the digit data to the shift register:
  shiftOut(DATA_PIN, CLOCK_PIN, MSBFIRST, getSegmentData(digit));

  // latch the data to the display:
  digitalWrite(LATCH_PIN, LOW);
  digitalWrite(LATCH_PIN, HIGH);

  // delay for a short time:
  delay(1);

  // turn off the specified digit:
  digitalWrite(digitPin, OFF);

  // clear the shift register:
  shiftOut(DATA_PIN, CLOCK_PIN, MSBFIRST, 0);
  // latch the data to the display:
  digitalWrite(LATCH_PIN, LOW);
  digitalWrite(LATCH_PIN, HIGH);
}

// returns the segment data for a given digit (0-9)
byte getSegmentData(int digit) {
  switch (digit) {
    case 0: return 0x5F;
    case 1: return 0x44;
    case 2: return 0x9D;
    case 3: return 0xD5;
    case 4: return 0xC6;
    case 5: return 0xD3;
    case 6: return 0xDB;
    case 7: return 0x45;
    case 8: return 0xDF;
    case 9: return 0xC7;
    default: return B00000000;
  }
}


```