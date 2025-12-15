cmake_minimum_required(VERSION 3.5)
project(imu_utils)

## Compile as C++11, supported in ROS Kinetic and newer
set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++17")
#-DEIGEN_USE_MKL_ALL")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g -fPIC")

# macOS + Conda Environment Support
if(APPLE AND DEFINED ENV{CONDA_PREFIX})
    message(STATUS "Detected macOS conda environment: $ENV{CONDA_PREFIX}")
    
    # Add conda include path FIRST with highest priority
    set(CMAKE_CXX_FLAGS "-isystem $ENV{CONDA_PREFIX}/include/eigen3 -isystem $ENV{CONDA_PREFIX}/include ${CMAKE_CXX_FLAGS}")
    set(CMAKE_C_FLAGS "-isystem $ENV{CONDA_PREFIX}/include ${CMAKE_C_FLAGS}")
    
    # Explicitly tell compiler to skip /opt/homebrew/include to avoid Eigen conflicts
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-command-line-argument")
    
    # Also add to include directories for better IDE support  
    include_directories(BEFORE SYSTEM $ENV{CONDA_PREFIX}/include/eigen3)
    include_directories(BEFORE SYSTEM $ENV{CONDA_PREFIX}/include)
endif()

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED
    roscpp
    std_msgs
    geometry_msgs
    nav_msgs
    code_utils
    )

## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)
find_package(OpenCV REQUIRED)
find_package(Ceres REQUIRED)
find_package(Eigen3  REQUIRED)

include_directories(
  ${EIGEN3_INCLUDE_DIR}
  ${CERES_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

catkin_package()

set(ACC_LIB_SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/src/acc_lib/allan_acc.cpp
    ${PROJECT_SOURCE_DIR}/src/acc_lib/fitallan_acc.cpp
    )
set(GYR_LIB_SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/src/gyr_lib/allan_gyr.cpp
    ${PROJECT_SOURCE_DIR}/src/gyr_lib/fitallan_gyr.cpp
    )

add_executable(imu_an
    src/imu_an.cpp
    ${GYR_LIB_SOURCE_FILES}
    ${ACC_LIB_SOURCE_FILES}
    )

target_link_libraries(imu_an ${catkin_LIBRARIES} ${OpenCV_LIBS} ${CERES_LIBRARIES})
