# Reflection on how to generate paths
### CarND-Path-Planning-Project

## Introduction
The goal of this project is to build a path planner that creates smooth, safe trajectories for the car to follow. The highway track has other vehicles, all going different speeds, but approximately obeying the 50 MPH speed limit. The car transmits its location, along with its sensor fusion data, which estimates the location of all the vehicles on the same side of the road.

#### Point Paths
The path planner should output a list of x and y global map coordinates. Each pair of x and y coordinates is a point, and all of the points together form a trajectory. You can use any number of points that you want, but the x list should be the same length as the y list.

Every 20 ms the car moves to the next point on the list. The car's new rotation becomes the line between the previous waypoint and the car's new location.

The car moves from point to point perfectly, so you don't have to worry about building a controller for this project.

The animation above shows how the car moves and rotates through a given list of points.

#### Velocity
The velocity of the car depends on the spacing of the points. Because the car moves to a new waypoint every 20ms, the larger the spacing between points, the faster the car will travel. The speed goal is to have the car traveling at (but not above) the 50 MPH speed limit as often as possible. But there will be times when traffic gets in the way.

#### Goals
In this project the goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. 
- The car's localization and sensor fusion data will be provided, and there is also a sparse map list of waypoints around the highway. 
- The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. 
- The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. 
- The car should be able to make one complete loop around the 6946m highway. 
- Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. 
- Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.




## Highway Map
Inside data/highway_map.csv there is a list of waypoints that go all the way around the track. The track contains a total of 181 waypoints, with the last waypoint mapping back around to the first. The waypoints are in the middle of the double-yellow dividing line in the center of the highway.

The track is 6945.554 meters around (about 4.32 miles). If the car averages near 50 MPH, then it should take a little more than 5 minutes for it to go all the way around the highway.

The highway has 6 lanes total - 3 heading in each direction. Each lane is 4 m wide and the car should only ever be in one of the 3 lanes on the right-hand side. The car should always be inside a lane unless doing a lane change.

#### Waypoint Data
Each waypoint has an (x,y) global map position, and a Frenet s value and Frenet d unit normal vector (split up into the x component, and the y component).

The s value is the distance along the direction of the road. The first waypoint has an s value of 0 because it is the starting point.

The d vector has a magnitude of 1 and points perpendicular to the road in the direction of the right-hand side of the road. The d vector can be used to calculate lane positions. For example, if you want to be in the left lane at some waypoint just add the waypoint's (x,y) coordinates with the d vector multiplied by 2. Since the lane is 4 m wide, the middle of the left lane (the lane closest to the double-yellow dividing line) is 2 m from the waypoint.

If you would like to be in the middle lane, add the waypoint's coordinates to the d vector multiplied by 6 = (2+4), since the center of the middle lane is 4 m from the center of the left lane, which is itself 2 m from the double-yellow dividing line and the waypoints.

#### Converting Frenet Coordinates
We have included a helper function, getXY, which takes in Frenet (s,d) coordinates and transforms them to (x,y) coordinates.

#### Interpolating Points
If you need to estimate the location of points between the known waypoints, you will need to "interpolate" the position of those points.

In previous lessons we looked at fitting polynomials to waypoints. Once you have a polynomial function, you can use it to interpolate the location of a new point.

There are also other methods you could use. For example, Bezier curve fitting with control points, or spline fitting, which guarantees that the generated function passes through every point.

Here is a great and easy to setup and use spline tool for C++, contained in just a single header file. A really helpful resource for creating smooth trajectories was using the spline function, and it is really easy to use.


## Sensor Fusion
It's important that the car doesn't crash into any of the other vehicles on the road, all of which are moving at different speeds around the speed limit and can change lanes.

The sensor_fusion variable contains all the information about the cars on the right-hand side of the road.

The data format for each car is: [ id, x, y, vx, vy, s, d]. The id is a unique identifier for that car. The x, y values are in global map coordinates, and the vx, vy values are the velocity components, also in reference to the global map. Finally s and d are the Frenet coordinates for that car.

The vx, vy values can be useful for predicting where the cars will be in the future. For instance, if you were to assume that the tracked car kept moving along the road, then its future predicted Frenet s value will be its current s value plus its (transformed) total velocity (m/s) multiplied by the time elapsed into the future (s).

## Changing Lanes
The last consideration is how to create paths that can smoothly changes lanes. Any time the ego vehicle approaches a car in front of it that is moving slower than the speed limit, the ego vehicle should consider changing lanes.

The car should only change lanes if such a change would be safe, and also if the lane change would help it move through the flow of traffic better.

For safety, a lane change path should optimize the distance away from other traffic. For comfort, a lane change path should also result in low acceleration and jerk. The acceleration and jerk part can be solved from linear equations for s and d functions. 

The provided Eigen-3.3 library can solve such linear equations. The getXY helper function can transform (s,d) points to (x,y) points for the returned path.





## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. 
- The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. 
- The vector going from a point to the next point in the list dictates the angle of the car. 
- Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. 
- The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. 
- Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. 
- During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. 
- previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. 
- You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.


