# Path planning project
[Check out final video](https://www.youtube.com/watch?v=-EomKbKOcb4)

[//]: # (Image References)

[prediction1]: ./images/predictor_001.png
[prediction2]: ./images/predictor_002.png
[prediction3]: ./images/predictor_003.png
[behavior]: ./images/behavior_004.png

### Goals
In this project my goal is to safely navigate around a virtual highway with other traffic that is driving +- 10 MPH of the 50 MPH speed limit.

### Modules
#### Prediction
Prediction module takes data from telemetry as argument and transform them to coordinate system of ego car. Each observable car can occupy 6 slots in belief (on the picture it occupies 3 yellow slots). Nothing to predict here, put this layer at position 0 (layer t=0)

Then predictor uses speeds of ego and observable cars to predict their positions 500ms later (layer t=1). And repeats this predictions up 30 seconds in future ( see src/config.h ```PREDICTOR_TIME_SLOTS```, ```PREDICTOR_TIME_QUANT``` )

![alt text][prediction1]

We don't need whole 3d array of predictions to plan trajectory, let's slice it: for all ```t```, for all ```lanes``` take slice with ```s=0```. Result on the picture below. Collision is still detected at ``` t = [ 8, 9, 10 ] ```

![alt text][prediction3]

Predictor fills this matrix by preferred speed. Empty slots filled by 22m/s (Speed limit), occupied slots filled by speed of observable. Few slots after occupation smoothly increase speed to speed limit ( yellow slots )

#### Behavioral planner
This module implements dynamic programming approach. Algorithm calculates DP matrix based on matrix returned from prediction module. DP state is summ of preferred velocities up to range of prediction. Algorithm tries to select lane with maximum value ( kind of 'longest path algorithm' ). It also checks possible lane change maneuvers for each cell of DP matrix ( it's complicated to show on the scheme, but it standart approach of DP).

![alt text][behavior]

To prevent aggressive lane changes i've introduced penalty for lane change maneuver (see ```src/config.h``` ```MANEUVER_PENALTY```)

##### Change speed for maneuver
In case of change lane maneuver algorithm will start change speed immediately. Preferred speed calculated as weighted summ for speed of current lane and speed of target lane:

![Weighted summ](https://latex.codecogs.com/gif.latex?v%20%3D%20%5Calpha%20%5Ccdot%20v_%7Bcurrent%7D%20&plus;%20%281%20-%20%5Calpha%29%20%5Ccdot%20v_%7Btarget%7D)

#### Trajectory generator
This module takes preferred lane and preferred speed from behavioral planner and calculates 25 trajectory points for next 500ms. In case of acceleration/deacceleration distance will be changed between each pair of trajectory points. Distance between points calculated according to restrictions for max acceleration and jerk.

To prevent aggressive lane changes this module has `BUSY` mode. This mode means that trajectory generator didn't finish lane change and will not accept new values of preferred lane (see ```src/trajectory.cpp```, line 15).

### Conclusion
It's more or less robust to pass more than 30 miles, but there are few points which could be improved:
- Prediction. Calculate values for slots after occupied slots according to difference of ego and observable cars speeds.
- Behavioral planner. Analyze more than one slice of belief. Means to check maneuvers with acceleration and deacceleration.
- Trajectory generator. Now it uses kind of fixed acceleration value -> calculate it dinamically according to difference between current and target speeds.
