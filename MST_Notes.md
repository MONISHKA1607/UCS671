# Embedded Vision MST Notes
# Camera Calibration • Stereo Vision • Stereo Correspondence • Optimization

---

# Table of Contents

1. Camera Calibration
2. Monocular Camera Calibration
3. Intrinsic & Extrinsic Parameters
4. Stereo Vision
5. Stereo Camera Model
6. Stereo Depth Estimation
7. Stereo Correspondence & Disparity
8. Feature Detection & Matching
9. Optimization & Model Formats
10. Model Conversion
11. TensorRT Optimization
12. Configuration Files
13. Exam-Oriented Questions

---

# 1. Camera Calibration

## Definition

Camera calibration is the process of estimating the **camera matrix (Projection Matrix P)** which maps a 3D world point to a 2D image point.

Purpose:

- Estimate camera intrinsic parameters
- Estimate camera extrinsic parameters
- Remove lens distortion
- Required for stereo vision, SLAM, pose estimation and depth estimation

---

## Projection Equation

\[
s
\begin{bmatrix}
u\\
v\\
1
\end{bmatrix}
=
K[R|t]
\begin{bmatrix}
X\\
Y\\
Z\\
1
\end{bmatrix}
\]

where

- (X,Y,Z) → World Coordinates
- (u,v) → Pixel Coordinates
- K → Intrinsic Matrix
- R,t → Extrinsic Parameters
- s → Scale factor

---

## Camera Matrix

\[
P=K[R|t]
\]

---

# 2. Monocular Camera Calibration

## Steps

### Step 1

Capture multiple chessboard images from different orientations.

---

### Step 2

Detect chessboard corners.

Example:

```python
cv2.findChessboardCorners()
```

---

### Step 3

Generate known 3D world coordinates.

Example

```
(0,0,0)
(1,0,0)
(2,0,0)
...
```

---

### Step 4

Find corresponding 2D image coordinates.

---

### Step 5

Compute Projection Matrix

\[
P=K[R|t]
\]

using Least Squares or SVD.

---

### Step 6

Extract intrinsic and extrinsic parameters using RQ factorization.

---

### Step 7

Correct lens distortion.

```python
cv2.undistort()
```

---

# Mathematical Formulation

Projection matrix

\[
P=
\begin{bmatrix}
p_{11}&p_{12}&p_{13}&p_{14}\\
p_{21}&p_{22}&p_{23}&p_{24}\\
p_{31}&p_{32}&p_{33}&p_{34}
\end{bmatrix}
\]

Expanded equations

\[
su=p_{11}X+p_{12}Y+p_{13}Z+p_{14}
\]

\[
sv=p_{21}X+p_{22}Y+p_{23}Z+p_{24}
\]

\[
s=p_{31}X+p_{32}Y+p_{33}Z+p_{34}
\]

Solve using

- Least Squares
- Singular Value Decomposition (SVD)

---

# Advantages

- Easy formulation
- Closed-form solution

---

# Limitations

- Does not estimate distortion
- Needs many correspondences
- Sensitive to noise

---

# 3. Intrinsic & Extrinsic Parameters

## Intrinsic Parameters

Describe internal camera characteristics.

Intrinsic Matrix

\[
K=
\begin{bmatrix}
f_x&0&c_x\\
0&f_y&c_y\\
0&0&1
\end{bmatrix}
\]

Parameters

- Focal Length
- Principal Point
- Skew

---

## Extrinsic Parameters

Describe camera pose.

\[
[R|t]
\]

where

- R → Rotation Matrix
- t → Translation Vector

---

Overall Camera Matrix

\[
P=K[R|t]
\]

---

# 4. Stereo Vision

## Definition

Stereo vision uses two cameras placed at a fixed baseline to estimate depth.

---

## Assumptions

- Two identical cameras
- Parallel optical axes
- Images are rectified

---

## Components

- Left Camera
- Right Camera
- Baseline (b)
- Focal Length (f)
- Image Planes

---

# Stereo Geometry

A 3D point projects onto

- Left image
- Right image

Difference in projections is called **Disparity**.

---

# 5. Stereo Camera Model

Projection equations

\[
\frac{Z}{f}=\frac{X}{x_L}
\]

\[
\frac{Z}{f}=\frac{X-b}{x_R}
\]

---

Rearranging

\[
Zx_L=fX
\]

\[
Zx_R=f(X-b)
\]

Subtract

\[
Z(x_L-x_R)=fb
\]

---

Define Disparity

\[
d=x_L-x_R
\]

---

Depth Equation

\[
Z=\frac{fb}{d}
\]

---

3D Coordinates

\[
X=\frac{Zx_L}{f}
\]

\[
Y=\frac{Zy_L}{f}
\]

---

Important Observation

Higher disparity

↓

Closer object

Lower disparity

↓

Farther object

---

# Stereo Pipeline

Stereo Images

↓

Calibration

↓

Rectification

↓

Disparity

↓

Depth

↓

3D Reconstruction

---

# 6. Stereo Correspondence

## Definition

Stereo correspondence is the process of finding the same 3D point in both images.

---

## Disparity

\[
d=x_L-x_R
\]

---

## Epipolar Constraint

Corresponding points lie on the same horizontal line.

Search reduces

2D

↓

1D

---

# Stereo Matching Methods

## Brute Force

Compare every pixel.

Complexity

High

---

## Block Matching

Compare image patches.

Measures

### SSD

\[
SSD=\sum(I_L-I_R)^2
\]

### NCC

\[
NCC=\frac{\sum I_LI_R}
{\sqrt{\sum I_L^2\sum I_R^2}}
\]

---

## Feature Matching

Use feature descriptors.

Example

```python
orb=cv2.ORB_create()
kp,des=orb.detectAndCompute(img,None)
```

---

# 7. Feature Detection

Common Features

## Point Features

- Harris
- Shi-Tomasi
- ORB
- SIFT
- SURF
- BRISK

---

## Edge Features

Useful for low texture regions.

---

## Texture Features

Useful for repetitive patterns.

---

## Region Features

Compare patches instead of points.

---

# ORB Example

```python
orb=cv2.ORB_create()

kp,des=orb.detectAndCompute(gray,None)

img=cv2.drawKeypoints(gray,kp,img)
```

---

# Feature Matching

```python
matcher=cv2.BFMatcher(
cv2.NORM_HAMMING,
crossCheck=True
)

matches=matcher.match(d1,d2)
```

Sort

```python
matches.sort(key=lambda x:x.distance)
```

---

# Homography

```python
H,mask=cv2.findHomography(
p1,
p2,
cv2.RANSAC
)
```

---

Warp

```python
cv2.warpPerspective(...)
```

---

# 8. Optimization

Optimization improves

- Speed
- Latency
- Memory
- GPU utilization

---

# Frameworks

## TensorFlow

Formats

- .ckpt
- .pb
- SavedModel

---

.ckpt

- Resume Training

.pb

- Inference Only

---

## PyTorch

Format

```
.pth
```

---

## Keras

Format

```
.h5
```

---

# Deployment Formats

## ONNX

Open Neural Network Exchange

Purpose

Framework-independent deployment

Supports

- CPU
- GPU
- Jetson
- Mobile

---

## TorchScript

Optimized PyTorch deployment format.

Supports

- C++
- No Python

Methods

### Script

```python
torch.jit.script(model)
```

Supports loops and conditionals.

---

### Trace

```python
torch.jit.trace(model,input)
```

Only static graphs.

---

## TensorRT

NVIDIA inference optimization engine.

Works on

- Jetson
- NVIDIA GPU

Benefits

- Faster inference
- Lower latency
- Lower memory
- FP16
- INT8

---

# Model Conversion

## PyTorch → ONNX

```python
torch.onnx.export(
model,
dummy_input,
"model.onnx"
)
```

---

## PyTorch → TorchScript

```python
scripted=torch.jit.script(model)

scripted.save("model.pt")
```

---

## Trace

```python
traced=torch.jit.trace(
model,
example_input
)
```

---

# TensorFlow → TensorRT

```python
converter=trt.TrtGraphConverterV2(
input_saved_model_dir="saved_model",
precision_mode=trt.TrtPrecisionMode.FP16
)

converter.convert()

converter.save("trt_model")
```

---

# TensorRT Optimizations

1. Layer Fusion

Conv

↓

BatchNorm

↓

ReLU

↓

Single Layer

---

2. Precision Reduction

FP32

↓

FP16

↓

INT8

---

3. Kernel Optimization

Best CUDA kernel selection.

---

4. Memory Optimization

Reuse buffers.

---

5. Tensor Fusion

Combine tensor operations.

---

# Configuration Files

## ONNX

```yaml
name: "object_detection"

platform: "onnxruntime_onnx"

max_batch_size: 8
```

Input

```yaml
input[
{
name:"images"
dims:[3,640,640]
}
]
```

Output

```yaml
output[
{
name:"boxes"
dims:[100,4]
}
]
```

---

## TorchScript

```yaml
platform:"pytorch_libtorch"
```

---

## TensorRT

```yaml
platform:"tensorrt_plan"
```

GPU

```yaml
instance_group[
{
kind:KIND_GPU
count:1
}
]
```

---

# Deployment Pipeline

PyTorch (.pth)

↓

ONNX (.onnx)

↓

TensorRT (.engine)

↓

Jetson Deployment

---

# Choosing Model Formats

| Scenario | Format |
|-----------|--------|
| Training | PyTorch / TensorFlow |
| Continue Training | .ckpt |
| Inference | .pb |
| Cross-platform deployment | ONNX |
| PyTorch optimization | TorchScript |
| NVIDIA Jetson | TensorRT |

---

# Frequently Asked Exam Questions

## Q1

Explain monocular camera calibration.

Write

- Definition
- Projection Equation
- Camera Matrix
- Steps
- SVD
- Diagram

---

## Q2

Explain stereo camera model.

Write

- Diagram
- Similar triangles
- Derivation
- Disparity
- Depth equation

---

## Q3

Intrinsic vs Extrinsic Parameters

Intrinsic

- Focal Length
- Principal Point

Extrinsic

- Rotation
- Translation

---

## Q4

Depth Derivation

Use

\[
Z=\frac{fb}{d}
\]

---

## Q5

Features for stereo correspondence

- Harris
- SIFT
- SURF
- BRISK
- ORB

---

## Q6

PyTorch to ONNX conversion

```python
torch.onnx.export(...)
```

---

## Q7

TorchScript conversion

```python
torch.jit.script(...)
```

---

## Q8

TensorRT Optimizations

- Layer Fusion
- FP16
- INT8
- Memory Optimization
- Kernel Optimization

---

## Q9

When to use Config Files?

Use configuration files only during deployment (e.g., Triton Inference Server or Jetson deployment), not for model conversion.

---

# Final Revision Flow

Camera Calibration

↓

Projection Matrix

↓

Intrinsic & Extrinsic

↓

Stereo Camera

↓

Disparity

↓

Depth Estimation

↓

Feature Matching

↓

Optimization

↓

ONNX / TorchScript

↓

TensorRT

↓

Jetson Deployment

---

# One-Line Formula Revision

Projection

\[
P=K[R|t]
\]

Depth

\[
Z=\frac{fb}{d}
\]

Disparity

\[
d=x_L-x_R
\]

3D Coordinates

\[
X=\frac{Zx_L}{f}
\]

\[
Y=\frac{Zy_L}{f}
\]

Calibration Goal

Estimate

- K
- R
- t
- P
