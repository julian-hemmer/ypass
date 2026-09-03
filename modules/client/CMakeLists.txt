cmake_minimum_required(VERSION 3.20)
project(client LANGUAGES CXX)

###

file(GLOB_RECURSE SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

###

add_executable(client ${SOURCES})
apply_project_settings(client)

target_include_directories(client PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/)

###

target_link_libraries(client PRIVATE common)
