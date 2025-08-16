## BrainCorp gentle-fork of:

<p align="center"><img src="doc/img/realsense.png" width="70%" /><br><br></p>

## tl;dr

> Note that the master branch has commits/files which may need to be cherry-picked on top of whatever version you're trying to build.  In particular, changes include:

* ./CMake/external_pybind11.cmake
* ./60-librealsense2-udev-rules.rules
* ./nfpm.yaml
* ./nfpm-devpkg.yaml


Allows us to build python bindings for in-house versions of Python (initially python3.12)

```
sudo apt install python3.12... # install our in-house python3 packages, including -dev and lib- -- grab the debs, add the repo, whatever
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 1

# install librealsense build deps
sudo apt-get install -y libx11-dev xorg-dev libglu1-mesa-dev libusb-1.0-0-dev libudev-dev
mkdir build && cd build
git checkout v2.56.5 # or whatever
cmake ../ -DBUILD_PYTHON_BINDINGS:bool=true -DPYTHON_EXECUTABLE=$(which python3.12) -DREQUIRED_PYTHON_VER=3.12
make -j4

# install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

cp -r ../wrappers/python/ Release/
cp Release/pyr*so* Release/python/pyrealsense2/
cd Release/python/
echo '__version__ = "2.56.5.9999"' > pyrealsense2/_version.py
uv build --wheel
# do stuff with `dist/pyrealsense2-2.56.5.9999-cp312-cp312-linux_x86_64.whl`
```

Apparently we also need the actual lib's and bin's and headers and who knows what else from this project, who knew.

```
# after building above, let's create a deb, need one for arm and one for x86
# Install `nfpm` for simplicity:
# Arm:
curl -L -O https://github.com/goreleaser/nfpm/releases/download/v2.43.0/nfpm_2.43.0_arm64.deb
dpkg -i ./nfpm*deb
DEBIAN_ARCH=arm64 TARGET_VERSION=2.56.5 TARGET_ARCH=aarch64 nfpm package --packager deb
DEBIAN_ARCH=arm64 TARGET_VERSION=2.56.5 TARGET_ARCH=aarch64 nfpm --config nfpm-devpkg.yaml package --packager deb
# x86:
curl -L -O https://github.com/goreleaser/nfpm/releases/download/v2.43.0/nfpm_2.43.0_amd64.deb
dpkg -i ./nfpm*deb
DEBIAN_ARCH=amd64 TARGET_VERSION=2.56.5 TARGET_ARCH=x86_64 nfpm package --packager deb
DEBIAN_ARCH=amd64 TARGET_VERSION=2.56.5 TARGET_ARCH=x86_64 nfpm --config nfpm-devpkg.yaml package --packager deb

# upload deb's to GCP Artifact Registry
```

-----------------
[![License](https://img.shields.io/github/license/IntelRealSense/librealsense.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Release](https://img.shields.io/github/v/release/IntelRealSense/librealsense?sort=semver)](https://github.com/IntelRealSense/librealsense/releases/latest)
[![Commits](https://img.shields.io/github/commits-since/IntelRealSense/librealsense/master/development?label=commits%20since)](https://github.com/IntelRealSense/librealsense/compare/master...development)
[![Issues](https://img.shields.io/github/issues/IntelRealSense/librealsense.svg)](https://github.com/IntelRealSense/librealsense/issues)
[![GitHub CI](../../actions/workflows/buildsCI.yaml/badge.svg?branch=development)](../../actions/workflows/buildsCI.yaml)
[![Forks](https://img.shields.io/github/forks/IntelRealSense/librealsense.svg)](https://github.com/IntelRealSense/librealsense/network/members)



## Overview
**Intel® RealSense™ SDK 2.0** is a cross-platform library for Intel® RealSense™ depth cameras.

> :pushpin: For other Intel® RealSense™ devices (F200, R200, LR200 and ZR300), please refer to the [latest legacy release](https://github.com/IntelRealSense/librealsense/tree/v1.12.1).

The SDK allows depth and color streaming, and provides intrinsic and extrinsic calibration information.
The library also offers synthetic streams (pointcloud, depth aligned to color and vise-versa), and a built-in support for [record and playback](./doc/record-and-playback.md) of streaming sessions.

Developer kits containing the necessary hardware to use this library are available for purchase at [store.intelrealsense.com](https://store.intelrealsense.com/products.html).
Information about the Intel® RealSense™ technology at [www.intelrealsense.com](https://www.intelrealsense.com/)

> :open_file_folder: Don't have access to a RealSense camera? Check-out [sample data](./doc/sample-data.md)

## Update on Recent Changes to the RealSense Product Line

Please visit this link for product updates - https://www.intelrealsense.com/message-to-customers/

## Building librealsense - Using vcpkg

You can download and install librealsense using the [vcpkg](https://github.com/Microsoft/vcpkg) dependency manager:

    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    ./bootstrap-vcpkg.sh
    ./vcpkg integrate install
    ./vcpkg install realsense2

The librealsense port in vcpkg is kept up to date by Microsoft team members and community contributors. If the version is out of date, please [create an issue or pull request](https://github.com/Microsoft/vcpkg) on the vcpkg repository.

## Download and Install
* **Download** - The latest releases including the Intel RealSense SDK, Viewer and Depth Quality tools are available at: [**latest releases**](https://github.com/IntelRealSense/librealsense/releases). Please check the [**release notes**](https://github.com/IntelRealSense/librealsense/wiki/Release-Notes) for the supported platforms, new features and capabilities, known issues, how to upgrade the Firmware and more.

* **Install** - You can also install or build from source the SDK (on [Linux](./doc/distribution_linux.md) \ [Windows](./doc/distribution_windows.md) \ [Mac OS](doc/installation_osx.md) \ [Android](./doc/android.md) \ [Docker](./scripts/Docker/readme.md)), connect your D400 depth camera and you are ready to start writing your first application.

> **Support & Issues**: If you need product support (e.g. ask a question about / are having problems with the device), please check the [FAQ & Troubleshooting](https://github.com/IntelRealSense/librealsense/wiki/Troubleshooting-Q%26A) section.
> If not covered there, please search our [Closed GitHub Issues](https://github.com/IntelRealSense/librealsense/issues?utf8=%E2%9C%93&q=is%3Aclosed) page,  [Community](https://communities.intel.com/community/tech/realsense) and [Support](https://www.intel.com/content/www/us/en/support/emerging-technologies/intel-realsense-technology.html) sites.
> If you still cannot find an answer to your question, please [open a new issue](https://github.com/IntelRealSense/librealsense/issues/new).

## What’s included in the SDK:
| What | Description | Download link|
| ------- | ------- | ------- |
| **[Intel® RealSense™ Viewer](./tools/realsense-viewer)** | With this application, you can quickly access your Intel® RealSense™ Depth Camera to view the depth stream, visualize point clouds, record and playback streams, configure your camera settings, modify advanced controls, enable depth visualization and post processing  and much more. | [**Intel.RealSense.Viewer.exe**](https://github.com/IntelRealSense/librealsense/releases) |
| **[Depth Quality Tool](./tools/depth-quality)** | This application allows you to test the camera’s depth quality, including: standard deviation from plane fit, normalized RMS – the subpixel accuracy, distance accuracy and fill rate. You should be able to easily get and interpret several of the depth quality metrics and record and save the data for offline analysis. |[**Depth.Quality.Tool.exe**](https://github.com/IntelRealSense/librealsense/releases) |
| **[Debug Tools](./tools/)** | Device enumeration, FW logger, etc as can be seen at the tools directory | Included in [**Intel.RealSense.SDK.exe**](https://github.com/IntelRealSense/librealsense/releases)|
| **[Code Samples](./examples)** |These simple examples demonstrate how to easily use the SDK to include code snippets that access the camera into your applications. Check some of the [**C++ examples**](./examples) including capture, pointcloud and more and basic [**C examples**](./examples/C) | Included in [**Intel.RealSense.SDK.exe**](https://github.com/IntelRealSense/librealsense/releases) |
| **[Wrappers](https://github.com/IntelRealSense/librealsense/tree/development/wrappers)** | [Python](./wrappers/python), [C#/.NET](./wrappers/csharp) API, as well as integration with the following 3rd-party technologies: [ROS1](https://github.com/IntelRealSense/realsense-ros/tree/ros1-legacy), [ROS2](https://github.com/IntelRealSense/realsense-ros/tree/ros2-master), [LabVIEW](./wrappers/labview), [OpenCV](./wrappers/opencv), [PCL](./wrappers/pcl), [Unity](./wrappers/unity), [Matlab](./wrappers/matlab), [OpenNI](./wrappers/openni2), [UnrealEngine4](./wrappers/unrealengine4) and more to come. | |


## Ready to Hack!

Our library offers a high level API for using Intel RealSense depth cameras (in addition to lower level ones).
The following snippet shows how to start streaming frames and extracting the depth value of a pixel:

```cpp
// Create a Pipeline - this serves as a top-level API for streaming and processing frames
rs2::pipeline p;

// Configure and start the pipeline
p.start();

while (true)
{
    // Block program until frames arrive
    rs2::frameset frames = p.wait_for_frames();

    // Try to get a frame of a depth image
    rs2::depth_frame depth = frames.get_depth_frame();

    // Get the depth frame's dimensions
    float width = depth.get_width();
    float height = depth.get_height();

    // Query the distance from the camera to the object in the center of the image
    float dist_to_center = depth.get_distance(width / 2, height / 2);

    // Print the distance
    std::cout << "The camera is facing an object " << dist_to_center << " meters away \r";
}
```
For more information on the library, please follow our [examples](./examples), and read the [documentation](./doc) to learn more.

## Contributing
In order to contribute to Intel RealSense SDK, please follow our [contribution guidelines](CONTRIBUTING.md).

## License
This project is licensed under the [Apache License, Version 2.0](LICENSE).
Copyright 2018 Intel Corporation
