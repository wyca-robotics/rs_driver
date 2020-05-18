cmake_minimum_required(VERSION 3.5)
project(lidar_driver)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

#========================
# Subdirectory
#========================
add_subdirectory(common)
include(CheckSymbolExists)
#========================
# Project details / setup
#========================
set(PROJECT_NAME lidar_driver)
string(TOUPPER ${PROJECT_NAME} PROJECT_NAME_UPPER)
string(TOLOWER ${PROJECT_NAME} PROJECT_NAME_LOWER)
#========================
# debug
#========================

option(RS_POINT_COMPLEX "Select point type of complex" OFF)
message("RS_POINT_COMPLEX is ${RS_POINT_COMPLEX}")

if(RS_POINT_COMPLEX)
message("add RS_POINT_COMPLEX macro!!")
add_definitions(-DRS_POINT_COMPLEX)
endif(RS_POINT_COMPLEX)

add_definitions(-DDEBUG_LEVEL=0)
add_definitions(-DPROJECT_PATH="${PROJECT_SOURCE_DIR}")

cmake_policy(SET CMP0048 NEW)
set(CMAKE_BUILD_TYPE Release)
add_definitions(-O3)
add_definitions(-std=c++14)
add_compile_options(-W)

find_package(PCL QUIET)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

find_package(Qt5 COMPONENTS Widgets REQUIRED)
#========================
# Dependencies Setup
#========================
set(EXTERNAL_LIBS "")
find_package(roscpp 1.12 QUIET)
if(roscpp_FOUND)
  message(=============================================================)
  message("-- ROS Found, Ros Support is turned On!")
  message(=============================================================)
  add_definitions(-DROS_FOUND)
  include_directories(${roscpp_INCLUDE_DIRS})
  set(EXTERNAL_LIBS ${EXTERNAL_LIBS} ${roscpp_LIBRARIES})
else(roscpp_FOUND)
  message(=============================================================)
  message("-- ROS Not Found, Ros Support is turned Off!")
  message(=============================================================)
endif(roscpp_FOUND)

find_package(Protobuf QUIET)
if(Protobuf_FOUND)
  message(=============================================================)
  message("-- Protobuf Found, Protobuf Support is turned On!")
  message(=============================================================)
  add_definitions(-DPROTO_FOUND)
  include_directories(${PROTOBUF_INCLUDE_DIRS})
  set(EXTERNAL_LIBS ${EXTERNAL_LIBS} ${PROTOBUF_LIBRARY})
else(Protobuf_FOUND)
  message(=============================================================)
  message("-- Protobuf Not Found, Protobuf Support is turned Off!")
  message(=============================================================)
endif(Protobuf_FOUND)

#========================
# Build Setup
#========================
include_directories(${PROJECT_SOURCE_DIR})
add_library(adapter SHARED
              adapter/lidar_points_ros_adapter.cpp
              adapter/lidar_packtes_ros_adapter.cpp      
              adapter/lidar_points_proto_adapter.cpp
              adapter/lidar_packets_proto_adapter.cpp
)
target_link_libraries(adapter rs_yaml)
add_executable(demo
            demo/demo.cpp
            manager/sensor_manager.cpp
            )

target_link_libraries(demo
                    adapter
                    ${roscpp_LIBRARIES}
                    ${PROTOBUF_LIBRARY}
                    ${PCL_LIBRARIES}
                    pthread
                    pcap
                       )
if(Protobuf_FOUND)
target_link_libraries(demo proto_msg)
endif(Protobuf_FOUND)

add_executable(ui
            UI/main.cpp
            UI/mainwindow.cpp
            UI/mainwindow.h
            UI/mainwindow.ui
            manager/sensor_manager.cpp

)
target_link_libraries(ui
                    adapter
                    ${roscpp_LIBRARIES}
                    ${PROTOBUF_LIBRARY}
                    ${PCL_LIBRARIES}
                    pthread
                    pcap
                    Qt5::Widgets)
if(Protobuf_FOUND)
target_link_libraries(ui proto_msg)
endif(Protobuf_FOUND)