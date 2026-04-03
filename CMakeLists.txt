cmake_minimum_required(VERSION 3.14)
project(redis_rate_limiter VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# 查找依赖
find_package(pybind11 CONFIG REQUIRED)
find_package(Threads REQUIRED)
find_path(HIREDIS_INCLUDE_DIR hiredis/hiredis.h)
find_library(HIREDIS_LIBRARY hiredis)

if (NOT HIREDIS_INCLUDE_DIR OR NOT HIREDIS_LIBRARY)
    message(FATAL_ERROR "hiredis not found. Set HIREDIS_INCLUDE_DIR and HIREDIS_LIBRARY.")
endif()

# 连接池库
add_library(redis_pool STATIC
    src/redis_pool.cpp
    src/sliding_window_limiter.cpp
)

target_include_directories(redis_pool PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${HIREDIS_INCLUDE_DIR}
)

target_link_libraries(redis_pool PUBLIC
    ${HIREDIS_LIBRARY}
    Threads::Threads
)

# Python模块
pybind11_add_module(redis_limiter src/python_binding.cpp)
target_link_libraries(redis_limiter PRIVATE redis_pool)
target_include_directories(redis_limiter PRIVATE ${HIREDIS_INCLUDE_DIR})

# 安装
install(TARGETS redis_limiter DESTINATION .)
