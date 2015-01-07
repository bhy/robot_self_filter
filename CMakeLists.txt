cmake_minimum_required(VERSION 2.8.6)
project(robot_self_filter)


# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries

set(ROS_BUILD_TYPE Release)

find_package(catkin REQUIRED COMPONENTS roscpp sensor_msgs tf tf_conversions visualization_msgs geometric_shapes urdf resource_retriever pcl_ros pcl_conversions shape_msgs filters)

set(ROS_BUILD_TYPE Release)

find_package(Boost REQUIRED COMPONENTS signals)

set(IS_ASSIMP3 true)

find_package(assimp QUIET)
if ( NOT ASSIMP_FOUND )
  find_package(Assimp QUIET)
  if ( NOT ASSIMP_FOUND )
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(ASSIMP assimp)
  endif()
endif()
if( ASSIMP_FOUND )
  if( NOT ${ASSIMP_VERSION} VERSION_LESS "2.0.1150" )
    add_definitions(-DASSIMP_UNIFIED_HEADER_NAMES)
  endif()
  if( NOT ${ASSIMP_VERSION} VERSION_LESS "2.0.885" )
    add_definitions(-DASSIMP_EXPORT_API)
  endif()
  include_directories(${ASSIMP_INCLUDE_DIRS})
  link_directories(${ASSIMP_LIBRARY_DIRS})
else()
  message(STATUS "could not find assimp (perhaps available thorugh ROS package?), so assuming assimp v2")
  set(ASSIMP_LIBRARIES assimp)
  set(ASSIMP_LIBRARY_DIRS)
  set(ASSIMP_CXX_FLAGS)
  set(ASSIMP_CFLAGS_OTHER)
  set(ASSIMP_LINK_FLAGS)
  set(ASSIMP_INCLUDE_DIRS)
endif()

option(WITH_OPENMP      "Build with OpenMP support for multithreading"                  ON)
# add OpenMP support
if (WITH_OPENMP)
    find_package(OpenMP QUIET)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")
    if (NOT OPENMP_FOUND)
        set(WITH_OPENMP OFF)
    endif()
endif()

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES ptu_d46
   CATKIN_DEPENDS roscpp sensor_msgs
#  DEPENDS library
)

include_directories(include ${catkin_INCLUDE_DIRS})

add_library(robot_self_filter src/self_mask.cpp)
target_link_libraries(robot_self_filter ${catkin_LIBRARIES} ${ASSIMP_LIBRARIES})

add_executable (test_filter src/test_filter.cpp)
target_link_libraries (test_filter robot_self_filter  ${catkin_LIBRARIES} ${ASSIMP_LIBRARIES})

add_executable(self_filter src/self_filter.cpp)
target_link_libraries(self_filter ${catkin_LIBRARIES} ${ASSIMP_LIBRARIES}  robot_self_filter)
