# FCND-Term1-P4-Building-an-Estimator  

## Overview  
This is the project 4 of term 1 in [Flying Car Nanodegree](https://www.udacity.com/course/flying-car-nanodegree--nd787) on Udacity. In this project, you will be developing the estimation portion of the controller used in the CPP simulator.  By the end of the project, your simulated quad will be flying with your estimator and your custom controller (from the previous project [FCND-Term1-P3-Control-of-a-3D-Quadrotor-CPP](https://github.com/ikcGitHub/FCND-Term1-P3-Control-of-a-3D-Quadrotor-CPP))!  

For easy navigation throughout this document, here is an outline:  

 - [Prerequisites](#prerequisites)
 - [Setup Instructions](#setup-instructions)
 - [The tasks](#the-tasks)
 - [Evaluation](#evaluation)
 - [Project Description](#project-description)
 - [Run the project](#run-the-project)
 - [Project Rubric](#project-rubric)

## Prerequisites
To run this project, you need to have the following software installed:  
- [Visual Studio](https://www.visualstudio.com/vs/community/)  

## Setup Instructions

This project will continue to use the C++ development environment you set up in the Controls C++ project.

 1. Clone the repository
 ```
 git clone https://github.com/udacity/FCND-Estimation-CPP.git
 ```

 2. Import the code into your IDE like done in the [Controls C++ project](https://github.com/udacity/FCND-Controls-CPP#development-environment-setup)
 
 3. You should now be able to compile and run the estimation simulator just as you did in the controls project

## The Tasks ##

Once again, you will be building up your estimator in pieces.  At each step, there will be a set of success criteria that will be displayed both in the plots and in the terminal output to help you along the way.

Project outline:

 - [Step 1: Sensor Noise](#step-1-sensor-noise)
 - [Step 2: Attitude Estimation](#step-2-attitude-estimation)
 - [Step 3: Prediction Step](#step-3-prediction-step)
 - [Step 4: Magnetometer Update](#step-4-magnetometer-update)
 - [Step 5: Closed Loop + GPS Update](#step-5-closed-loop--gps-update)
 - [Step 6: Adding Your Controller](#step-6-adding-your-controller)



### Step 1: Sensor Noise ###

For the controls project, the simulator was working with a perfect set of sensors, meaning none of the sensors had any noise.  The first step to adding additional realism to the problem, and developing an estimator, is adding noise to the quad's sensors.  For the first step, you will collect some simulated noisy sensor data and estimate the standard deviation of the quad's sensor.

1. Run the simulator in the same way as you have before

2. Choose scenario `06_NoisySensors`.  In this simulation, the interest is to record some sensor data on a static quad, so you will not see the quad move.  You will see two plots at the bottom, one for GPS X position and one for The accelerometer's x measurement.  The dashed lines are a visualization of a single standard deviation from 0 for each signal. The standard deviations are initially set to arbitrary values (after processing the data in the next step, you will be adjusting these values).  If they were set correctly, we should see ~68% of the measurement points fall into the +/- 1 sigma bound.  When you run this scenario, the graphs you see will be recorded to the following csv files with headers: `config/log/Graph1.txt` (GPS X data) and `config/log/Graph2.txt` (Accelerometer X data).

3. Process the logged files to figure out the standard deviation of the the GPS X signal and the IMU Accelerometer X signal.

4. Plug in your result into the top of `config/6_Sensornoise.txt`.  Specially, set the values for `MeasuredStdDev_GPSPosXY` and `MeasuredStdDev_AccelXY` to be the values you have calculated.

5. Run the simulator. If your values are correct, the dashed lines in the simulation will eventually turn green, indicating you’re capturing approx 68% of the respective measurements (which is what we expect within +/- 1 sigma bound for a Gaussian noise model)

***Success criteria:*** *Your standard deviations should accurately capture the value of approximately 68% of the respective measurements.*

NOTE: Your answer should match the settings in `SimulatedSensors.txt`, where you can also grab the simulated noise parameters for all the other sensors.


### Step 2: Attitude Estimation ###

Now let's look at the first step to our state estimation: including information from our IMU.  In this step, you will be improving the complementary filter-type attitude filter with a better rate gyro attitude integration scheme.

1. Run scenario `07_AttitudeEstimation`.  For this simulation, the only sensor used is the IMU and noise levels are set to 0 (see `config/07_AttitudeEstimation.txt` for all the settings for this simulation).  There are two plots visible in this simulation.
   - The top graph is showing errors in each of the estimated Euler angles.
   - The bottom shows the true Euler angles and the estimates.
Observe that there’s quite a bit of error in attitude estimation.

2. In `QuadEstimatorEKF.cpp`, you will see the function `UpdateFromIMU()` contains a complementary filter-type attitude filter.  To reduce the errors in the estimated attitude (Euler Angles), implement a better rate gyro attitude integration scheme.  You should be able to reduce the attitude errors to get within 0.1 rad for each of the Euler angles, as shown in the screenshot below.

![attitude example](images/attitude-screenshot.png)

In the screenshot above the attitude estimation using linear scheme (left) and using the improved nonlinear scheme (right). Note that Y axis on error is much greater on left.

***Success criteria:*** *Your attitude estimator needs to get within 0.1 rad for each of the Euler angles for at least 3 seconds.*

**Hint: see section 7.1.2 of [Estimation for Quadrotors](https://www.overleaf.com/read/vymfngphcccj) for a refresher on a good non-linear complimentary filter for attitude using quaternions.**


### Step 3: Prediction Step ###

In this next step you will be implementing the prediction step of your filter.


1. Run scenario `08_PredictState`.  This scenario is configured to use a perfect IMU (only an IMU). Due to the sensitivity of double-integration to attitude errors, we've made the accelerometer update very insignificant (`QuadEstimatorEKF.attitudeTau = 100`).  The plots on this simulation show element of your estimated state and that of the true state.  At the moment you should see that your estimated state does not follow the true state.

2. In `QuadEstimatorEKF.cpp`, implement the state prediction step in the `PredictState()` functon. If you do it correctly, when you run scenario `08_PredictState` you should see the estimator state track the actual state, with only reasonably slow drift, as shown in the figure below:

![predict drift](images/predict-slow-drift.png)

3. Now let's introduce a realistic IMU, one with noise.  Run scenario `09_PredictionCov`. You will see a small fleet of quadcopter all using your prediction code to integrate forward. You will see two plots:
   - The top graph shows 10 (prediction-only) position X estimates
   - The bottom graph shows 10 (prediction-only) velocity estimates
You will notice however that the estimated covariance (white bounds) currently do not capture the growing errors.

4. In `QuadEstimatorEKF.cpp`, calculate the partial derivative of the body-to-global rotation matrix in the function `GetRbgPrime()`.  Once you have that function implement, implement the rest of the prediction step (predict the state covariance forward) in `Predict()`.

**Hint: see section 7.2 of [Estimation for Quadrotors](https://www.overleaf.com/read/vymfngphcccj) for a refresher on the the transition model and the partial derivatives you may need**

**Hint: When it comes to writing the function for GetRbgPrime, make sure to triple check you've set all the correct parts of the matrix.**

**Hint: recall that the control input is the acceleration!**

5. Run your covariance prediction and tune the `QPosXYStd` and the `QVelXYStd` process parameters in `QuadEstimatorEKF.txt` to try to capture the magnitude of the error you see. Note that as error grows our simplified model will not capture the real error dynamics (for example, specifically, coming from attitude errors), therefore  try to make it look reasonable only for a relatively short prediction period (the scenario is set for one second).  A good solution looks as follows:

![good covariance](images/predict-good-cov.png)

Looking at this result, you can see that in the first part of the plot, our covariance (the white line) grows very much like the data.

If we look at an example with a `QPosXYStd` that is much too high (shown below), we can see that the covariance no longer grows in the same way as the data.

![bad x covariance](images/bad-x-sigma.PNG)

Another set of bad examples is shown below for having a `QVelXYStd` too large (first) and too small (second).  As you can see, once again, our covariances in these cases no longer model the data well.

![bad vx cov large](images/bad-vx-sigma.PNG)

![bad vx cov small](images/bad-vx-sigma-low.PNG)

***Success criteria:*** *This step doesn't have any specific measurable criteria being checked.*


### Step 4: Magnetometer Update ###

Up until now we've only used the accelerometer and gyro for our state estimation.  In this step, you will be adding the information from the magnetometer to improve your filter's performance in estimating the vehicle's heading.

1. Run scenario `10_MagUpdate`.  This scenario uses a realistic IMU, but the magnetometer update hasn’t been implemented yet. As a result, you will notice that the estimate yaw is drifting away from the real value (and the estimated standard deviation is also increasing).  Note that in this case the plot is showing you the estimated yaw error (`quad.est.e.yaw`), which is drifting away from zero as the simulation runs.  You should also see the estimated standard deviation of that state (white boundary) is also increasing.

2. Tune the parameter `QYawStd` (`QuadEstimatorEKF.txt`) for the QuadEstimatorEKF so that it approximately captures the magnitude of the drift, as demonstrated here:

![mag drift](images/mag-drift.png)

3. Implement magnetometer update in the function `UpdateFromMag()`.  Once completed, you should see a resulting plot similar to this one:

![mag good](images/mag-good-solution.png)

***Success criteria:*** *Your goal is to both have an estimated standard deviation that accurately captures the error and maintain an error of less than 0.1rad in heading for at least 10 seconds of the simulation.*

**Hint: after implementing the magnetometer update, you may have to once again tune the parameter `QYawStd` to better balance between the long term drift and short-time noise from the magnetometer.**

**Hint: see section 7.3.2 of [Estimation for Quadrotors](https://www.overleaf.com/read/vymfngphcccj) for a refresher on the magnetometer update.**


### Step 5: Closed Loop + GPS Update ###

1. Run scenario `11_GPSUpdate`.  At the moment this scenario is using both an ideal estimator and and ideal IMU.  Even with these ideal elements, watch the position and velocity errors (bottom right). As you see they are drifting away, since GPS update is not yet implemented.

2. Let's change to using your estimator by setting `Quad.UseIdealEstimator` to 0 in `config/11_GPSUpdate.txt`.  Rerun the scenario to get an idea of how well your estimator work with an ideal IMU.

3. Now repeat with realistic IMU by commenting out these lines in `config/11_GPSUpdate.txt`:
```
#SimIMU.AccelStd = 0,0,0
#SimIMU.GyroStd = 0,0,0
```

4. Tune the process noise model in `QuadEstimatorEKF.txt` to try to approximately capture the error you see with the estimated uncertainty (standard deviation) of the filter.

5. Implement the EKF GPS Update in the function `UpdateFromGPS()`.

6. Now once again re-run the simulation.  Your objective is to complete the entire simulation cycle with estimated position error of < 1m (you’ll see a green box over the bottom graph if you succeed).  You may want to try experimenting with the GPS update parameters to try and get better performance.

***Success criteria:*** *Your objective is to complete the entire simulation cycle with estimated position error of < 1m.*

**Hint: see section 7.3.1 of [Estimation for Quadrotors](https://www.overleaf.com/read/vymfngphcccj) for a refresher on the GPS update.**

At this point, congratulations on having a working estimator!

### Step 6: Adding Your Controller ###

Up to this point, we have been working with a controller that has been relaxed to work with an estimated state instead of a real state.  So now, you will see how well your controller performs and de-tune your controller accordingly.

1. Replace `QuadController.cpp` with the controller you wrote in the last project.

2. Replace `QuadControlParams.txt` with the control parameters you came up with in the last project.

3. Run scenario `11_GPSUpdate`. If your controller crashes immediately do not panic. Flying from an estimated state (even with ideal sensors) is very different from flying with ideal pose. You may need to de-tune your controller. Decrease the position and velocity gains (we’ve seen about 30% detuning being effective) to stabilize it.  Your goal is to once again complete the entire simulation cycle with an estimated position error of < 1m.

**Hint: you may find it easiest to do your de-tuning as a 2 step process by reverting to ideal sensors and de-tuning under those conditions first.**

***Success criteria:*** *Your objective is to complete the entire simulation cycle with estimated position error of < 1m.*


## Evaluation ##

To assist with tuning of your controller, the simulator contains real time performance evaluation.  We have defined a set of performance metrics for each of the scenarios that your controllers must meet for a successful submission.

There are two ways to view the output of the evaluation:

 - in the command line, at the end of each simulation loop, a **PASS** or a **FAIL** for each metric being evaluated in that simulation
 - on the plots, once your quad meets the metrics, you will see a green box appear on the plot notifying you of a **PASS**
 
 ## Project Description  
- [QuadEstimatorEKF.cpp](./src/QuadEstimatorEKF.cpp): This file contains all of the code for the estimator that you will be developing.
- [QuadEstimatorEKF.txt](./config/QuadEstimatorEKF.txt): This file contains all your estimator gains and other desired tuning.
- [QuadControl.cpp](./src/QuadControl.cpp): This file contains all of the code for the controller that you will be developing.  
- [QuadControlParams.txt](./config/QuadControlParams.txt): This file contains all your control gains and other desired tuning parameters.  
- [README.md](./README.md): Writeup for this project, including setup, running instructions and project rubric addressing.  

## Run the project  
To compile and run the project / simulator, simply click on the green play button at the top of the screen.  When you run the simulator, you should see a single quadcopter, falling down in senario `1_Intro`.  

## Project Rubric  
### 1. Writeup  
#### 1.1 Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf.  
You are reading it!  
### 2. Implemented Estimator
#### 2.1 Determine the standard deviation of the measurement noise of both GPS X data and Accelerometer X data.  
Standard deviation of the measurement noise of both GPS X data and Accelerometer X data were calculated in the [CalculateMeanStdDev.sln](https://github.com/ikcGitHub/FCND-Term1-P4-Building-an-Estimator/tree/ikc-solution/MyAwesomeTool/CalculateMeanStdDev).
Results were stored in [06_SensorNoise.txt](https://github.com/ikcGitHub/FCND-Term1-P4-Building-an-Estimator/blob/ikc-solution/config/06_SensorNoise.txt)  
Here is a quick look.  
- MeasuredStdDev_GPSPosXY = 2  
- MeasuredStdDev_AccelXY = .1  
#### 2.2 Implement a better rate gyro attitude integration scheme in the `UpdateFromIMU()` function.  
Better rate gyro attitude integration was implemented in `UpdateFromIMU()` at [`QuadEstimatorEKF.cpp` Line91-129](./src/QuadEstimatorEKF.cpp#L91-L129)  
#### 2.3 Implement all of the elements of the prediction step for the estimator.  
State prediction step was implemented in `PredictState()` at [`QuadEstimatorEKF.cpp` Line188-219](./src/QuadEstimatorEKF.cpp#L188-L219)  
Calculation on the partial derivative of the body-to-global rotation matrix was implemented in `GetRbgPrime()` at [`QuadEstimatorEKF.cpp` Line243-259](./src/QuadEstimatorEKF.cpp#L243-L259)  
The rest of the prediction step (predict the state covariance forward) was implemented in `Predict()` at [`QuadEstimatorEKF.cpp` Line302-315](./src/QuadEstimatorEKF.cpp#L302-L315)  
#### 2.4 Implement the magnetometer update.  
Magnetometer update was implemented in `UpdateFromMag()` at [`QuadEstimatorEKF.cpp` Line370-388](./src/QuadEstimatorEKF.cpp#L370-L388)  
#### 2.5 Implement the GPS update.  
The EKF GPS Update was implemented in `UpdateFromGPS()` at [`QuadEstimatorEKF.cpp` Line337-351](./src/QuadEstimatorEKF.cpp#L337-L351)  
### 3. Flight Evaluation  
#### 3.1 Meet the performance criteria of each step.  
- Step 1: Sensor Noise  
Video after pluged in the calculated result into the 6_Sensoroise.txt.  
06_SensorNoise  
![06_SensorNoise](./videos/06_SensorNoise.gif)  

- Step 2: Attitude Estimation  
Video after better attitude integration implemented.  
07_AttitudeEstimation  
![07_AttitudeEstimation](./videos/07_AttitudeEstimation.gif)  

- Step 3: Prediction Step  
Video with nothing changed.  
08_PredictState  
![08_PredictState](./videos/08_PredictState.gif)  

Video after `PredictState()` completed.  
09_PredictCovariance_Before_change_GetRbgPrime  
![09_PredictCovariance_Before_change_GetRbgPrime](./videos/09_PredictCovariance_Before_change_GetRbgPrime.gif)  

Video after `GetRbgPrime()` completed.  
09_PredictCovariance_After_change_GetRbgPrime_and_Predict  
![09_PredictCovariance_After_change_GetRbgPrime_and_Predict](./videos/09_PredictCovariance_After_change_GetRbgPrime_and_Predict.gif)  

Video after tuning get done.  
09_PredictCovariance_After_tuning  
![09_PredictCovariance_After_tuning](./videos/09_PredictCovariance_After_tuning.gif)  

- Step 4: Magnetometer Update  
Video after `UpdateFromMag` completed and tuning.
10_MagUpdate  
![10_MagUpdate](./videos/10_MagUpdate.gif)  

- Step 5: Closed Loop + GPS Update  
Video with nothing changed.  
11_GPSUpdate_Before_Changes_Made  
![11_GPSUpdate_Before_Changes_Made](./videos/11_GPSUpdate_Before_Changes_Made.gif)  

Video after using my estimator.  
11_GPSUpdate_After_Using_My_Estimator  
![11_GPSUpdate_After_Using_My_Estimator](./videos/11_GPSUpdate_After_Using_My_Estimator.gif)  

Video after using realistic IMU.  
11_GPSUpdate_After_Using_Realistic_IMU  
![11_GPSUpdate_After_Using_Realistic_IMU](./videos/11_GPSUpdate_After_Using_Realistic_IMU.gif)  

Video after `UpdateFromGPS()` completed.  
11_GPSUpdate_After_Completing_UpdateFromGPS_and_Tuning  
![11_GPSUpdate_After_Completing_UpdateFromGPS_and_Tuning](./videos/11_GPSUpdate_After_Completing_UpdateFromGPS_and_Tuning.gif)  

#### 3.2 De-tune your controller to successfully fly the final desired box trajectory with your estimator and realistic sensors.  
- Step 6: Adding Your Controller  
Video after using my previous controller and getting the control parameters tuned.  
11_GPSUpdate_After_Using_Previous_Controller_and_Parameters  
![11_GPSUpdate_After_Using_Previous_Controller_and_Parameters](./videos/11_GPSUpdate_After_Using_Previous_Controller_and_Parameters.gif)  
