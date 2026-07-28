# SLAM (Simultaneous Localization and Mapping) - Complete Exam Notes
## Embedded Vision EST Preparation

---

# Question
## A robotic device has 3 degrees of freedom and is controlled with acceleration in x, y and z directions. Consider that the movement in each axis is independent of each other. Provide the motion and measurement model for implementing the system with SLAM. Consider that the sensor measures x, y and z position.

---

# Introduction

The robotic device has:

- 3 Degrees of Freedom (DOF)
- Motion along:
  - x-axis
  - y-axis
  - z-axis

The robot is controlled using:

- Acceleration in x direction
- Acceleration in y direction
- Acceleration in z direction

The problem states that:

> Motion along each axis is independent.

Therefore,

- Motion in x, y and z can be modeled separately.
- There is no coupling between the axes.
- The system becomes a linear motion model.

The sensor directly measures:

- x position
- y position
- z position

To implement SLAM (or EKF-SLAM), two mathematical models are required:

1. Motion Model (Prediction Model)
2. Measurement Model (Observation Model)

---

# Step 1: Define the State Vector

Since acceleration affects velocity and velocity affects position, the robot state should include both position and velocity.

The state vector is:

\[
X_k =
\begin{bmatrix}
x\\
y\\
z\\
v_x\\
v_y\\
v_z
\end{bmatrix}
\]

### Meaning

| Variable | Meaning |
|----------|----------|
| x | Position along x-axis |
| y | Position along y-axis |
| z | Position along z-axis |
| vx | Velocity along x-axis |
| vy | Velocity along y-axis |
| vz | Velocity along z-axis |

---

# Step 2: Define the Control Input

The robot is controlled using acceleration.

Control vector:

\[
u_k=
\begin{bmatrix}
a_x\\
a_y\\
a_z
\end{bmatrix}
\]

where

- ax = acceleration in x direction
- ay = acceleration in y direction
- az = acceleration in z direction

---

# Assumptions

The following assumptions are made:

- Motion is linear.
- Each axis moves independently.
- Acceleration remains constant during one sampling interval.
- Process noise is Gaussian.

---

# Step 3: Motion Model (Prediction Model)

The prediction model estimates the robot's next state using

- previous state
- acceleration inputs

Using the basic equations of kinematics,

## Position Update

### X-axis

\[
x_k=x_{k-1}+v_{x,k-1}\Delta t+\frac12a_x\Delta t^2
\]

### Y-axis

\[
y_k=y_{k-1}+v_{y,k-1}\Delta t+\frac12a_y\Delta t^2
\]

### Z-axis

\[
z_k=z_{k-1}+v_{z,k-1}\Delta t+\frac12a_z\Delta t^2
\]

---

## Velocity Update

### X-axis

\[
v_{x,k}=v_{x,k-1}+a_x\Delta t
\]

### Y-axis

\[
v_{y,k}=v_{y,k-1}+a_y\Delta t
\]

### Z-axis

\[
v_{z,k}=v_{z,k-1}+a_z\Delta t
\]

---

# Complete Motion Model

The complete prediction equation is

\[
X_k=AX_{k-1}+Bu_k+w_k
\]

where

- A = State Transition Matrix
- B = Control Matrix
- wk = Process Noise

---

# Step 4: State Transition Matrix (A)

Since all three axes are independent,

\[
A=
\begin{bmatrix}
1&0&0&\Delta t&0&0\\
0&1&0&0&\Delta t&0\\
0&0&1&0&0&\Delta t\\
0&0&0&1&0&0\\
0&0&0&0&1&0\\
0&0&0&0&0&1
\end{bmatrix}
\]

### Interpretation

The matrix updates

- current position using velocity
- current velocity using previous velocity

---

# Step 5: Control Matrix (B)

\[
B=
\begin{bmatrix}
\frac12\Delta t^2&0&0\\
0&\frac12\Delta t^2&0\\
0&0&\frac12\Delta t^2\\
\Delta t&0&0\\
0&\Delta t&0\\
0&0&\Delta t
\end{bmatrix}
\]

### Interpretation

Acceleration contributes to

- position update
- velocity update

---

# Step 6: Process Noise

The prediction is never perfect because of

- wheel slip
- actuator errors
- disturbances
- modeling inaccuracies

Therefore,

\[
w_k\sim N(0,Q)
\]

where

Q = Process Noise Covariance Matrix

---

# Step 7: Measurement Model

The sensor directly measures

- x position
- y position
- z position

Therefore,

Measurement Vector:

\[
Z_k=
\begin{bmatrix}
x\\
y\\
z
\end{bmatrix}
\]

---

# Measurement Equation

The observation model is

\[
Z_k=HX_k+v_k
\]

where

- H = Measurement Matrix
- vk = Measurement Noise

---

# Step 8: Measurement Matrix (H)

Since only positions are observed,

\[
H=
\begin{bmatrix}
1&0&0&0&0&0\\
0&1&0&0&0&0\\
0&0&1&0&0&0
\end{bmatrix}
\]

### Interpretation

The measurement matrix selects only

- x
- y
- z

from the complete state vector.

Velocities are estimated internally by the filter.

---

# Step 9: Measurement Noise

Sensors are never perfectly accurate.

Noise arises due to

- calibration errors
- environmental disturbances
- sensor limitations

Measurement noise is represented as

\[
v_k\sim N(0,R)
\]

where

R = Measurement Noise Covariance Matrix

---

# Step 10: EKF / Kalman Filter Implementation

The complete estimation process consists of two phases.

---

# Phase 1: Prediction

Using the motion model,

## State Prediction

\[
\hat X_k^-=A\hat X_{k-1}+Bu_k
\]

Predicts

- position
- velocity

---

## Covariance Prediction

\[
P_k^-=AP_{k-1}A^T+Q
\]

Updates the uncertainty after prediction.

---

# Phase 2: Correction

The sensor now measures

- x
- y
- z

The filter compares

Predicted Position

vs

Measured Position

---

## Innovation

Innovation is the difference between

- predicted measurement
- actual measurement

\[
y_k=Z_k-H\hat X_k^-
\]

---

## Kalman Gain

Kalman Gain decides

> How much should the prediction be corrected?

\[
K=P_k^-H^T(HP_k^-H^T+R)^{-1}
\]

---

## State Update

The corrected state estimate is

\[
\hat X_k=\hat X_k^-+Ky_k
\]

---

## Covariance Update

The uncertainty after correction becomes

\[
P_k=(I-KH)P_k^-
\]

---

# Flow of the Complete Algorithm

```text
Initial State

        │

        ▼

Apply Acceleration Inputs

        │

        ▼

Prediction using Motion Model

        │

        ▼

Predict Position and Velocity

        │

        ▼

Sensor Measures x, y, z

        │

        ▼

Compute Innovation

        │

        ▼

Calculate Kalman Gain

        │

        ▼

Correct Robot State

        │

        ▼

Update Covariance Matrix
```

---

# Important Observations

## Observation 1

Since the motion in x, y and z is independent,

- matrices are sparse
- implementation becomes simpler
- computation becomes faster

---

## Observation 2

The motion model is linear because

- acceleration is the control input
- no nonlinear rotation exists

Therefore,

A standard Kalman Filter can solve this problem.

If nonlinear motion (such as rotation using sinθ and cosθ) is introduced, an Extended Kalman Filter (EKF) would be required.

---

## Observation 3

The sensor measures only

- x
- y
- z

Velocities are never measured directly.

They are estimated internally by the filter.

---

# Final Answer (Exam Conclusion)

For the given robotic device, the SLAM implementation consists of:

- A state vector containing position and velocity along x, y and z axes.
- A control vector consisting of accelerations in each axis.
- A motion model based on constant acceleration kinematics to predict the next state.
- A measurement model where the sensor directly observes x, y and z positions.
- The Kalman Filter/EKF first predicts the robot state using acceleration inputs and then corrects the prediction using sensor measurements while minimizing estimation uncertainty.

This combination of prediction and correction enables accurate localization and forms the basis of the SLAM implementation.
