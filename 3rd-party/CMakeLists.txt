# Workaround for zlib placing its generated zconf.h file in the build dir,
# and protobuf not knowing so finding the system version instead
include_directories(${CMAKE_BINARY_DIR}/3rd-party/grpc/third_party/zlib)

# Add the subdir here to avoid polluting their cmake scope
add_subdirectory(grpc EXCLUDE_FROM_ALL)

# Generates gRPC and protobuf C++ sources and headers from the given .proto files
#
# generate_grpc_cpp (<SRCS> <DEST> [<ARGN>...])
#
#  SRCS - output list of autogenerated source files
#  DEST - the output directory where the autogenerated files will be created in
#  ARGN - the .proto files
#
function(generate_grpc_cpp SRCS DEST)
  if (NOT ARGN)
    message(SEND_ERROR "Error: generate_grpc_cpp() called without any proto files")
    return()
  endif ()

  set(${SRCS})
  foreach (FIL ${ARGN})
    get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
    get_filename_component(FIL_WE ${FIL} NAME_WE)
    get_filename_component(FIL_DIR ${ABS_FIL} DIRECTORY)

    list(APPEND ${SRCS} "${DEST}/${FIL_WE}.grpc.pb.cc")
    list(APPEND HDRS "${DEST}/${FIL_WE}.grpc.pb.h")
    list(APPEND ${SRCS} "${DEST}/${FIL_WE}.pb.cc")
    list(APPEND HDRS "${DEST}/${FIL_WE}.pb.h")

    add_custom_command(
      OUTPUT "${DEST}/${FIL_WE}.grpc.pb.cc"
      "${DEST}/${FIL_WE}.grpc.pb.h"
      "${DEST}/${FIL_WE}.pb.cc"
      "${DEST}/${FIL_WE}.pb.h"
      COMMAND $<TARGET_FILE:protoc>
      ARGS --grpc_out=${DEST} --cpp_out=${DEST} --proto_path=${FIL_DIR} --plugin=protoc-gen-grpc=$<TARGET_FILE:grpc_cpp_plugin> ${ABS_FIL}
      DEPENDS ${ABS_FIL} protoc grpc_cpp_plugin
      COMMENT "Running gRPC C++ protocol buffer compiler on ${FIL}"
      VERBATIM)
  endforeach ()

  set_source_files_properties(${${SRCS}} ${HDRS} PROPERTIES GENERATED TRUE)
  set(${SRCS} ${${SRCS}} PARENT_SCOPE)
endfunction()

add_library(gRPC INTERFACE)

target_include_directories(gRPC INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/grpc/include
  ${CMAKE_CURRENT_SOURCE_DIR}/grpc/third_party/protobuf/src)

target_link_libraries(gRPC INTERFACE
  grpc++
  libprotobuf
  zlibstatic)

target_compile_options(gRPC INTERFACE "-Wno-unused-parameter")

# YAML C++
# Disable tests here to avoid double-including gtest
option(YAML_CPP_BUILD_TOOLS OFF)
option(YAML_CPP_BUILD_CONTRIB OFF)
option(YAML_CPP_BUILD_TESTS OFF)

add_subdirectory(yaml-cpp EXCLUDE_FROM_ALL)

add_library(yaml INTERFACE)

target_include_directories(yaml INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/yaml-cpp/include)

target_link_libraries(yaml INTERFACE
  yaml-cpp)

# libssh config
add_subdirectory(libssh)

# fmt header only library
add_library(fmt INTERFACE)
target_compile_definitions(fmt INTERFACE FMT_HEADER_ONLY=1)
target_include_directories(fmt INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/fmt/include)

# premock header only library to mock c-functions
add_library(premock INTERFACE)
target_include_directories(premock INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/premock)

# scope_guard header only library
add_library(scope_guard INTERFACE)
target_include_directories(scope_guard INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/scope_guard)

# semver library
add_subdirectory(semver EXCLUDE_FROM_ALL)
target_include_directories(semver INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/semver/include)

# xz-decoder config
add_subdirectory(xz-decoder EXCLUDE_FROM_ALL)

# qhotkey config
add_subdirectory(qhotkey EXCLUDE_FROM_ALL)
