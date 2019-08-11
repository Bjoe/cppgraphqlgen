# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.

cmake_minimum_required(VERSION 3.8.2)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.209.tar.gz"
    SHA1 "4c8195aa5407b54abed39f2e7b18d02caca278ca"
    LOCAL
)

project(cppgraphqlgen VERSION 3.0.0)

hunter_add_package(Boost COMPONENTS system filesystem program_options)
hunter_add_package(pegtl)
hunter_add_package(GTest)

set(CMAKE_CXX_STANDARD 17)

set(GRAPHQL_INSTALL_INCLUDE_DIR include CACHE PATH "Header file install directory")
set(GRAPHQL_INSTALL_TOOLS_DIR bin CACHE PATH "schemagen install directory")
set(GRAPHQL_INSTALL_CMAKE_DIR lib/cmake CACHE PATH "CMake config files install directory")

option(BUILD_SHARED_LIBS "Build shared libraries instead of static libs" OFF)

if(WIN32 AND BUILD_SHARED_LIBS)
  # Let CMake figure out the exports for the SHARED library (DLL) on Windows.
  set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()

function(add_bigobj_flag target)
  if(MSVC)
    # MSVC requires the /bigobj flag if the number of sections gets too big.
    target_compile_options(${target} PRIVATE /bigobj)
  endif()
endfunction()

find_package(Threads MODULE REQUIRED)

find_package(pegtl CONFIG)
#find_package(pegtl 3.0.0 QUIET CONFIG)
if(NOT pegtl_FOUND)
  # If a compatible version of PEGTL is not already installed, build and install it from the submodule directory.
  set(PEGTL_BUILD_TESTS OFF CACHE BOOL "Disable PEGTL tests")
  set(PEGTL_BUILD_EXAMPLES OFF CACHE BOOL "Disable PEGTL examples")
  set(PEGTL_INSTALL_INCLUDE_DIR ${GRAPHQL_INSTALL_INCLUDE_DIR} CACHE STRING "Override PEGTL include install directory")
  set(PEGTL_INSTALL_CMAKE_DIR ${GRAPHQL_INSTALL_CMAKE_DIR}/pegtl CACHE STRING "Override PEGTL cmake install directory")
  add_subdirectory(PEGTL)
endif()

add_subdirectory(src)

option(GRAPHQL_UPDATE_SAMPLES "Regenerate the sample schema sources whether or not we're building the tests." ON)

if(GRAPHQL_BUILD_TESTS OR GRAPHQL_UPDATE_SAMPLES)
  add_subdirectory(samples)

  if(GRAPHQL_BUILD_TESTS)
    include(CTest)
    add_subdirectory(test)
  endif()
endif()
