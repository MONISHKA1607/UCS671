# Embedded Vision Exam Notes -- Kalman Filter, EKF and SLAM

## 1. Kalman Filter (KF)

### Purpose

Kalman Filter estimates the true state of a dynamic system by
combining: - Motion model (prediction) - Sensor measurements
(correction)

It operates in two stages: 1. Prediction 2. Correction

------------------------------------------------------------------------

## State Vector

For 1D motion:

``` text
x = [ p
      v ]
```

where: - p = position - v = velocity

Control input: - u = acceleration

------------------------------------------------------------------------

## Motion Model

Constant velocity model (same as lecture slides):

x_k = F x\_(k-1) + G u\_(k-1) + w

State transition matrix:

F = \[\[1, Δt\], \[0, 1\]\]

Control matrix:

G = \[\[0\], \[Δt\]\]

Process noise: w \~ N(0,Q)

------------------------------------------------------------------------

## Measurement Model

Sensor measures only position.

y_k = Hx_k + v

H = \[1 0\]

Measurement noise: v \~ N(0,R)

------------------------------------------------------------------------

## Prediction

State prediction: x̂⁻ = Fx̂ + Gu

Covariance prediction: P⁻ = FPFᵀ + Q

------------------------------------------------------------------------

## Correction

Kalman Gain: K = P⁻Hᵀ(HP⁻Hᵀ + R)\^(-1)

State update: x̂ = x̂⁻ + K(y − Hx̂⁻)

Covariance update: P = (I − KH)P⁻

------------------------------------------------------------------------

## KF Workflow

Prediction → Measurement → Kalman Gain → Correction → Repeat

------------------------------------------------------------------------

# 2. Extended Kalman Filter (EKF)

## Why EKF?

KF only works for linear systems.

EKF extends KF to nonlinear systems by linearizing them using a
first-order Taylor expansion.

Motion model:

x_k = f(x,u,w)

Measurement model:

y_k = h(x,v)

------------------------------------------------------------------------

## Linearization

Use first-order Taylor approximation.

The Jacobians replace F and H.

Motion Jacobian: F = ∂f/∂x

Measurement Jacobian: H = ∂h/∂x

------------------------------------------------------------------------

## EKF Algorithm

Prediction: x̂⁻ = f(x̂,u)

P⁻ = FP Fᵀ + LQLᵀ

Correction:

K = P⁻Hᵀ(HP⁻Hᵀ + MRMᵀ)\^(-1)

x̂ = x̂⁻ + K(y − h(x̂))

P = (I − KH)P⁻

------------------------------------------------------------------------

## Jacobian

A Jacobian is the matrix of first-order partial derivatives.

It represents the local slope of a nonlinear function.

------------------------------------------------------------------------

# 3. Stereo Vision

Disparity:

d = xL − xR

Depth:

Z = fB/d

where: - f = focal length - B = baseline - d = disparity

Large disparity → object close

Small disparity → object far

------------------------------------------------------------------------

## Monocular Depth

Z = fH/h

where: - H = real object height - h = image height

Less accurate than stereo.

------------------------------------------------------------------------

# 4. Exam Strategy

## Motion Model

Describe how the robot moves.

## Measurement Model

Describe what the sensor measures.

## Prediction

Predict next state using motion model.

## Correction

Update predicted state using measurements.

------------------------------------------------------------------------

# 5. Common Exam Questions

## Object Following Robot

Use: - Stereo depth - Kalman Filter - Distance error - Robot motion
update

Distance error: e = Z − DesiredDistance

------------------------------------------------------------------------

## Obstacle Avoidance

If Z \< threshold: - Sound alarm - Turn left

------------------------------------------------------------------------

## 3 DOF Robot

State: \[x y z vx vy vz\]\^T

Control: \[ax ay az\]\^T

Measurement: \[x y z\]\^T

Motion model: x_k = Fx\_(k-1) + Gu\_(k-1) + w

Measurement model: y_k = Hx_k + v

Use linear KF because both motion and measurements are linear.

------------------------------------------------------------------------

# 6. KF vs EKF

  Kalman Filter         Extended Kalman Filter
  --------------------- --------------------------------
  Linear systems        Nonlinear systems
  Constant F, H         Jacobian matrices
  No Taylor expansion   First-order Taylor expansion
  Simpler               More computationally expensive

------------------------------------------------------------------------

# 7. Important One-Liners

-   Kalman Filter = Prediction + Correction.
-   Process model predicts future state.
-   Measurement model relates state to sensor readings.
-   Kalman Gain decides how much to trust prediction vs measurement.
-   EKF linearizes nonlinear systems using Jacobians.
-   Stereo vision estimates depth using disparity.
-   SLAM uses prediction and sensor updates to estimate robot pose.

