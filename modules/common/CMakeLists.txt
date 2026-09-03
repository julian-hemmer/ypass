cmake_minimum_required(VERSION 3.20)
project(common LANGUAGES CXX)

###

file(GLOB_RECURSE SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

###

add_library(common STATIC ${SOURCES})
apply_project_settings(common)

target_include_directories(common PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/)
