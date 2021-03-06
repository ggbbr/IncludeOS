cmake_minimum_required (VERSION 3.5.1)
project (userland_tests C CXX)

# create OS version string from git describe (used in CXX flags)
execute_process(COMMAND git describe --tags --dirty
	        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/..
	        OUTPUT_VARIABLE OS_VERSION)
string(STRIP ${OS_VERSION} OS_VERSION)

#set(CMAKE_CXX_STANDARD 17)
set(COMMON "-g -O2 -march=native -Wall -Wextra")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 ${COMMON}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON}")

option(PERFORMANCE "Enable maximum performance" OFF)
option(DEBUGGING "Enable debugging" OFF)
option(GPROF "Enable profiling with gprof" OFF)
option(PGO_ENABLE "Enable guided profiling (PGO)" OFF)
option(PGO_GENERATE "PGO is in profile generating mode" ON)
option(SANITIZE "Enable undefined- and address sanitizers" OFF)
option(ENABLE_LTO "Enable LTO for use with Clang/GCC" OFF)
option(CUSTOM_BOTAN "Enable building with a local Botan" OFF)

if (PERFORMANCE)
  if (DEBUGGING)
    message(FATAL_ERROR "You can not mix PERFORMANCE and DEBUGGING")
  endif()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Ofast")
endif()

if (DEBUGGING)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0")
endif()

if (ENABLE_LTO)
  if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -flto")
    set(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -flto")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -flto=thin")
    set(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -flto=thin")
  endif()
endif()

if(GPROF)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg -fno-omit-frame-pointer")
endif()

if (PGO_ENABLE)
  if (PGO_GENERATE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-dir=$ENV{HOME}/pgo -fprofile-generate")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-dir=$ENV{HOME}/pgo -fprofile-use")
  endif()
endif()

if(SANITIZE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=undefined -fsanitize=address")
endif()

if(CUSTOM_BOTAN)
  include_directories("/usr/local/botan/include/botan-2")
endif()

add_definitions(-DARCH="x86_64" -DARCH_x86_64)
add_definitions(-DOS_VERSION=\"${OS_VERSION}\")
add_definitions(-DOS_TERMINATE_ON_CONTRACT_VIOLATION)
add_definitions(-DARP_PASSTHROUGH)
add_definitions(-DNO_DEBUG)
add_definitions(-DUSERSPACE_LINUX)

include_directories(../api)
include_directories(../mod)
include_directories(../mod/GSL)

add_subdirectory(src)
add_subdirectory(src/plugins)
add_subdirectory(userspace)
