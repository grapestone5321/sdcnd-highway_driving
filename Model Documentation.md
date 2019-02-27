# Reflection on how to generate paths
### CarND-Path-Planning-Project

## Introduction
The goal of this project is to build a path planner that creates smooth, safe trajectories for the car to follow. The highway track has other vehicles, all going different speeds, but approximately obeying the 50 MPH speed limit. The car transmits its location, along with its sensor fusion data, which estimates the location of all the vehicles on the same side of the road.

#### Point Paths
The path planner should output a list of x and y global map coordinates. Each pair of x and y coordinates is a point, and all of the points together form a trajectory. You can use any number of points that you want, but the x list should be the same length as the y list. Every 20 ms the car moves to the next point on the list. The car's new rotation becomes the line between the previous waypoint and the car's new location. The car moves from point to point perfectly, so you don't have to worry about building a controller for this project.


#### Velocity
The velocity of the car depends on the spacing of the points. Because the car moves to a new waypoint every 20ms, the larger the spacing between points, the faster the car will travel. The speed goal is to have the car traveling at (but not above) the 50 MPH speed limit as often as possible. But there will be times when traffic gets in the way.

## Complex Paths
#### Using Previous Path Points
The code snippet builds a 50 point path. The code snippet starts the new path with whatever previous path points were left over from the last cycle. Then we append new waypoints, until the new path has 50 total waypoints. Using information from the previous path ensures that there is a smooth transition from cycle to cycle. But the more waypoints we use from the previous path, the less the new path will reflect dynamic changes in the environment. Ideally, we might only use a few waypoints from the previous path and then generate the rest of the new path based on new data from the car's sensor fusion information.

#### Timing
The simulator runs a cycle every 20 ms (50 frames per second), but your C++ path planning program will provide a new path at least one 20 ms cycle behind. The simulator will simply keep progressing down its last given path while it waits for a new generated path. This means that using previous path data becomes even more important when higher latency is involved. Imagine, for instance, that there is a 500ms delay in sending a new path to the simulator. As long as the new path incorporates a sufficient length of the previous path, the transition will still be smooth. A concern, though, is how accurately we can predict other traffic 1-2 seconds into the future. An advantage of newly generated paths is that they take into account the most up-to-date state of other traffic.

#### Setting Point Paths with Latency
The C++ path planner will at the very least be one cycle behind the simulator because the C++ program can't receive and send data on the same cycle. As a result, any path that the simulator receives will be from the perspective of a previous cycle. This might mean that by the time a new path reaches the simulator, the vehicle has already passed the first few waypoints on that path. Luckily you don't have to worry about this too much. The simulator has built-in tools to deal with this timing difference. The simulator actually expects the received path to be a little out of date compared to where the car is, and the simulator will consider which point on the received path is closest to the car and adjust appropriately.


## Highway Map
Inside data/highway_map.csv there is a list of waypoints that go all the way around the track. The track contains a total of 181 waypoints, with the last waypoint mapping back around to the first. The waypoints are in the middle of the double-yellow dividing line in the center of the highway. The track is 6945.554 meters around (about 4.32 miles). If the car averages near 50 MPH, then it should take a little more than 5 minutes for it to go all the way around the highway. The highway has 6 lanes total - 3 heading in each direction. Each lane is 4 m wide and the car should only ever be in one of the 3 lanes on the right-hand side. The car should always be inside a lane unless doing a lane change.

#### Waypoint Data
Each waypoint has an (x,y) global map position, and a Frenet s value and Frenet d unit normal vector (split up into the x component, and the y component). The s value is the distance along the direction of the road. The first waypoint has an s value of 0 because it is the starting point. The d vector has a magnitude of 1 and points perpendicular to the road in the direction of the right-hand side of the road. The d vector can be used to calculate lane positions. For example, if you want to be in the left lane at some waypoint just add the waypoint's (x,y) coordinates with the d vector multiplied by 2. Since the lane is 4 m wide, the middle of the left lane (the lane closest to the double-yellow dividing line) is 2 m from the waypoint. If you would like to be in the middle lane, add the waypoint's coordinates to the d vector multiplied by 6 = (2+4), since the center of the middle lane is 4 m from the center of the left lane, which is itself 2 m from the double-yellow dividing line and the waypoints.

#### Converting Frenet Coordinates
We have included a helper function, getXY, which takes in Frenet (s,d) coordinates and transforms them to (x,y) coordinates.

#### Interpolating Points
If you need to estimate the location of points between the known waypoints, you will need to "interpolate" the position of those points. Here is a great and easy to setup and use spline tool for C++, contained in just a single header file. A really helpful resource for creating smooth trajectories was using the spline function, and it is really easy to use.


## Sensor Fusion
It's important that the car doesn't crash into any of the other vehicles on the road, all of which are moving at different speeds around the speed limit and can change lanes. The sensor_fusion variable contains all the information about the cars on the right-hand side of the road. The data format for each car is: [ id, x, y, vx, vy, s, d]. The id is a unique identifier for that car. The x, y values are in global map coordinates, and the vx, vy values are the velocity components, also in reference to the global map. Finally s and d are the Frenet coordinates for that car. The vx, vy values can be useful for predicting where the cars will be in the future. For instance, if you were to assume that the tracked car kept moving along the road, then its future predicted Frenet s value will be its current s value plus its (transformed) total velocity (m/s) multiplied by the time elapsed into the future (s).

## Changing Lanes
The last consideration is how to create paths that can smoothly changes lanes. Any time the ego vehicle approaches a car in front of it that is moving slower than the speed limit, the ego vehicle should consider changing lanes. The car should only change lanes if such a change would be safe, and also if the lane change would help it move through the flow of traffic better. For safety, a lane change path should optimize the distance away from other traffic. For comfort, a lane change path should also result in low acceleration and jerk. The acceleration and jerk part can be solved from linear equations for s and d functions. The provided Eigen-3.3 library can solve such linear equations. The getXY helper function can transform (s,d) points to (x,y) points for the returned path.


## Results
The car is safely navigated around a virtual highway with other traffic. 
- The car's localization and sensor fusion data are provided, and there is also a sparse map list of waypoints around the highway. 
- The car goes as close as possible to the 50 MPH speed limit. 
- The car avoids hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. 
- The car is able to make one complete loop around the 6946m highway. 
- Since the car is trying to go 50 MPH, it takes a little over 5 minutes to complete 1 loop. 
- Also the car does not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.


