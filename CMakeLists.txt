cmake_minimum_required(VERSION 3.16)
project(grpc-cmake)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O3 -Wall")
set(CMAKE_CXX_STANDARD 11)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)


add_executable(greeter_server
    ${CMAKE_CURRENT_SOURCE_DIR}/src/greeter_server.cc
)

target_link_libraries(greeter_server
    PRIVATE
        protos
)
add_subdirectory(protos)