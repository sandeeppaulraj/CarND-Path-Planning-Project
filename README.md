# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
   
### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates. 

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

## Reflection

I leveraged the project helper video and built the project one step at a time just like the video.
This really helped in understanding the various building blocks. Thereafter i essentially added
sequences for three main pieces of logic which i mention below.

### Detect the Car Lane

I initialize the car lane variable to zero. Based on the Frenet 'd' value, i go about trying to
gauge the lane in which the car currently is. I used the logic below.
I was skeptical about assigning a value at a 'd' of 0, 4, 8 and 12 but these did not effect the 
simulation run. At exactly d equal to 4, it is one's judgement whether the car is in lane 0 or 1.
There are other such cases as well. When the 'd' value is less than zero or greater than twelve,
i essentially go back to the top of the for loop.

```
    if ((d >= 0 ) && (d <= 4))
        car_lane = 0;
    else if ((d > 4) && (d <= 8))
        car_lane = 1;
    else if ((d > 8) && (d <= 12))
        car_lane = 2;
    else if (d < 0)
        continue;
    else if (d > 12)
        continue;
```

### Check for Other Cars

Below i try to gauge whwther the car is in front, left or right.
The logic is an extension of what was provided in the project.
To see if there is a car on the left or the right, what i do is to ensure that there
is always a gap of a safe 30 metres. Anything not within this range means that there
is a car and this notion is used later in lane changing decisions.

```
    if (car_lane == lane) {	
		if ((check_car_s > car_s) && ((check_car_s - car_s) < 30)) {
			car_front = true;
		}
	} else if ((car_lane - lane) == -1) {
        if (((car_s + 30) > check_car_s) && ((car_s - 30) < check_car_s)) {
            car_left = true;
        } 
	} else if ((car_lane - lane) == 1) {
        if (((car_s + 30) > check_car_s) && ((car_s - 30) < check_car_s)) {
            car_right = true;
        } 
    }
```

### Lane Changing

The logic is simple and we have to be cognizant of the fact that we start of at a speed of zero.
Hence, we slowly increase speed to avoid jerk. This is done till the maximum allowed.
This condition of increasing speed can be hit when the car slows down to avoid collision as well.

If a car is in front, there are three separate actions that can can be taken.
If it is safe to move to the left if there is no other car on the left and the car lane
is not zero then go ahead and decrease the lane number.
Similarly, if it is safe to move to the right if there is no other car on the right
and the car lane is not two then go ahead and increase the lane number.
We can only increase or decrease the lane, one lane at a time.
We DO NOT change two lanes at once.
If the car is not able to change lanes then to avoid a collision reduce the same and
stay in the same lane. 

```
    if (car_front) {
	    /* If a car is in front and if there is no car on the left
		 * and the lane is not 0, then move one lane to the left.
         */				 
        if (!car_left && (lane != 0))
            lane--;
		/* If a car is in front and if there is no car on the right
		 * and the lane is not 2, then move one lane to the right.
         */
        else if (!car_right && (lane != 2))
            lane++;
		/* since lane cannot be changed to avoid collision
		 * reduce speed */
        else   
		    ref_vel -= 0.224;
	} else if (ref_vel < 49.5) {
		/* Initialized to 0 and gradually increase */
		/* this condition can be hit other times as well
		 * when changing lanes or avoiding traffic so speed
		 * up when possible
		 */
		ref_vel += 0.224;
	}
```

### Compilation and Testing

The Project was built with the Docker Image on Windows and the simulator was on Windows as well.
The car drove without any issues for multiple laps.
There were no major changes made to the source file and hence, i expect this to work without and issues
on any platform. I don't expect any build issues since the changes are not platform dependent.

The build directory has the binary that i built on Docker that i used to test along with the sim.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

