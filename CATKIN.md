# Embedded Vision EST Notes
# Topic: Catkin Workspace & Manual Dependency Building (ROS Noetic)

---

# Why This Topic is Important

As **ROS Noetic is archived**, many repositories and package dependencies are no longer actively maintained. While building ROS packages, automatic dependency installation may fail due to:

- Missing repositories
- Version incompatibility
- Missing shared libraries
- CMake dependency resolution failures

Hence, some dependencies must be **built manually**.

---

# Catkin Workspace Architecture

A Catkin workspace is the standard ROS development workspace.

```
catkin_ws/
│
├── src/        → Contains all ROS packages
├── build/      → Temporary build files
├── devel/      → Generated environment setup
└── install/    → Installed binaries/libraries (optional)
```

### Folder Functions

| Folder | Purpose |
|---------|----------|
| src | Stores ROS packages and source code |
| build | Contains temporary compilation files |
| devel | Stores generated executables and environment setup |
| install | Stores installed packages |

---

# ROS Build Flow

```
Source Code
      │
      ▼
catkin_make
      │
      ▼
CMake Configuration
      │
      ▼
Dependency Resolution
      │
      ▼
Compilation
      │
      ▼
Executables + Libraries
      │
      ▼
devel/setup.bash
```

---

# Why Delete build and devel Folders?

Command

```bash
rm -rf build devel
```

## Why?

If a build fails, Catkin keeps

- cached CMake configurations
- partially compiled object files
- incomplete dependency links
- stale build artifacts

These may cause repeated compilation failures.

Deleting these folders forces a **clean rebuild**.

### Exam Answer

> The build and devel folders are removed to clear stale CMake cache, incomplete object files, and partially linked dependencies before rebuilding the Catkin workspace.

---

# Manual Building of cv_bridge

---

## What is cv_bridge?

`cv_bridge` connects ROS image messages with OpenCV.

ROS Camera publishes:

```
sensor_msgs/Image
```

OpenCV processes:

```
cv::Mat
```

Since both formats are different, `cv_bridge` converts between them.

---

## Vision Pipeline

```
Camera Node
      │
      ▼
sensor_msgs/Image
      │
      ▼
cv_bridge
      │
      ▼
cv::Mat
      │
      ▼
OpenCV Algorithm
```

---

## Build Steps

Go inside source directory

```bash
cd catkin_ws/src
```

Clone repository

```bash
git clone https://github.com/ros-perception/vision_opencv.git
```

Move inside package

```bash
cd vision_opencv/
```

Checkout Noetic branch

```bash
git checkout noetic
```

Return

```bash
cd ..
```

---

## Why checkout "noetic"?

Different ROS distributions support different

- APIs
- OpenCV versions
- Python versions
- Dependency versions

Using the correct branch ensures compatibility.

### Exam Answer

> The Noetic branch is checked out to ensure compatibility with the ROS Noetic distribution and avoid API or dependency mismatches.

---

# Manual Building of image_transport

---

## What is image_transport?

image_transport provides efficient image transmission in ROS.

Instead of sending raw images, it supports

- compressed images
- compressedDepth
- Theora streaming
- plugin-based transports

---

## Why is it Needed?

Raw camera images consume

- high bandwidth
- more network resources
- lower FPS

image_transport minimizes bandwidth while maintaining image quality.

---

## ROS Image Flow

```
Camera Node
      │
      ▼
image_transport
      │
      ▼
Compressed Image Stream
      │
      ▼
Subscriber Node
```

---

## Build Steps

Clone repository

```bash
git clone https://github.com/ros-perception/image_common.git
```

Enter package

```bash
cd image_common/
```

Checkout compatible branch

```bash
git checkout noetic-devel
```

Return

```bash
cd ..
```

---

## Why noetic-devel?

This branch contains

- latest Noetic fixes
- compatibility updates
- maintained source code

---

# Boost Library Errors

---

## Common Error

```
cannot find libboost_python37.so
```

Available library

```
libboost_python3.so
```

---

## Why Does This Happen?

The build system searches for a specific library version.

However, the installed Boost version has a different filename.

This creates a linker error.

---

## Solution

Create a symbolic link.

```bash
ln -s /usr/lib/aarch64-linux-gnu/libboost_python3.so libboost_python37.so
```

---

## What is a Symbolic Link?

A symbolic link is simply a shortcut to another file.

```
Program requests

libboost_python37.so
          │
          ▼
Symbolic Link
          │
          ▼
libboost_python3.so
```

---

## Exam Answer

> Symbolic links resolve library naming mismatches by redirecting the expected library name to the available installed version.

---

# yaml-cpp Errors

---

## What is yaml-cpp?

yaml-cpp is the C++ library used for parsing YAML configuration files.

ROS uses YAML for

- parameters
- camera calibration
- launch configurations
- application settings

---

## Why Build Manually?

System-installed yaml-cpp may

- be outdated
- have ABI incompatibility
- lack required symbols

Hence, a compatible version is built manually.

---

## Build Steps

Clone repository

```bash
git clone https://github.com/jbeder/yaml-cpp.git
```

Enter directory

```bash
cd yaml-cpp/
```

Checkout stable version

```bash
git checkout yaml-cpp-0.6.3
```

Create build directory

```bash
mkdir build
cd build
```

Configure

```bash
cmake .. -DCMAKE_POSITION_INDEPENDENT_CODE=ON
```

Compile

```bash
make -j4
```

Install

```bash
sudo make install
```

Refresh linker cache

```bash
sudo ldconfig
```

Update CMake path

```bash
export CMAKE_PREFIX_PATH=/usr/local:$CMAKE_PREFIX_PATH
```

Return

```bash
cd ../..
```

---

# Why POSITION_INDEPENDENT_CODE?

Shared libraries can be loaded into different memory locations.

Position Independent Code (PIC) allows code to execute correctly regardless of where it is loaded.

It is essential for

- shared libraries
- dynamic loading
- ROS plugins

### Exam Answer

> POSITION_INDEPENDENT_CODE enables generation of position-independent machine code required for dynamically linked shared libraries.

---

# Why make -j4?

```
make -j4
```

Uses **4 parallel compilation threads**, reducing build time.

---

# Why sudo make install?

Copies

- libraries
- header files
- binaries

to system directories like

```
/usr/local/
```

---

# Why ldconfig?

```
sudo ldconfig
```

Linux maintains a cache of available shared libraries.

After installing a new library, this cache must be refreshed.

Otherwise, the linker cannot locate the newly installed library.

### Exam Answer

> ldconfig refreshes the dynamic linker cache so newly installed shared libraries become discoverable during compilation and execution.

---

# Why Export CMAKE_PREFIX_PATH?

```
export CMAKE_PREFIX_PATH=/usr/local:$CMAKE_PREFIX_PATH
```

CMake searches predefined directories for packages.

This command tells CMake to also search

```
/usr/local
```

where yaml-cpp was installed.

---

# Complete Dependency Resolution Workflow

```
Dependency Missing
        │
        ▼
Clone Source Repository
        │
        ▼
Checkout Compatible Version
        │
        ▼
Configure using CMake
        │
        ▼
Compile
        │
        ▼
Install Library
        │
        ▼
Refresh Linker Cache
        │
        ▼
Update CMake Search Path
        │
        ▼
Clean Catkin Workspace
        │
        ▼
Rebuild Workspace
```

---

# Common Commands Summary

### Clean Workspace

```bash
rm -rf build devel
```

---

### Build cv_bridge

```bash
cd catkin_ws/src

git clone https://github.com/ros-perception/vision_opencv.git

cd vision_opencv/

git checkout noetic

cd ..
```

---

### Build image_transport

```bash
git clone https://github.com/ros-perception/image_common.git

cd image_common/

git checkout noetic-devel

cd ..
```

---

### Fix Boost Error

```bash
ln -s /usr/lib/aarch64-linux-gnu/libboost_python3.so libboost_python37.so
```

---

### Build yaml-cpp

```bash
git clone https://github.com/jbeder/yaml-cpp.git

cd yaml-cpp/

git checkout yaml-cpp-0.6.3

mkdir build

cd build

cmake .. -DCMAKE_POSITION_INDEPENDENT_CODE=ON

make -j4

sudo make install

sudo ldconfig

export CMAKE_PREFIX_PATH=/usr/local:$CMAKE_PREFIX_PATH
```

---

# Frequently Asked Exam Questions

---

## Q1. Why are dependencies manually built in ROS Noetic?

### Answer

- ROS Noetic is archived.
- Some repositories are unavailable.
- Package versions may be incompatible.
- Automatic dependency resolution fails.
- Therefore dependencies are manually compiled from source.

---

## Q2. Explain the role of cv_bridge.

### Answer

cv_bridge converts ROS image messages (`sensor_msgs/Image`) into OpenCV image format (`cv::Mat`) and vice versa, allowing OpenCV algorithms to process images published by ROS camera nodes.

---

## Q3. What is image_transport?

### Answer

image_transport provides efficient image transmission through transport plugins such as compressed image transport, reducing bandwidth usage while maintaining performance.

---

## Q4. Why is a symbolic link created for Boost?

### Answer

A symbolic link resolves library naming mismatches between the required Boost library version and the installed version, allowing the linker to locate the correct shared library.

---

## Q5. Why is yaml-cpp built manually?

### Answer

The default system package may not be compatible with the required ROS version. Building a stable version manually ensures correct API and ABI compatibility.

---

## Q6. Why is ldconfig used?

### Answer

ldconfig refreshes the system's shared library cache after installation so newly installed libraries can be found during compilation and execution.

---

## Q7. Why delete build and devel before rebuilding?

### Answer

To remove stale CMake cache, incomplete object files, and partially linked dependencies that may prevent successful recompilation.

---

# One-Minute Revision

- ROS Noetic is archived → Manual dependency building required.
- Catkin Workspace → src, build, devel, install.
- Delete build & devel after failed builds.
- cv_bridge → Converts `sensor_msgs/Image` ↔ `cv::Mat`.
- image_transport → Efficient compressed image transmission.
- Boost Error → Fix using symbolic link.
- yaml-cpp → Manual build for compatibility.
- `cmake` → Configure build.
- `make -j4` → Parallel compilation.
- `sudo make install` → Install library.
- `ldconfig` → Refresh linker cache.
- `CMAKE_PREFIX_PATH` → Helps CMake locate manually installed libraries.
