# Copyright (c) Meta Platforms, Inc. and affiliates.
# All rights reserved.
#
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.

################################################################################
# CMake Prelude
################################################################################

cmake_minimum_required(VERSION 3.25.0 FATAL_ERROR)

set(CMAKEMODULES ${CMAKE_CURRENT_SOURCE_DIR}/../cmake/modules)
set(FBGEMM ${CMAKE_CURRENT_SOURCE_DIR}/..)
set(THIRDPARTY ${FBGEMM}/third_party)

include(${CMAKEMODULES}/Utilities.cmake)


################################################################################
# FBGEMM_GPU Build Options
################################################################################

option(FBGEMM_CPU_ONLY  "Build FBGEMM_GPU without GPU support" OFF)
option(USE_ROCM         "Build FBGEMM_GPU for ROCm" OFF)

if((NOT FBGEMM_CPU_ONLY) AND
   ((EXISTS "/opt/rocm/") OR (EXISTS $ENV{ROCM_PATH})) AND
   (NOT EXISTS "/bin/nvcc"))
  message(
    "CMake has been set to build a non-CPU variant"
    "and AMD GPU has been detected; "
    "will default to ROCm build"
  )
  set(USE_ROCM ON)
endif()

if(FBGEMM_CPU_ONLY)
  BLOCK_PRINT("Building the CPU-only variant of FBGEMM-GPU")
elseif(USE_ROCM)
  BLOCK_PRINT("Building the ROCm variant of FBGEMM-GPU")
else()
  BLOCK_PRINT("Building the CUDA variant of FBGEMM-GPU")
endif()


################################################################################
# FBGEMM_GPU Build Kickstart
################################################################################

# FBGEMM_GPU C++ Setup - must be set BEFORE project declaration
include(${CMAKEMODULES}/CxxCompilerSetup.cmake)

if(SKBUILD)
  BLOCK_PRINT("The project is built using scikit-build")
endif()

if(FBGEMM_CPU_ONLY OR USE_ROCM)
  project(
    fbgemm_gpu
    VERSION 0.7.0
    LANGUAGES CXX C)
else()
  project(
    fbgemm_gpu
    VERSION 0.7.0
    LANGUAGES CXX C CUDA)
endif()

# AVX Flags Setup - must be set AFTER project declaration
include(${CMAKEMODULES}/FindAVX.cmake)

# PyTorch Dependencies Setup
include(${CMAKEMODULES}/PyTorchSetup.cmake)

# CUDA Setup
include(${CMAKEMODULES}/CudaSetup.cmake)

# ROCm and HIPify Setup
include(${CMAKEMODULES}/RocmSetup.cmake)


################################################################################
# Build FBGEMM_GPU (Main) Module
################################################################################

include(FbgemmGpu.cmake)

################################################################################
# Build Experimental Modules
################################################################################

add_subdirectory(experimental/example)


if(NOT FBGEMM_CPU_ONLY)
  add_subdirectory(experimental/gen_ai)
endif()
