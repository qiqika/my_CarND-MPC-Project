# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

[image1]: ./vehicle_model.png "formula"

---
## Rubric
### Describes model in detail
1.Model use vehicle model, like Global Kinematic Model, to describe the state, actuators and how the state changes over time based on the previous state and current actuator inputs.
here formula:

![alt text][image1]

note: the delta0 is opposed to real orientation, so sign is negative.


2.MPC is a QP problem and like PID controller. the PID controller could try to compute a control input based on a future error, but without a vehicle model it's unlikely this will be accurate.


### Timestep Length and Elapsed Duration (N & dt)
Timestep Length N and Elapsed Duration dt will effect on constraint function iteration times.And the N determines the number of variables optimized by the MPC.Larger values of dt result in less frequent actuations, which makes it harder to accurately approximate a continuous reference trajectory. This is sometimes called "discretization error".

when N is large, the car's steer change will become more fierce. and in the same N, the smaller dt will make predictive line  shorter.
After trying different values, you should also try to give the reasoning behind your values e.g. explaining why some values work better than others. You can try answering the following questions:

Why smaller dt is better? (finer resolution)
Why larger N isn't always better? (computational time)
How does time horizon (N*dt) affect the predicted path? This relates to the car speed too.
In general, smaller dt gives better accuracy but that will require higher N for given horizon (N*dt). However, increase N will result in longer computational time which effectively increase the latency. The most common choice of values is N=10 and dt=0.1.

### Polynomial Fitting and MPC Preprocessing
After get vehicle coordinates, model use polyfit() function to fit third-degree polynomial line. cte the cross-track error which comes from polyfit value and epsi the orientation error which comes from -atan() of polyfit value.

### Model Predictive Control with Latency
By changing weight of  additions cost in cost function, it will help to control latency. And refer to the jeremy-shannon/CarND-MPC-Project , using (delta*v) cost can reduce cumulative error that may come from latency.

Monitor review: You incorporated latency simulation into your MPC model. Well done! However, this isn't ideal as the latency simulation kicks in only after the first value which is what we really interested about (that's why there are N states but N-1 actuator values because the very first actuator value is for the next timestep). There are a few ways to do this, the most common is to use kinematic equations to predict the states for after 100ms before sending them to MPC.The update can be placed before polynomial fitting using global map coordinate, or after polynomial fitting and use vehicle map coordinate.

If you do the update after polynomial fitting, then this involves 2 steps:

At current time t=0, your car's states are px=0, py=0, psi=0 right after converting to car coordinate. There you calcualte cte and epsi

cte = desired_y - actual_y
    = polyeval(coeffs,px)-py
    = polyeval(coeffs,0) because px=py=0
epsi =  actual psi-desired psi
     = psi - atan(coeffs[1]+coeffs[2]*2*px+...) 
     = -atan(coeffs[1])  because px=psi=0
Now predict all the states for t=latency. You can simplify them further knowing some of the variables are 0.
//  change of sign because turning left is negative sign in simulator but positive yaw for MPC
double delta = -j[1]["steering_angle"]; 
//to convert miles per hour to meter per second, and you should convert ref_v too
v*=0.44704;
psi = delta; // in coordinate now, so use steering angle to predict x and y
px = px + v*cos(psi)*latency; 
py = py + v*sin(psi)*latency;
cte= cte + v*sin(epsi)*latency;
epsi = epsi + v*delta*latency/Lf;
psi = psi + v*delta*latency/Lf;
v = v + a*latency;

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
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

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
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
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

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

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
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
