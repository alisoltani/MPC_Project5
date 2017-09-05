# Model Predictive Controller Project
[video1]: data/mpc.mp4 "MPC @ 85mph"

The goal of this project is to implement a model predictive controller to autonmously drive a simulated car around a track. This implementation is done in C++ and uses IPOPT and CPPAD libraries to calculate an optimal trajectories for the vehicle.


---
 ## Vehicle model
 The Vehicle Model used in this project is a simple kinematic bicycle model that ignores dynamical effects like friction, torque, and etc. The model structure used is shown in the equations below:
 
![Alt text](data/modelstructure.JPG)

px is the x-position of the vehicle, py is the y-position of the vehicle, psi is the orientation, and v velocity.

The kinemetic model uses the vehicles current x and y positions, its orientation angle psi, velocit v, and calculated cross-track-error cte and psi error epsi. The accelaration a and steering angle delta are the actuator outputs of the model. For simplicity, the accelartion is assumed to be between [-1, 1], where negative values signify braking and positive values accelaration. The steering angle should fall between -25 and +25 degrees (or -0.436332 to 0.436332 radians). Lf is the measured distance between the front of the vehicle and center of gravity provided by Udacity. 

##  Timestamps and frequency 
In this project I started from the initial 10 timestamps with frequency of 0.1 second suggested in the Udacity course. After increasing these values to handle higher speeds it was noticed that the car behaved too conservativly on turns (especially the turn after the bridge where it would sometimes come to a stand stil) due to it trying too hard to find the best path. Using 5 timestamps I noticed that the vehicle was able to handle the track at low speeds, but not at hight speeds and would fall out of the track constantly. 

The final settings used in this project was a 11 timestamps and frequency of 0.125. These values still resulted in conservative turns, but were able to handle the higher speeds of 85 mph.

## Polynomial fitting of the waypoints
The waypoints are preprocessed by transforming them to the vehicle's perspective (main.cpp lines 100-105). This greatly simplifies the fitting process as the vehicle x and y are set to the origin, and the orientation is zero.

## Latency of 100ms
In this project, it is assumed that there is a latency of 100ms between when the controller issues its command until when the actuators actually apply those commands. This means that the MPC designed has to take into account this latency, otherwise when approaching the corners and bends the controller would react to the wrong input. In order to combat this, the timestep was increased and a huge penalty was added to both using the steering actuator (delta) and an additional cost penalizing the combination of steering and velocity (that penalizes steering at high speeds) in line 79 of MPC.cpp helps overcome this latency.

## Discussion
The model predictive controller developed here is capable of handling up to 90 mph, with conservative turns. During tests it seems that increasing the timestamp would greatly effect the result, and having too many points to optimize normally resulted in oscillations and falling out of the track. Too few points meant that the vehicle was not able to react appropriatly and fast enough to turns and bends in the track. 

Finally a video of the car around the track is shown here.
[video][video1]


--
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
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
       +  Some Mac users have experienced the following error:
       ```
       Listening to port 4567
       Connected!!!
       mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
       - object was probably modified after being freed.
       *** set a breakpoint in malloc_error_break to debug
       ```
       This error has been resolved by updrading ipopt with
       ```brew upgrade ipopt --with-openblas```
       per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/).
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

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

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.
