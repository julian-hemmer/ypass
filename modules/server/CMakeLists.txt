cmake_minimum_required(VERSION 3.20)
project(server LANGUAGES CXX)

###

file(GLOB_RECURSE SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

###

add_executable(server ${SOURCES})
apply_project_settings(server)

target_include_directories(server PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/)

###

target_link_libraries(server PRIVATE common)
