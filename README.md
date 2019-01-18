# Path planning project


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

#### Behavioral planning
This module implements dynamic programming approach. Algorithm calculates DP matrix based on matrix returned from prediction module. DP state is summ of preferred velocities up to range of prediction
![alt text][behavior]
