cmake_minimum_required(VERSION 3.20)
project(ypass LANGUAGES CXX)

include(cmake/ProjectOptions.cmake)
include(cmake/CompilerOptions.cmake)
include(cmake/Utils.cmake)

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../libs)

add_subdirectory(modules/common)
add_subdirectory(modules/server)
add_subdirectory(modules/client)
