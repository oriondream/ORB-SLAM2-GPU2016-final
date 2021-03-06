cmake_minimum_required(VERSION 2.8)
project(ORB_SLAM2_CUDA)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE(STATUS "Build type: " ${CMAKE_BUILD_TYPE})

SET(CMAKE_C_FLAGS_DEBUG "-O2 -g")
SET(CMAKE_CXX_FLAGS_DEBUG "-O2 -g")

SET(CMAKE_C_FLAGS_RELEASE "-O3")
SET(CMAKE_CXX_FLAGS_RELEASE "-O3")

LIST(APPEND CMAKE_C_FLAGS "-Wall -Wextra -DUSE_NVTX")
LIST(APPEND CMAKE_CXX_FLAGS "-Wall -Wextra -DUSE_NVTX")

find_package(CUDA REQUIRED)
set(CUDA_PROPAGATE_HOST_FLAGS OFF)
SET(CUDA_HOST_COMPILER /usr/bin/g++)
LIST(APPEND CUDA_NVCC_FLAGS "--compiler-options -fno-strict-aliasing -use_fast_math")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   SET(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   SET(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(OpenCV 3.4.0 REQUIRED PATHS /usr/local/opencv3.4)
find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

cuda_include_directories(
  ${CUDA_TOOLKIT_ROOT_DIR}/samples/common/inc
)

cuda_add_library(${PROJECT_NAME} SHARED
src/System.cc
src/Tracking.cc
src/LocalMapping.cc
src/LoopClosing.cc
src/ORBextractor.cc
src/ORBmatcher.cc
src/FrameDrawer.cc
src/Converter.cc
src/MapPoint.cc
src/KeyFrame.cc
src/Map.cc
src/MapDrawer.cc
src/Optimizer.cc
src/PnPsolver.cc
src/Frame.cc
src/KeyFrameDatabase.cc
src/Sim3Solver.cc
src/Initializer.cc
src/Viewer.cc
src/cuda/Allocator_gpu.cu
src/cuda/Fast_gpu.cu
src/cuda/Orb_gpu.cu
src/cuda/Cuda.cu
)

#message(STATUS "OpenCV_INCLUDE_DIR is: " ${OpenCV_INCLUDE_DIR})
#message(STATUS "OpenCV_LINK_DIRECTORIES is: " ${OpenCV_LINK_DIRECTORIES})
#message(FATAL_ERROR "OpenCV_INCLUDE_DIR is: " ${OpenCV_INCLUDE_DIR})

target_link_libraries(${PROJECT_NAME}
    ${OpenCV_LIBS}
    ${EIGEN3_LIBS}
    ${Pangolin_LIBRARIES}
    ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
    ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
    ${CUDA_LIBRARIES}
    /usr/local/cuda/lib64/libnvToolsExt.so
)

#SET(NO_EXAMPLES true)

# Build examples
if(NO_EXAMPLES)
    message(STATUS "Don't build examples")
else ()
    message(STATUS "Build examples")

    # message(STATUS "OpenCV_LIBS: ")
    # message(${OpenCV_LIBS})
    message(${CUDA_LIBRARIES})
    # message(STATUS "OpenCV_INCLUDE_DIR: " ${OpenCV_INCLUDE_DIR})
    # message(STATUS "OpenCV_DIR: " ${OpenCV_DIR})
    # message(STATUS "OpenCV_VERSION" ${OpenCV_VERSION})

    add_executable(rgbd_tum Examples/RGB-D/rgbd_tum.cc)
    target_link_libraries(rgbd_tum ${PROJECT_NAME} ${OpenCV_LIBS} ${CUDA_LIBRARIES} /usr/local/cuda/lib64/libnvToolsExt.so)

    add_executable(stereo_kitti Examples/Stereo/stereo_kitti.cc)
    target_link_libraries(stereo_kitti ${PROJECT_NAME} ${OpenCV_LIBS} /usr/local/cuda/lib64/libnvToolsExt.so)

    add_executable(mono_tum Examples/Monocular/mono_tum.cc)
    target_link_libraries(mono_tum ${PROJECT_NAME} ${OpenCV_LIBS} /usr/local/cuda/lib64/libnvToolsExt.so)

    add_executable(mono_kitti Examples/Monocular/mono_kitti.cc)
    target_link_libraries(mono_kitti ${PROJECT_NAME} /usr/local/cuda/lib64/libnvToolsExt.so)

endif() # Build Examples

unset(CMAKE_RUNTIME_OUTPUT_DIRECTORY)
add_subdirectory(gpu)
