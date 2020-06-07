---
title: "Build Tensorflow 1.9.0 with CUDA 9.2 on Windows 10"
tags: ["reinforcement learning"]
---
# Prerequisite
 * CUDA 9.2
 * CUDNN 7.1
 * Anaconda3
 * Visual Studio 2017 with toolset v140 enabled
 * SWIG on windows

# Install dependencies
```python
conda install numpy
```
```
git clone https://github.com/tensorflow/tensorflow.git
```
# Configure & Generate

Change `CUDA_VERSION` to `9.2` and `CUDNN_VERSION` to `7.1` in `tensorflow\tensorflow\contrib\cmake\CMakeLists.txt`
```
set(tensorflow_CUDA_VERSION "9.2" CACHE STRING "CUDA version to build against")
set(tensorflow_CUDNN_VERSION "7.1" CACHE STRING "cuDNN version to build against")
```
In `x64 Native Tools Command Prompt for VS 2017`
```
cmake .. -G "Visual Studio 15 2017" -T v140 ^
-A x64 ^
-DCMAKE_BUILD_TYPE=Release ^
-DSWIG_EXECUTABLE=C:\swigwin-3.0.12\swig.exe ^
-DPYTHON_EXECUTABLE=C:\Users\WuMo\Anaconda3\python.exe ^
-DPYTHON_LIBRARIES=C:\Users\WuMo\Anaconda3\libs\python36.lib ^
-Dtensorflow_ENABLE_GPU=ON 
-DCUDNN_HOME="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.2" ^
-Dtensorflow_BUILD_SHARED_LIB=ON ^
-DCUDA_HOST_COMPILER="C:/Program Files (x86)/Microsoft Visual Studio 14.0/VC/bin/amd64/cl.exe"
```
```
cmake .. -G "Visual Studio 15 2017" -T v140 -A x64 -DCMAKE_BUILD_TYPE=Release -DSWIG_EXECUTABLE=C:\swigwin-3.0.12\swig.exe -DPYTHON_EXECUTABLE=C:\Users\WuMo\Anaconda3\python.exe -DPYTHON_LIBRARIES=C:\Users\WuMo\Anaconda3\libs\python36.lib -Dtensorflow_ENABLE_GPU=ON -DCUDNN_HOME="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.2" -Dtensorflow_BUILD_SHARED_LIB=ON -DCUDA_HOST_COMPILER="C:/Program Files (x86)/Microsoft Visual Studio 14.0/VC/bin/amd64/cl.exe"
```
```
cmake .. -G "Visual Studio 15 2017" -T v140 -A x64 -DCMAKE_BUILD_TYPE=Release -DSWIG_EXECUTABLE=C:\swigwin-3.0.12\swig.exe -DPYTHON_EXECUTABLE=C:\Users\WuMo-Zion\Miniconda3\python.exe -DPYTHON_LIBRARIES=C:\Users\WuMo-Zion\Miniconda3\libs\python36.lib -Dtensorflow_BUILD_SHARED_LIB=ON -Dtensorflow_WIN_CPU_SIMD_OPTIONS=/arch:AVX2
```
<!--more-->
# Build

```
“C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\amd64\MSBuild.exe” ^
/p:Configuration=Release ^
/p:Platform=x64 ^
/p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj ^
/filelogger
```
```
“C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\amd64\MSBuild.exe” /p:Configuration=Release /p:Platform=x64 /p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj /filelogger
```

# Build JavaCPP-Presets
modify `javacpp-presets/tensorflow/cppbuild.sh` as follows:

```
 windows-x86_64)
        # help cmake's findCuda-method to find the right cuda version
        export CUDA_PATH="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v$TF_CUDA_VERSION"
        patch -Np1 < ../../../tensorflow-java.patch
        patch -Np1 < ../../../tensorflow-windows.patch
        mkdir -p ../build
        cd ../build
        "$CMAKE" -G "Visual Studio 15 2017" -T v140 -A x64 -DCMAKE_BUILD_TYPE=Release -DSWIG_EXECUTABLE="C:/swigwin-3.0.12/swig.exe" -DPYTHON_EXECUTABLE="C:/Users/WuMo-Zion/Miniconda3/python.exe" -DPYTHON_LIBRARIES="C:/Users/WuMo-Zion/Miniconda3/libs/python36.lib" -Dtensorflow_BUILD_PYTHON_BINDINGS=ON -Dtensorflow_BUILD_SHARED_LIB=ON -Dtensorflow_WIN_CPU_SIMD_OPTIONS=/arch:AVX -Dtensorflow_ENABLE_MKLDNN_SUPPORT=ON $CMAKE_GPU_FLAGS -DCUDNN_HOME="$CUDA_PATH" -DCUDA_HOST_COMPILER="C:/Program Files (x86)/Microsoft Visual Studio 14.0/VC/bin/amd64/cl.exe" ../tensorflow-$TENSORFLOW_VERSION/tensorflow/contrib/cmake
        MSBuild.exe //p:Configuration=Release //p:Platform=x64 //p:PreferredToolArchitecture=x64 //filelogger ALL_BUILD.vcxproj 
        # if [[ ! -f ../build/Release/tensorflow_static.lib ]]; then
        #     MSBuild.exe //p:Configuration=Release //p:Platform=x64 //p:PreferredToolArchitecture=x64 //filelogger "tensorflow_static.vcxproj"
        # fi
        # if [[ "$EXTENSION" == *gpu ]] && [[ "${PARTIAL_CPPBUILD:-}" != "1" ]]; then
        #     MSBuild.exe //p:Configuration=Release /p:Platform=x64 /maxcpucount:$MAKEJ /p:PreferredToolArchitecture=x64 tf_core_gpu_kernels.vcxproj /filelogger
        # fi
        cd ../tensorflow-$TENSORFLOW_VERSION
        ;;
    *)
```
