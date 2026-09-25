# sam2-onnx-cpp/cpp/CMakeLists.txt
cmake_minimum_required(VERSION 3.14)
project(SAM2_ONNX)

set(TARGET_NAME "Segment")

# C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Output exe into build/bin
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# QoL: compiler flags
add_compile_options(-O3 -fno-omit-frame-pointer)

# ---- 1) OpenCV ----
# Allow CMake to find OpenCV automatically on the system paths
find_package(OpenCV REQUIRED)
message(STATUS "Found OpenCV ${OpenCV_VERSION} at: ${OpenCV_DIR}")
include_directories(${OpenCV_INCLUDE_DIRS})

# ---- 2) ONNX Runtime headers + library ----
set(ONNXRUNTIME_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/onnxruntime-include")
set(ONNXRUNTIME_LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/onnxruntime-lib")

include_directories(${ONNXRUNTIME_INCLUDE_DIR})

# Find the ORT library directly from the local directory
find_library(ONNXRUNTIME_LIB
  NAMES onnxruntime
  HINTS ${ONNXRUNTIME_LIB_DIR}
  NO_DEFAULT_PATH
  REQUIRED)

# Let the loader look next to the binary as well
set(CMAKE_BUILD_RPATH   "$ORIGIN")
set(CMAKE_INSTALL_RPATH "$ORIGIN")

message(STATUS "ONNX Runtime lib: ${ONNXRUNTIME_LIB}")

# ---- 3) TensorRT ----
set(TENSORRT_INCLUDE_DIR "/usr/include/aarch64-linux-gnu")
set(TENSORRT_LIB_DIR "/usr/lib/aarch64-linux-gnu")

include_directories(${TENSORRT_INCLUDE_DIR})

# Find the core TensorRT library (nvinfer) in the specified path
find_library(TENSORRT_LIB
  NAMES nvinfer
  HINTS ${TENSORRT_LIB_DIR}
  NO_DEFAULT_PATH
  REQUIRED)

message(STATUS "TensorRT lib: ${TENSORRT_LIB}")

# ---- 4) Sources ----
include_directories(${CMAKE_SOURCE_DIR}/src)
include_directories(${CMAKE_SOURCE_DIR})  # headers at repo root

set(OPENFILEDIALOG_SRC src/openFileDialog.cpp)

add_executable(${TARGET_NAME}
  src/main.cpp
  src/onnx_test_image.cpp
  src/onnx_test_video.cpp
  src/SAM2Session.cpp
  src/SAM2Image.cpp
  src/SAM2Video.cpp
  ${OPENFILEDIALOG_SRC}
)

# ---- 5) Link ----
target_link_libraries(${TARGET_NAME} PRIVATE
  ${ONNXRUNTIME_LIB}
  ${TENSORRT_LIB}
  ${OpenCV_LIBS}
)

# ---- 6) Install / package ----
install(TARGETS ${TARGET_NAME}
        RUNTIME DESTINATION bin)

install(DIRECTORY "${CMAKE_SOURCE_DIR}/models/"
        DESTINATION share/models)

set(CPACK_PACKAGE_NAME "SAM2_ONNX")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_GENERATOR "TGZ")
include(CPack)