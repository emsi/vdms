## Installation

Here is the detailed process of installation of VDMS dependencies.

## Dependencies

    sudo apt-get install scons
    sudo apt-get install libjsoncpp-dev
    sudo apt-get install automake libtool curl make g++ unzip libgtest-dev
    sudo apt-get install cmake wget zlib1g-dev libbz2-dev libssl-dev liblz4-dev
    sudo apt-get install libtiff5-dev libjasper-dev libgtk-3-dev
    sudo apt-get install wget
    sudo apt-get install ffmpeg

    // Also, install one of the following for MPI
    sudo apt-get install libopenmpi-dev
    sudo apt-get install mpich

### Install gtest

Unfortunately apt doesn't build gtest;
you need to do the following steps to get it to work correctly:

    cd /usr/src/gtest/
    sudo cmake CMakeLists.txt
    sudo make
    sudo cp *.a /usr/lib

### Protobuf

 Google Protobufs. Default install location is /usr/local.

    git clone https://github.com/google/protobuf.git
    cd protobuf/
    ./autogen.sh
    ./configure
    make && make check
    sudo make install
    sudo ldconfg

### [OpenCV](https://opencv.org/)

Below are instructions for installing OpenCV v3.3.1.
It may also work with newer versions of OpenCV.
Download [OpenCV 3.3.1](https://github.com/opencv/opencv/archive/3.3.1.zip)

    unzip 3.3.1.zip
    cd 3.3.1
    mkdir build
    cd build/
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
    make -jX   # substitute X for maximum number of CPUs (or leave it off to use all available)
    sudo make install

**Note**: When using videos, and getting the following error: "Unable to stop the stream: Inappropriate ioctl for device", you may need to include more flags when compiling OpenCV. Follow these instructions ([source](https://stackoverflow.com/questions/41200201/opencv-unable-to-stop-the-stream-inappropriate-ioctl-for-device)):

    sudo apt-get install ffmpeg
    sudo apt-get install libavcodec-dev libavformat-dev libavdevice-dev

    // Rebuild OpenCV with the following commands:

    cd build
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local  -D WITH_FFMPEG=ON -D WITH_TBB=ON -D WITH_GTK=ON -D WITH_V4L=ON -D WITH_OPENGL=ON -D WITH_CUBLAS=ON -DWITH_QT=OFF -DCUDA_NVCC_FLAGS="-D_FORCE_INLINES" ..
    make -j7
    sudo make install

### [TileDB](https://tiledb.io/)
VDMS works with TileDB v1.3.1.
The directions below will help you install TileDB v1.3.1 from source.
You can also follow the directions listed
[here](https://docs.tiledb.io/en/latest/installation.html).

    sudo apt-get install zlib1g-dev libssl-dev liblz4-dev libbz2-dev

Install Blosc

    git clone https://github.com/Blosc/c-blosc.git`
    cd c-blosc/
    mkdir build
    cd build/
    cmake -DCMAKE_INSTALL_PREFIX='/usr' ..
    cmake --build .
    ctest
    sudo cmake --build . --target install

Install ZStandard

    wget https://github.com/facebook/zstd/archive/v1.1.0.tar.gz
    tar xf v1.1.0.tar.gz
    cd zstd-1.1.0
    sudo make install PREFIX='/usr'

Build TileDB

    git clone https://github.com/TileDB-Inc/TileDB
    cd TileDB
    git checkout 1.3.1
    mkdir build
    cd build
    ../bootstrap --prefix=/usr/local/
    make -jX   # substitute X for maximum number of CPUs (or leave it off to use all available)
    sudo make install-tiledb

### [Faiss](https://github.com/facebookresearch/faiss)
Facebook Faiss library for similarity search, used as alternitive engines.

Download [Faiss 1.2.1](https://github.com/facebookresearch/faiss/archive/v1.2.1.tar.gz)

    tar -xzvf v1.2.1.tar.gz
    cd faiss-1.2.1
    mkdir build
    cd build/
    cmake ..
    make -j

You may need to change some flags in the CMakeFile depending on
configurations on your system.

Things that we have needed to change in CMakeLists.txt:
* If your system doesn't have a GPU, make sure that BUILD_WITH_GPU is OFF
* You may need to add -msse4 to set(CMAKE_CXX_FLAGS
* Change the add_library call to be SHARED instead of STATIC

This library does not offer an install. Make sure you move .h files
to /usr/lib/include/faiss or /usr/local/lib/include/faiss,
and make sure you make the library (libfaiss.so) is available system-wide
Or follow instructions
[here](https://github.com/facebookresearch/faiss/blob/v1.2.1/INSTALL.md)

### Valijson
  * git clone https://github.com/tristanpenman/valijson.git
  * cd valijson
  * cp -r include/* /usr/local/include (may need to run as sudo)
  * This is a headers-only library, no compilation/installation necessary

### Persistent Memory Graph Database (PMGD)
  * Download version 2.0.0 from: https://github.com/IntelLabs/pmgd/releases
  * Follow installation instructions


## Python Client Module

VDMS offers the Python Client Module through the pip package manager,
and it is compatible with Python 2.7+ and 3.3+.
pip (or pip2 and pip3) will automatically install dependencies (protobuf).

    pip install vdms

## Compilation

    git clone https://github.com/intellabs/vdms
    // Or download a release.

    cd vdms
    scons [FLAGS]

Flag | Explanation
------------ | -------------
--no-server | Compiles client libraries (C++/Python) only. (will not compile neither server nor tests)
--timing    | Compiles server with chronos for internal timing, experimental.
-jX         | Compiles in parallel, using X cores
INTEL_PATH=path  | Path to the root folder containing pmgd and vcl. Default is "./" which is pmgd and vcl inside vdms folder. Example: scons INTEL_PATH=/opt/intel/

### Running The VDMS Server

The config-vdms.json file contains the configuration of the server.
Some of the parameters include the TCP port that will be use for incoming
connections, maximun number of simultaneous clients, and paths to the
folders where data/metadata will be stored.

We provide a script (run_server.sh) that will create some default directories,
corresponding the default values in the config-vdms.json.

To run the server using the default directories and port, simply run:

    sh run_server.sh

