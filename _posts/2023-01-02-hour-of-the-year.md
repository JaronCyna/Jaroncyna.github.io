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

## Schematics 
In order to make an outline in how the wiring would work, I made a schematic in Altium which could be followed when wiring the components together with a breadboard.
