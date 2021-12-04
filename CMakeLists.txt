cmake_minimum_required(VERSION 3.19)

project(constemu VERSION 0.0.1)

#TODO Use cmake-utils toolchain for multiple-compiler support when ready
include(dependencies/cmake-utils/cmake-fetch.cmake)

#Required defaults
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

#Explicitly disallow in-source builds
if("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
 message(
  FATAL_ERROR
  "In-source builds are not permitted."
 )
endif()

#Determine whether or not this is the top-level project
if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_CURRENT_LIST_DIR}")
 set(CE_TOP_LEVEL TRUE)
else()
 set(CE_TOP_LEVEL FALSE)
endif()

#Fetch CX
add_latent_dependency(
 NAME CX
 GIT_REPOSITORY https://github.com/Matthewacon/CX.git
 GIT_TAG HEAD
)

#All build flags
set(CE_DEBUG OFF CACHE BOOL "Enable constemu debug build flags")
set(CE_QA OFF CACHE BOOL "Enable constemu sanitizer build flags")
set(CE_UNIT_TESTS OFF CACHE BOOL "Enable constemu unit tests")
set(CE_BENCHMARKS OFF CACHE BOOL "Enable constemu benchmarks")
set(CE_COVERAGE OFF CACHE BOOL "Enable coverage reports for tests")

#Set default compile flags
list(
 APPEND CE_COMPILE_FLAGS
 -Wall
 -Werror
 -Wextra
 -pedantic
 -ftemplate-backtrace-limit=0
 -fno-common
 -Wno-zero-length-array
)

if(CE_DEBUG)
 #Set the `CE_DEBUG` preprocessor flag and add debug build flags
 set(CMAKE_BUILD_TYPE Debug)
 list(
  APPEND CE_COMPILE_FLAGS
  -DCE_DEBUG
  -g
  -O0
  -fno-inline
  -fno-inline-functions
  -fno-omit-frame-pointer
  -fno-lto
  -fstandalone-debug
 )
else()
 #Set release build flags
 set(CMAKE_BUILD_TYPE Release)
 list(
  APPEND CE_COMPILE_FLAGS
  -O3
 )
endif()

#Set the `CE_QA` preprocessor flag and add QA build flags
if(CE_QA)
 # Set QA compiler flags
 list(
  APPEND CE_COMPILE_FLAGS
  -DCE_QA
  -fno-sanitize-recover=null
  -fsanitize=address
  -fsanitize=undefined
  -fsanitize=address-use-after-scope
  -fsanitize=bounds
  -fsanitize=null
 )

 # Set QA linker flags
 list(
  APPEND CE_LINKER_FLAGS
  -fsanitize=address
  -fsanitize=undefined
 )
endif()

#Configure CX
set(CX_ERROR_MSG ON CACHE BOOL "")
set(CX_ERROR_TRACE ON CACHE BOOL "")
set(CX_CONSTEXPR_SEMANTICS ON CACHE BOOL "")
if(CE_DEBUG)
 set(CX_DEBUG ON CACHE BOOL "")
endif()

#Fetch project dependencies
fetch_latent_dependencies()

#Configure build
if(CE_TOP_LEVEL)
 set(CE_OPTION_SCOPE INTERFACE)
else()
 set(CE_OPTION_SCOPE PRIVATE)
endif()
add_library(constemu INTERFACE)
target_include_directories(constemu INTERFACE include ${cx_SOURCE_DIR}/include)
target_compile_options(constemu ${CE_OPTION_SCOPE} ${CX_COMPILE_FLAGS})

#Testing
#TODO Coverage support
if(CE_UNIT_TESTS OR CE_BENCHMARKS)
 add_subdirectory(test)
endif()
