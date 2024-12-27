---
title: The full VEX design process (Spin-Up)
date: 2022-10-22 
categories: [Robotics]
tags: [vex, programming, cad, design]     # TAG names should always be lowercase
author: Jaron
math: true
image:
  path: /assetsweb/vexrobot/spinup-robot.jpg
  lqip: 
  alt: A robot on a VRC field 
---

## Introduction
For my third year in VEX robotics, I wanted to improve my performance from previous years. To do this, much more planning and rigor went into each step of the design process, which will be shown below.

## Starter Ideas
After first seeing the game announcement, I Immediately got to work writing down plausible ideas and discussing the intricacies of the game. After having some time to let the idea of the game, I created the CAD model shown below:

![](/assetsweb/vexrobot/spinupcad1.png)

This design had benifits such as
- Mecanum wheels allowing for easy strafing
- 4 wheel drive for extra torque on the drive train
- Simple flywheel Mechanism likely easy to repair if damaged
- Dual Flywheel system for added compression
  
However, after thinking about it more, core design flaws appeared leading me to consider a redesign, these include:
- Inefficient use of motors
- Difficulties in getting the two flywheels to be identical in speed could lead to weird flight paths
- The mechanism shown for intaking disks likely does not have enough friction to stop sliding
- Due to the Mecanum wheels, the drivetrain is quite slow as extra torque is important.

Although this design could work in theory, I felt it would be much harder to bring to life than if I made a more effective redesign.

## Iterations
To make an improved version of the design, I split the tasks into three parts, reflecting the robots functionallity:
- Base
- Indexer
- Flywheel

### Improving the Base
After experimentation, we found that the ideal gear ratio for the drive train was 60:36 for a the 200RPM gear box, making it run around 333RPM. Each wheel is then given its own motor to further increase the drivetrain torque. In this design, mecanum wheels were ditched due to the lack of application in this years game.

![](/assetsweb/vexrobot/base2.png)
  
### Improving the Indexer
To allow for a more efficent use of motors, the goal of this design was to combine the roller and indexer into the same system. This would be accomplished by connecting chains through several axels to allow for a single motor to be responsible for two jobs.

![](/assetsweb/vexrobot/Index.png)

### Improving the Flywheel
As said earlier, the duel flywheel system was clunky for several reasons, leading to a single fly wheel to be used for this redesign which is capable of spinning at 3000RPM. The disk is then sent out by compressing it between the flywheel and a C-channel. A smaller wheel would be used to bring the indexed disks. 

![](/assetsweb/vexrobot/Flywheel.png)

## Final Design
Although the CAD model lead the way for building the tobot, some last minute additions were made to increase the quality and in game performance. The changes are shown below and include changes to the structure motor placements and flywheel gear ratio.

![](/assetsweb/vexrobot/Changes.png)


On top of this, we added pnuematics to the robot to fling string onto and across the feild to increase points in the endgame. This was a challenge due to the limited space left on the robot, however, with some zipties and extra support beams it was doable.

![](/assetsweb/vexrobot/pnu.png)

## Calculations for Program
As the robot does not have an adjustible angle for the flywheel, the way it would deal with different distances is by changing the velocity it launches the disks at. To do this, kinematics was employed to seehow fast the disk would need to travel at different distances from the net, in order to score.
$$ \vec h = v_{iy}t+\frac{1}{2}\vec gt^2$$
$$ v_{x}=v cos(\theta) $$
$$ v_{y}=v sin(\theta)$$
$$ t= \frac{d_{x}}{v_{x}}\ $$

$$\therefore \vec v = \sqrt{\frac{-gd^2}{2(cos\theta)^2(h-dtan\theta)}}$$

where g is the acceleration due to gravity, d is the horizontal displacement, and theta is the angle the disk is released. This would then output a inital launch velocity at any distance from the goal creating a graph looking like this:

![](/assetsweb/vexrobot/disgraph.png)

## The Code

