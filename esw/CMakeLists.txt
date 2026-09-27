# complier info
cmake_minimum_required(VERSION 3.28)
project(itch_stats LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
add_compile_options(-Wall -Wextra -Wpedantic)

# subdirectories
add_subdirectory(apps/itch_stats)
