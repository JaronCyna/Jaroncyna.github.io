---
title: Making A Game In Desmos
date: 2023-07-09 
categories: [Math, Programming]
tags: [desmos, math]     # TAG names should always be lowercase
author: Jaron
image:
  path: /assetsweb/desmosgame/desmos.png
  lqip: 
  alt: A picture showing the game running within Desmos 
---



# Introduction

When I started learning about calculus and graphing became a fundamental part of my courses, Desmos naturally presented an intuitive and appealing way of displaying functions. I quickly became interested in the different elements that could be modified to varying degrees of succsess. After some exploring, I stumbled across the Desmos Art Competition, which gave me the idea to see if it was possible to program a game within Desmos.

# What Is the Game
The idea behind the game was to make it similar to some of the first games with simple objectives such as pong and galaga. The idea materialized as a bagel shaped character who needed to shield from oncoming projectiles using a wall.

# Functions and Graphical Elements
To display what the user sees, Desmos is limited to using functions and area. Despite these limitations, as there is a seemingly infinite amount of functions, making interesting and appealing shapes is doable.

As for the wall that the sheilds the character, it was made using a multivariable function which is able to be reversed using a multiplier that is either equal to 1 or negative 1. Then, to make the motion more fluid, a changing variable was added to the parameter of the function to create an interesting effect

{:refdef: style="text-align: center;"}
<div class="container">
  <div class="video">
    <video controls muted style="border-radius: 4px;" width="100%" preload="auto">
      <source src="/assetsweb/desmosgame/wallfunc.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>
{: refdef}

For the main character, the standard formula for an elipse was used, but a changing variable was added to create the illusion of spinning.

{:refdef: style="text-align: center;"}
<div class="container">
  <div class="video">
    <video controls muted style="border-radius: 4px;" width="100%" preload="auto">
      <source src="/assetsweb/desmosgame/charspin.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>
{: refdef}

# Logical Operators
In order to make any simple game, there needs to be a way to get an input and an output based on what is currently going on. Desmos provided a unique challenge in finding these as they needed to be defined mathimatically. To learn how making different logical operators within Desmos works, trial and error along with looking through forums was done. As for the actual operators used they were the following:

