cmake_minimum_required(VERSION 3.16)

project(ORB_SLAM2 CXX)

# Standard settings
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

# Output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)

# Dependencies
find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED NO_MODULE)
find_package(Pangolin REQUIRED)
find_package(yaml-cpp REQUIRED)

# DBoW2 and g2o from Thirdparty folders
add_subdirectory(Thirdparty/DBoW2)
add_subdirectory(Thirdparty/g2o)

# Interface library for common warnings
add_library(orbslam2_warnings INTERFACE)
target_compile_options(orbslam2_warnings INTERFACE
    $<$<CXX_COMPILER_ID:GNU,Clang>:-Wall -Wextra -Wpedantic>
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
)

# Optimization/Architecture flags
set(ORBSLAM2_OPTIMIZATION_FLAGS -O3 -march=native) 

# Create shared library
add_library(${PROJECT_NAME} SHARED
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
)

# Apply warnings and optimization flags
target_link_libraries(${PROJECT_NAME} PRIVATE orbslam2_warnings)
target_compile_options(${PROJECT_NAME} PRIVATE ${ORBSLAM2_OPTIMIZATION_FLAGS})

# Set include directories for the target
target_include_directories(${PROJECT_NAME} PUBLIC
   $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
   $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
   $<INSTALL_INTERFACE:include>
   ${DBoW2_INCLUDE_DIRS}
   ${g2o_INCLUDE_DIRS}
)

# Link dependencies
target_link_libraries(${PROJECT_NAME} 
    PUBLIC ${OpenCV_LIBS}
    PUBLIC Eigen3::Eigen
    PUBLIC yaml-cpp::yaml-cpp
    ${Pangolin_LIBRARIES}
    PUBLIC DBoW2
    PUBLIC g2o
)

# Example executables
add_executable(vslamlab_orbslam2_mono Examples/Monocular/vslamlab_orbslam2_mono.cpp)
target_link_libraries(vslamlab_orbslam2_mono PRIVATE ${PROJECT_NAME})

add_executable(vslamlab_orbslam2_rgbd Examples/RGB-D/vslamlab_orbslam2_rgbd.cpp)
target_link_libraries(vslamlab_orbslam2_rgbd ${PROJECT_NAME})

add_executable(vslamlab_orbslam2_stereo Examples/Stereo/vslamlab_orbslam2_stereo.cpp)
target_link_libraries(vslamlab_orbslam2_stereo ${PROJECT_NAME})

# Install targets and headers
install(TARGETS ${PROJECT_NAME} vslamlab_orbslam2_mono
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(TARGETS ${PROJECT_NAME} vslamlab_orbslam2_rgbd
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(TARGETS ${PROJECT_NAME} vslamlab_orbslam2_stereo
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(DIRECTORY include/ DESTINATION include)