# CarND path planning project term3 P1
## Overview
In this project, we need to implement a path planning algorithms to drive a car on a highway on a simulator provided by Udacity. he simulator sends car telemetry information (car's position and velocity) and sensor fusion information about the rest of the cars in the highway (Ex. car id, velocity, position). It expects a set of points spaced in time at 0.02 seconds representing the car's trajectory. The communication between the simulator and the path planner is done using websocket.Udacity provides a seed project to start from on this project.
Main goal is to utilize the information from the simulator and performing situation analysis to keep the car free from colliding and also optimizing the car's trajectory by choosing the best lane, at the same time ensuring that car remains within speed limit.

## Pre-requisites
The project has the following dependencies (from Udacity's seed project):

* cmake >= 3.5
* make >= 4.1
* gcc/g++ >= 5.4
* libuv 1.12.0
* Udacity's simulator.

## Implementation details
Implementation is based on start up code presented in project walk through in the class. 
The algorithm performs the following steps:
* Read sensor fusion data from simulator
* Identify the position of EGO and other vehicles on the road.
* Detect collision by checking for vehicles in 30m vicinity ahead, to the right and to the left of the the EGO vehicle
* If a collision ahead is detected, check if lane change to the right or left is possible provided no collision was
detected side ways.
* If lane change is possible, then change lane immediately. First, left lane is checked for collision. If left lane is free, then a change to left lane is performed. If left lane is not free, right lane change is performed.
* If lane change is not possible, then reduce the speed of EGO vehicle in steps such that total acceleration and jerk values are within the limit.(Here a value of 0.224 is choosen as step size for speed increase/decrease which turns out to be maximum allowed acceleration of 5m/s). 
* Once collision is no longer detected, speed is again increased to 50mph in steps of 0.224 mps.
If a collision ahead is not detected, then EGO attempts to switch to a best lane provided there is no risk of collision in such a maneuver. Best lane is chosen based on the cost associated with each lane.
In the current implementation, double lane change is not allowed since it was observed that a double lane change often resulted jerk increasing beyond the limits unless speed is reduced.

## Trajectory formation
For lane trajectory generation, spline points is used. Using end of previous path or current state of the car(if previous path points are all used up) and another 3 optimally spaced points(which considers lane change), spline is fit to generate future points.
In each cycle, at the most 50 path points are generated depending on how many points from previous path are consumed.

## Best lane


