# DeepStream EST Notes (Exam-Oriented)

> **Topic Weightage:** DeepStream is one of the most important EST topics. According to the exam pattern, **any streaming application should be solved using DeepStream**, and you should mention the **source settings, sink settings, and model configuration**.

---

# 1. What is DeepStream?

DeepStream is NVIDIA's GPU-accelerated streaming analytics SDK used for deploying real-time AI applications on Jetson devices.

It is used for:

- Object Detection
- Image Classification
- Segmentation
- Multi-camera Analytics
- Video Streaming Applications

Instead of processing one image at a time, DeepStream processes continuous video streams efficiently using GPU acceleration.

---

# 2. Why Do We Use DeepStream?

Advantages:

- GPU accelerated inference
- Uses TensorRT optimized models
- Low latency
- High FPS
- Supports multiple camera streams
- Production-ready deployment
- Uses GStreamer pipeline internally

---

# 3. DeepStream Pipeline

Every DeepStream application follows this pipeline:

```text
Camera / Video
      ↓
Source
      ↓
Decoder
      ↓
StreamMux
      ↓
Primary GIE (Inference)
      ↓
Tracker (Optional)
      ↓
OSD (Bounding Boxes / Labels)
      ↓
Sink (Display / File / RTSP)
```

Remember this diagram.

---

# 4. DeepStream Docker

Docker Image:

```text
nvcr.io/nvidia/deepstream-l4t:6.0-triton
```

Contents:

- DeepStream SDK
- TensorRT
- CUDA
- Triton Inference Server
- GStreamer Plugins

---

## Check Docker

```bash
docker images
```

---

## Pull Docker

```bash
sudo docker pull nvcr.io/nvidia/deepstream-l4t:6.0-triton
```

---

## Run Docker

```bash
sudo docker run -it --rm --net=host --runtime nvidia \
-v /tmp/.X11-unix/:/tmp/.X11-unix \
-e DISPLAY=$DISPLAY \
nvcr.io/nvidia/deepstream-l4t:6.0-triton
```

---

# 5. Why Can't We Directly Use a PyTorch Model?

DeepStream **cannot directly execute a PyTorch (.pt) model**.

Reason:

- PyTorch models are designed for training and general inference.
- DeepStream performs optimized inference using TensorRT.
- Therefore, the model must first be converted.

Conversion Pipeline:

```text
PyTorch (.pt)
      ↓
ONNX (.onnx)
      ↓
TensorRT Engine (.engine)
      ↓
DeepStream
```

---

# 6. Model Conversion

## Step 1: Export PyTorch to ONNX

```python
torch.onnx.export(model, dummy_input, "model.onnx")
```

---

## Step 2: Convert ONNX to TensorRT Engine

```bash
trtexec --onnx=model.onnx --saveEngine=model.engine
```

The generated `.engine` file is used by DeepStream.

---

# 7. DeepStream Configuration Files

DeepStream generally uses **two configuration files**.

## 1. App Configuration File

Defines the **entire DeepStream pipeline**.

It controls:

- Source
- StreamMux
- Primary GIE
- OSD
- Sink

Example:

```ini
[source0]
...

[streammux]
...

[primary-gie]
config-file=config_infer_primary.txt

[osd]
...

[sink0]
...
```

---

## 2. Model Configuration File

Defines only the **AI inference details**.

It controls:

- TensorRT engine
- Labels
- Network type
- Batch size
- Precision
- Preprocessing

Example:

```ini
[property]
model-engine-file=model.engine
labelfile-path=labels.txt
network-type=1
batch-size=1
network-mode=2
```

---

# Relationship Between App Config and Model Config

The **model config is a subset of the application configuration**.

Hierarchy:

```text
App Config
│
├── Source
├── StreamMux
├── Primary GIE
│       │
│       └── Model Config
│
├── OSD
└── Sink
```

The app config controls the **entire pipeline**, whereas the model config controls **only the inference stage**.

---

# 8. Source Settings

## What is Source?

The source specifies **where the input video comes from**.

Examples:

- USB Camera
- CSI Camera
- Video File
- RTSP Stream

---

## Example

```ini
[source0]
enable=1
type=1
camera-width=1280
camera-height=720
camera-fps-n=30
camera-fps-d=1
```

---

## Source Types

| Type | Input |
|------|-------|
| 1 | Camera |
| 2 | RTSP / URI |
| 3 | Video File |

---

# 9. StreamMux

## Purpose

StreamMux:

- Collects frames
- Resizes frames
- Creates batches
- Sends batches to GPU

Example:

```ini
[streammux]
batch-size=1
width=1280
height=720
```

---

# 10. Primary GIE

Primary GIE = Primary GPU Inference Engine.

This is where the AI model executes.

Example:

```ini
[primary-gie]
enable=1
config-file=config_infer_primary.txt
```

The actual inference details are read from the model config file.

---

# 11. Model Configuration

Example:

```ini
[property]
gpu-id=0
model-engine-file=model.engine
labelfile-path=labels.txt
batch-size=1
network-type=1
network-mode=2
gie-unique-id=1
```

---

## Important Parameters

### model-engine-file

TensorRT engine.

Example:

```ini
model-engine-file=model.engine
```

---

### labelfile-path

Contains class names.

Example:

```text
cat
dog
car
bus
```

---

### batch-size

Number of images processed together.

Example:

```ini
batch-size=1
```

---

### network-type

Defines the type of AI problem.

| Value | Problem |
|--------|----------|
| 0 | Detection |
| 1 | Classification |
| 2 | Segmentation |

---

### network-mode

Precision used.

| Value | Precision |
|--------|------------|
| 0 | FP32 |
| 1 | INT8 |
| 2 | FP16 |

Jetson generally prefers FP16 because it offers faster inference with lower memory usage.

---

### gie-unique-id

Unique identifier for the inference engine.

Used when multiple models are present.

---

# 12. OSD (On Screen Display)

Purpose:

- Draw labels
- Draw bounding boxes
- Display confidence

Example:

```ini
[osd]
enable=1
text-size=15
```

---

# 13. Sink Settings

## What is Sink?

Sink defines **where the processed output goes**.

Possible outputs:

- Display
- Video File
- RTSP Stream

---

## Display Output

```ini
[sink0]
enable=1
type=1
```

---

## Save Video

```ini
[sink0]
enable=1
type=3
output-file=output.mp4
```

---

## RTSP Streaming

```ini
[sink0]
enable=1
type=4
```

---

## Sink Types

| Type | Output |
|------|--------|
| 1 | Display |
| 2 | Fake Sink |
| 3 | Video File |
| 4 | RTSP Stream |

---

# 14. Model Repository Structure

Example:

```text
classifier/
│
├── labels.txt
├── model.engine
└── config_infer_primary.txt
```

---

# 15. Complete DeepStream Pipeline Configuration

```ini
[source0]
enable=1
type=1

[streammux]
batch-size=1
width=1280
height=720

[primary-gie]
enable=1
config-file=config_infer_primary.txt

[osd]
enable=1

[sink0]
enable=1
type=3
output-file=output.mp4
```

---

# 16. Running the Application

```bash
deepstream-app -c app_config.txt
```

DeepStream performs:

1. Capture camera frames
2. Batch frames using StreamMux
3. Run TensorRT inference
4. Draw labels using OSD
5. Save output using Sink

---

# 17. Typical EST Application Question

**Question:**

> Prepare a DeepStream container to run a classification model using camera input and save the output as a video. The model is trained in PyTorch.

## Step 1

Identify the problem.

**Classification**

---

## Step 2

Use NVIDIA DeepStream Docker.

```text
nvcr.io/nvidia/deepstream-l4t:6.0-triton
```

---

## Step 3

Convert the model.

```text
PyTorch (.pt)
      ↓
ONNX (.onnx)
      ↓
TensorRT (.engine)
```

---

## Step 4

Create the model configuration.

```ini
network-type=1
model-engine-file=model.engine
labelfile-path=labels.txt
```

---

## Step 5

Configure the source.

```ini
[source0]
type=1
```

(Camera input)

---

## Step 6

Configure StreamMux.

```ini
batch-size=1
```

---

## Step 7

Configure Primary GIE.

```ini
config-file=config_infer_primary.txt
```

---

## Step 8

Configure Sink.

```ini
[sink0]
type=3
output-file=output.mp4
```

---

## Step 9

Run the application.

```bash
deepstream-app -c app_config.txt
```

---

# 18. Important Differences

| App Config | Model Config |
|------------|--------------|
| Defines the entire DeepStream pipeline | Defines only inference details |
| Contains Source, StreamMux, OSD, Sink | Contains TensorRT engine, labels, preprocessing |
| References the model config | Used by Primary GIE |
| Pipeline configuration | Model configuration |

---

# 19. Final Memory Sheet

```text
DeepStream

Pipeline:
Source → StreamMux → Primary GIE → OSD → Sink

Source = Input

StreamMux = Batch Frames

Primary GIE = AI Inference

OSD = Draw Labels / Bounding Boxes

Sink = Output

App Config
    ↓
Primary GIE
    ↓
Model Config
    ↓
TensorRT Engine

PyTorch
    ↓
ONNX
    ↓
TensorRT
    ↓
DeepStream

Source Types:
1 → Camera
2 → RTSP
3 → Video

Sink Types:
1 → Display
3 → Video File
4 → RTSP

Network Types:
0 → Detection
1 → Classification
2 → Segmentation
```
