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
When there is no immediate risk of forward collision, the car attempts to switch to lane where the distance to nearest vehicle is maximum. The idea is to ensure that car avoids being in the situation where it might get to close to another vehicle in first place. In this way, the need to slow down is reduced as much as possible. This is accomplished by using a cost function based on longitudinal distance between the EGO and other vehicles in each lane. Each lane is then assigned a cost. The lane with minimum cost is the preferred lane. Car then attempts to switch to the lane with minimum cost provided there are no vehices too close to the left or right(depending on direction of lane change).
Double lane change is not allowed in this implementation since it is generally a risky maneuver in real traffic. Also, it results in jerk exceeding the limit unless speed is reduced or lane change trajectory spans much larger longitudinal distance.

Every time a new best lane is selected, it is necessary that a certain time has passed before the car decides to switch to new best lane(only after checking for lateral collision). This is to avoid slalom movement of the car due to rapid toggling of best lane which may happen in dense traffic. Also, when the car starts first time, the current lane is retained as best lane for a certain time to allow for cost to be computed properly for all lanes.

## Build Instructions and other details
Clone this repo.
Make a build directory: mkdir build && cd build
Compile: cmake .. && make
Run it: ./path_planning.
Here is the data provided from the Simulator to the C++ Program

### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

Previous path's end s and d values

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's (car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates.

## Details

The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.


## Rubric points
* Code compiles without any error. No change is made to provided CMake files for build.
* Car is able to drive safely without exceeding speed limit, acceleration limit or jerk limit for at least 4.32 miles.
* Car does not collide with other vehicles
* Car is able to change lanes when ever required and only when there is no traffic in the adjacent lane
* Car avoids collision by braking when ever necessary
* Car does not stay out of the lane for duration longer than specified.


