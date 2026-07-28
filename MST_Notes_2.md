# Embedded Vision MST Notes - Jetson Docker, Deployment & TAO Toolkit
> **Exam-Oriented Notes (Based on Ma'am's Pattern)**

---

# Table of Contents

1. Connecting to Jetson
2. Docker Basics
3. Docker Images vs Containers
4. Running Docker Containers
5. Dockerfile
6. Building & Modifying Docker Images
7. Adding Data to Docker
8. AI Deployment on Jetson
9. Classification Deployment
10. Detection Deployment
11. Segmentation Deployment
12. Case Study Questions
13. Docker Commands
14. TAO Toolkit
15. Frequently Asked Deployment Questions

---

# 1. Connecting to Jetson

## SSH Connection

```bash
ssh nvidia@192.168.55.1
```

Default credentials

```
Username : nvidia
Password : nvidia
```

### Why SSH?

- Remote login
- Execute commands
- Run Docker
- Deploy applications

---

## Common SSH Error

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED
```

Solution

```bash
ssh-keygen -R 192.168.55.1
```

---

## SCP (File Transfer)

### Jetson → Laptop

```bash
scp nvidia@192.168.55.1:/home/nvidia/data ./
```

### Laptop → Jetson

```bash
scp ./data nvidia@192.168.55.1:/home/nvidia/
```

---

# 2. Docker Basics

## What is Docker?

Docker is a **containerization platform** that packages:

- Code
- Libraries
- Dependencies
- Runtime

inside isolated environments called **containers**.

---

## Docker Image vs Container

| Docker Image | Docker Container |
|--------------|-----------------|
| Template | Running instance |
| Read only | Read-write |
| Blueprint | Execution |

Example

```
Image
 ↓
Container
```

---

# 3. Verifying Docker

Check installed images

```bash
sudo docker images
```

---

## Docker Daemon Error

```
Cannot connect to Docker daemon
```

Reason

- Docker data on SD card not mounted

---

## Mount SD Card

```bash
sudo mount /dev/mmcblk1p1 /sdcard
```

Verify

```bash
df -h
```

Restart Docker

```bash
sudo systemctl restart docker
```

---

# 4. Running Docker

## Pull Image

Example

```bash
sudo docker pull nvcr.io/nvidia/l4t-pytorch:r32.6.1-pth1.8-py3
```

---

## Run Docker

```bash
sudo docker run -it --rm \
--runtime nvidia \
--network host \
--device /dev/video0 \
-v /home/nvidia/project:/workspace \
nvcr.io/nvidia/l4t-pytorch:r32.6.1-pth1.8-py3
```

---

## Meaning of Parameters

| Parameter | Purpose |
|-----------|----------|
| -it | Interactive terminal |
| --rm | Remove container after exit |
| --runtime nvidia | GPU access |
| --network host | Host networking |
| --device | Hardware access |
| -v | Volume mounting |

---

## Exit Docker

```bash
exit
```

---

## View Containers

```bash
sudo docker ps -a
```

---

## Save Changes

```bash
sudo docker commit container_id mydocker:v1
```

---

# 5. Dockerfile

## Important Instructions

| Instruction | Purpose |
|------------|----------|
| FROM | Base image |
| ARG | Build variable |
| RUN | Execute commands |
| WORKDIR | Working directory |
| ENV | Environment variable |
| COPY | Copy files |
| SOURCE | Execute shell script |

---

## Example Dockerfile

```dockerfile
FROM nvcr.io/nvidia/l4t-pytorch:r32.6.1-pth1.8-py3

WORKDIR /workspace

COPY . /workspace

RUN apt-get update
RUN apt-get install -y python3-pip

ENV CUDA_VISIBLE_DEVICES=0
```

---

## Build Docker

```bash
sudo docker build -t mydocker:v1 .
```

---

# 6. Modifying Docker

## Method 1 (Commit)

Run container

↓

Modify

↓

Exit

↓

Commit

```bash
sudo docker commit container_id newrepo:v1
```

---

## Method 2 (Dockerfile)

Modify Dockerfile

↓

Build new image

```bash
sudo docker build -t newimage:v2 .
```

Recommended because it is reproducible.

---

# 7. Adding Data

## Method 1 (Volume Mount)

```bash
-v /home/nvidia/data:/workspace/data
```

Advantages

- Persistent
- Easy editing

---

## Method 2

Dockerfile

```
COPY
git clone
wget
```

---

## Method 3

Commit after copying files.

---

# 8. AI Deployment on Jetson

Three approaches

---

## Approach 1

Build everything manually

```
Framework

↓

Pull container

↓

Write code

↓

Commit
```

---

## Approach 2 (Used in Lab)

jetson-inference

Clone

```bash
git clone --recursive --depth=1 https://github.com/dusty-nv/jetson-inference
```

Run

```bash
./docker/run.sh
```

---

## Approach 3

Advanced frameworks

- DeepStream
- Triton
- ROS

---

# 9. Image Classification

Run

```bash
cd build/aarch64/bin

./imagenet.py images/orange_0.jpg images/test/output.jpg
```

Default network

```
GoogLeNet
```

---

Change Network

```bash
./imagenet.py --network=resnet-18 image.jpg output.jpg
```

---

Camera Input

```bash
./imagenet.py /dev/video0 output.mp4
```

---

# 10. Object Detection

Run

```bash
./detectnet.py \
--network=ssd-mobilenet-v2 \
images/peds_0.jpg \
images/test/output.jpg
```

Default network

```
SSD-Mobilenet-V2
```

---

Batch Images

```bash
./detectnet.py "images/*.jpg" images/output_%i.jpg
```

---

Video

```bash
./detectnet.py pedestrians.mp4 output.mp4
```

---

Download Models

```bash
./tools/download-models.sh
```

---

# 11. Image Segmentation

Run

```bash
./segnet.py \
--network=fcn-resnet18-cityscapes \
images/city_0.jpg \
images/test/output.jpg
```

---

Pipeline

```
Input

↓

Segmentation Model

↓

Pixel Labels

↓

Mask Overlay

↓

Output
```

---

# 12. Application Design Questions

---

## Count Students

Task

Object Detection

Pipeline

```
Camera

↓

Person Detection

↓

Count Persons
```

---

## Students Using Laptop

Task

Object Detection

Detect

- Person
- Laptop

Logic

Person + Laptop nearby

↓

Count

---

## Pet Detection

Task

Object Detection

Detect

```
Dog

Cat

Bird
```

Trigger alert.

---

## Group Images

Task

Classification

```
Image

↓

Classifier

↓

Category Folder
```

---

## Save Unique Chairs

Task

Detection + Feature Matching

```
Detect Chairs

↓

Extract Features

↓

Compare Similarity

↓

Store Unique Chairs
```

---

# 13. Frequently Used Docker Commands

| Task | Command |
|------|---------|
| Images | docker images |
| Containers | docker ps -a |
| Pull | docker pull |
| Run | docker run |
| Build | docker build |
| Commit | docker commit |
| Remove Image | docker rmi |
| Remove Container | docker rm |

---

# 14. NVIDIA TAO Toolkit

## What is TAO?

TAO (Train Adapt Optimize)

Used for

- Transfer learning
- Training
- Fine-tuning
- Optimization

without writing deep learning code.

---

## TAO Workflow

```
Dataset

↓

Train

↓

Evaluate

↓

Export

↓

TensorRT

↓

Deploy on Jetson
```

---

## Available Containers

| Container | Purpose |
|-----------|----------|
| tao-toolkit | Main training |
| tao-toolkit-tf | TensorFlow |
| tao-toolkit-pytorch | PyTorch |
| tao-toolkit-deploy | Deployment |

---

## Pull TAO

```bash
sudo docker pull nvcr.io/nvidia/tao/tao-toolkit
```

---

## Run TAO

```bash
sudo docker run -it \
--runtime=nvidia \
-v /home/nvidia/data:/workspace/data \
nvcr.io/nvidia/tao/tao-toolkit
```

---

## Train

```bash
tao model train \
-e spec.yaml \
-r results \
-k $KEY
```

---

## Evaluate

```bash
tao model evaluate \
-m model.tlt \
-e spec.yaml
```

---

## Export

```bash
tao model export \
-m model.tlt \
-o model.etlt \
-k $KEY
```

---

## TensorRT Conversion

```bash
tao-converter model.etlt
```

---

# 15. Universal Deployment Pipeline (Most Important)

For **Classification / Detection / Segmentation**

```
Problem Statement

↓

Choose AI Task

↓

Choose Model

↓

Choose Framework

↓

Choose Docker Container

↓

Pull Docker

↓

Run Docker

↓

Mount Data

↓

Run Inference

↓

Generate Output

↓

(Optional)
Commit Docker
```

---

# Quick Revision (1 Minute)

SSH

```bash
ssh nvidia@192.168.55.1
```

Copy Files

```bash
scp
```

Pull Docker

```bash
docker pull
```

Run Docker

```bash
docker run
```

Build Docker

```bash
docker build
```

Save Docker

```bash
docker commit
```

Classification

```bash
imagenet.py
```

Detection

```bash
detectnet.py
```

Segmentation

```bash
segnet.py
```

TAO

```
Train → Evaluate → Export → TensorRT → Deploy
```

---

# Universal 7-Mark Answer Structure (Deployment Questions)

1. Identify the AI task (classification/detection/segmentation).
2. Select a suitable model and justify the choice.
3. Choose the appropriate Docker/NGC container.
4. Explain the deployment pipeline.
5. Include the Docker or inference command.
6. Mention any required modifications (camera, video, model, tracking, etc.).
7. Explain the expected output.

> **Exam Tip:** Always include a small pipeline diagram and at least one command snippet wherever applicable, as instructed by ma'am.
