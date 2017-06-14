# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

My MPC simulation in action!

https://youtu.be/b4yMeSp99vo

# Rubric Questions:
## The Model:
The model follows the lecture code.  This model is based on the following states: x position, y position, psi (vehicle angle), velocity, cross track error (cte) and psi error.  The cross tracking error is calculated as the distance from the middle of the road, as is provided by waypoints.  The psi error is calculated as the current direction compared to the desired direction (going down the road in the direction of the waypoints).

The actuators for the system are the throttle, which can go from -1 (full reverse) to +1 (max acceleration) and steering, which can go left and right from -.46 to +.46.  The steering is dimensionless, so there are conversions in the code to make the steering angle work with the trigonometry used in calculating states.

The update equations are meant to build subsequent states based on the current state.  For example, the subsequent x and y positions are calculated based on the previous states' x and y position, velocity and turn angle.  

The model is solved for the lowest "cost" path.  This is handled by the Ipopt library, but to setup the library to solve for it, a cost function needs to be provided.  The cost is calculated by multiplying undesirable state conditions with suitable coefficients.  These coefficients need to be tuned to achieve suitable performance.  These are shown in MPC.cpp, but these undesirable costs are cross tracking error (CTE), psi error, velocity error, changes to the steering angle or throttle, and extreme control efforts (max throttle, or steering).  By adding these to the cost function, the solver finds a set of states that minimizes the cost function.  In practice, this pushes the car to stay close to the desired line, at the desired speed, without dramatic changes in steering or braking/acceleration.

Once the solution is found by the solver, the control effort from the first state prediction is used for the next steering and throttle command for the vehicle.

## Timesteps and Predictions
This implementation uses a timestep of .1 seconds and 8 state predictions.  The target speed is 100 mph.  I tried higher state prediction counts (10, 15, 20), but for high angle turns, this created oversteering conditions where it would continue turning in the direction of the corner AFTER it came out of the corner.  At higher speeds, the higher number of iterations is attempting to predict greater distances.  Also at high speeds, these greater distance predictions are less accurate, so they're not useful.

I also attempted faster state estimation of .05 seconds.  This caused a lot of jitter / noise.  I think it probably could have been tuned out; and it makes sense that if you had a race car you'd probably be interested in shorter iterations.  I also tried longer time of .2 seconds, but then the steps seemed to be too far apart and caused some slow response times for sharper corners.

## Dealing with Latency

The simulator system has a 100ms latency built into it to add a little realism.  To handle this, I took a very straight forward approach.  When receiving new telemetry, I updated the actual x and y position of the vehicle to a predicted state.  The predicted x and y values are used for all the equations (including the starting state of the MPC state iterations).  To predict the latency based positions, I take the current x and y values, and shift them by velocity * dt * cos(psi) for x and v*dt*sin(psi) for y.  For dt, I started with .1, but I found that overdriving it to .11 seemed to work well.  In my mind, this also accounts for any minor processing and execution delays.

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
* [uWebSockets](https://github.com/uWebSockets/uWebSockets) == 0.14, but the master branch will probably work just fine
  * Follow the instructions in the [uWebSockets README](https://github.com/uWebSockets/uWebSockets/blob/master/README.md) to get setup for your platform. You can download the zip of the appropriate version from the [releases page](https://github.com/uWebSockets/uWebSockets/releases). Here's a link to the [v0.14 zip](https://github.com/uWebSockets/uWebSockets/archive/v0.14.0.zip).
  * If you have MacOS and have [Homebrew](https://brew.sh/) installed you can just run the ./install-mac.sh script to install this.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/CarND-MPC-Project/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.



## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.
