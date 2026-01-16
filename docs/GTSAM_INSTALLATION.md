# GTSAM Installation Guide for Linux

## Overview
The `vicon2gt` package requires GTSAM (Georgia Tech Smoothing and Mapping) library version 4.2.0 to build successfully. This guide provides installation instructions for Linux systems.

## Quick Installation (Ubuntu/Debian)

If you have network connectivity, the easiest method is to install via apt:

```bash
sudo apt-get update
sudo apt-get install -y libgtsam-dev libgtsam4
```

This will install GTSAM 4.2.0 which is the exact version required by this package.

## Alternative: Installation via Conda/Mamba

If you're using the conda environment approach (recommended in the main README), GTSAM should be installed automatically when creating the environment:

```bash
cd catkin_ws/src
mamba create -n fusionpose_env -f env/environment_flexible.yml
mamba activate fusionpose_env
```

The `environment_flexible.yml` file includes `gtsam=4.2.0` as a dependency.

## Manual Installation from Source

If the above methods don't work (e.g., due to network issues or unavailable packages), you can build GTSAM from source:

### 1. Install Dependencies

```bash
sudo apt-get install -y \
    build-essential \
    cmake \
    libboost-all-dev \
    libeigen3-dev \
    libtbb-dev
```

### 2. Download and Build GTSAM 4.2.0

```bash
# Navigate to a temporary directory
cd /tmp

# Clone GTSAM repository
git clone https://github.com/borglab/gtsam.git
cd gtsam
git checkout 4.2.0

# Create build directory
mkdir build && cd build

# Configure with system Eigen (important!)
cmake .. \
    -DGTSAM_USE_SYSTEM_EIGEN=ON \
    -DGTSAM_BUILD_TESTS=OFF \
    -DGTSAM_BUILD_EXAMPLES_ALWAYS=OFF \
    -DCMAKE_BUILD_TYPE=Release

# Build (use -j flag for parallel compilation)
make -j$(nproc)

# Install
sudo make install

# Update library cache
sudo ldconfig
```

### 3. Verify Installation

Check that GTSAM is installed correctly:

```bash
ldconfig -p | grep gtsam
```

You should see output showing the GTSAM libraries.

## Verifying the Build Configuration

The `vicon2gt/CMakeLists.txt` has been updated to properly find and link GTSAM:

- Line 12: `find_package(GTSAM REQUIRED)` is now enabled
- Line 13: `set(GTSAM_LIBRARIES gtsam)` is now enabled
- Line 84: Links against the GTSAM library

## Troubleshooting

### Error: "gtsam/base/*.h: No such file or directory"
This means GTSAM is not installed or CMake can't find it. Follow the installation instructions above.

### Error: CMake can't find GTSAM
If CMake can't locate GTSAM after installation, you may need to set the GTSAM_DIR:

```bash
export GTSAM_DIR=/usr/local/lib/cmake/GTSAM
```

Or specify it during the catkin_make process:

```bash
catkin_make -DGTSAM_DIR=/usr/local/lib/cmake/GTSAM
```

### Network Connectivity Issues
If you're in an environment without network access (like Docker/containers), you need to:
1. Build GTSAM from source outside the container
2. Copy the built libraries into the container
3. Or use a base image that already has GTSAM installed

## Next Steps

After installing GTSAM, rebuild the workspace:

```bash
cd catkin_ws
catkin_make clean  # Optional: clean previous build artifacts
catkin_make
```

The build should now complete successfully without GTSAM-related errors.
