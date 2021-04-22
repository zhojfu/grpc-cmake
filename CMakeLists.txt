cmake_minimum_required(VERSION 3.16)
project(grpc-cmake)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O3 -Wall")
set(CMAKE_CXX_STANDARD 11)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)


# file(COPY conf DESTINATION ${PROJECT_BINARY_DIR}/output)
# file(COPY data DESTINATION ${PROJECT_BINARY_DIR}/output)
# file(COPY include DESTINATION ${PROJECT_BINARY_DIR}/output)
file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/output/protos)


set(ALL_INCLUDES
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_BINARY_DIR}/output/protos
)
include_directories(${ALL_INCLUDES})


# Find Protobuf and gRPC

set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf CONFIG REQUIRED)
message(STATUS "Using protobuf ${Protobuf_VERSION}")

set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_REFLECTION gRPC::grpc++_reflection)
if(CMAKE_CROSSCOMPILING)
  find_program(_PROTOBUF_PROTOC protoc)
else()
  set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
endif()

# Find gRPC installation
# Looks for gRPCConfig.cmake file installed by gRPC's cmake installation.
find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

set(_GRPC_GRPCPP gRPC::grpc++)
if(CMAKE_CROSSCOMPILING)
  find_program(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
else()
  set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin>)
endif()


message("_PROTOBUF_PROTOC: ${_PROTOBUF_PROTOC}")
message("_GRPC_CPP_PLUGIN_EXECUTABLE: ${_GRPC_CPP_PLUGIN_EXECUTABLE}")


# Generated proto sources
set(_PROTO_SRCS "")
set(_GRPC_SRCS "")
set(_PROTO_OUTPUT ${PROJECT_BINARY_DIR}/output/protos)
file(GLOB PROTO_FILES ${PROJECT_SOURCE_DIR}/protos/*.proto)
foreach(_proto_file ${PROTO_FILES})
    # message("${_proto_file}")
    get_filename_component(_proto_file "${_proto_file}" ABSOLUTE)
    get_filename_component(_proto_file_path "${_proto_file}" PATH)
    get_filename_component(_proto_file_name "${_proto_file}" NAME_WE)
    set(_proto_src "${_PROTO_OUTPUT}/${_proto_file_name}.pb.cc")
    set(_proto_hdr "${_PROTO_OUTPUT}/${_proto_file_name}.pb.h")
    set(_grpc_src "${_PROTO_OUTPUT}/${_proto_file_name}.grpc.pb.cc")
    set(_grpc_hdr "${_PROTO_OUTPUT}/${_proto_file_name}.grpc.pb.h")
    # message("${_proto_src}")
    add_custom_command(
        OUTPUT "${_proto_src}" "${_proto_hdr}" "${_grpc_src}" "${_grpc_hdr}"
        COMMAND ${_PROTOBUF_PROTOC}
        ARGS  --grpc_out "${_PROTO_OUTPUT}"
            --cpp_out "${_PROTO_OUTPUT}"
            -I "${_proto_file_path}"
            --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
            "${_proto_file}"
        DEPENDS "${_proto_file}"
    )
    list(APPEND _PROTO_SRCS "${_proto_src}")
    list(APPEND _GRPC_SRCS "${_grpc_src}")
endforeach()

add_executable(greeter_server
    ${CMAKE_CURRENT_SOURCE_DIR}/src/greeter_server.cc
    ${_PROTO_SRCS}
    ${_GRPC_SRCS}

)

target_link_libraries(greeter_server
    gRPC::grpc++_reflection
    protobuf::libprotobuf
)