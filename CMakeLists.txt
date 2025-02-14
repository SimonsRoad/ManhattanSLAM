cmake_minimum_required(VERSION 2.8)
project(ManhattanSLAM)

IF (NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE Release)
ENDIF ()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3 -march=native")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if (COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    add_definitions(-DCOMPILEDWITHC11)
    message(STATUS "Using flag -std=c++11.")
elseif (COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    add_definitions(-DCOMPILEDWITHC0X)
    message(STATUS "Using flag -std=c++0x.")
else ()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif ()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(OpenCV 3.0 QUIET)
if (NOT OpenCV_FOUND)
    find_package(OpenCV 2.4.3 QUIET)
    if (NOT OpenCV_FOUND)
        message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
    endif ()
endif ()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)
find_package(PCL 1.7 REQUIRED)
find_package(tinyply REQUIRED)

include_directories(
        ${PROJECT_SOURCE_DIR}
        ${PROJECT_SOURCE_DIR}/include
        ${PROJECT_SOURCE_DIR}/g2oAddition
        ${EIGEN3_INCLUDE_DIR}
        ${Pangolin_INCLUDE_DIRS}
        ${PCL_INCLUDE_DIRS}
        ${tinyply_INCLUDE_DIR}
)

include_directories(include/peac)

add_definitions(${PCL_DEFINITIONS})
link_directories(
        ${PCL_LIBRARY_DIRS}
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

add_library(${PROJECT_NAME} SHARED
        src/System.cc
        src/Tracking.cc
        src/LocalMapping.cc
        src/ORBextractor.cc
        src/ORBmatcher.cc
        src/FrameDrawer.cc
        src/Converter.cc
        src/MapPoint.cc
        src/MapLine.cpp
        src/KeyFrame.cc
        src/Map.cc
        src/MapDrawer.cc
        src/Optimizer.cc
        src/PnPsolver.cc
        src/Frame.cc
        src/KeyFrameDatabase.cc
        src/Viewer.cc
        src/3DLineExtractor.cpp
        src/LSDextractor.cpp
        src/LSDmatcher.cpp
        src/PlaneExtractor.cpp
        src/MapPlane.cc
        src/PlaneMatcher.cpp
        src/SurfelFusion.cpp
        src/SurfelMapping.cpp
        )
file(GLOB sources "*.cpp")
target_link_libraries(${PROJECT_NAME}
        ${OpenCV_LIBS}
        ${EIGEN3_LIBS}
        ${Pangolin_LIBRARIES}
        ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
        ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
        ${PCL_LIBRARIES}
        ${tinyply_LIBRARY_DIR}/libtinyply.so
        )

# Build examples

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Example)

add_executable(manhattan_slam Example/manhattan_slam.cc)
target_link_libraries(manhattan_slam ${PROJECT_NAME})

