# project settings
cmake_minimum_required(VERSION 3.28)
project(tick_to_trade_esw LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
add_compile_options(-Wall -Wextra -Wpedantic)

# subdirectories
add_subdirectory(apps/itch_stats)
