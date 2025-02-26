# ------------------------------------------------------------------------------
# General CMake settings
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
cmake_policy(SET CMP0074 NEW) # force find_package to take <PackageName>_ROOT variables into account

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")


# ------------------------------------------------------------------------------
# Project
# ------------------------------------------------------------------------------
project(
  egads VERSION 1.24
  DESCRIPTION " "
  LANGUAGES CXX C
)

## Compiler flags
### C++ standard
set(CMAKE_CXX_EXTENSIONS        OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON )
### fPIC
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
### Default build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE) # default to Release
endif()


# ------------------------------------------------------------------------------
# Dependencies
# ------------------------------------------------------------------------------
project_find_package(OpenCASCADE REQUIRED)


# ------------------------------------------------------------------------------
# Compilation
# ------------------------------------------------------------------------------
add_subdirectory(src)
add_subdirectory(lite)


# ------------------------------------------------------------------------------
# Summary
# ------------------------------------------------------------------------------
get_directory_property(top_lvl_cmakelists PARENT_DIRECTORY)
if(top_lvl_cmakelists STREQUAL ${PROJECT_SOURCE_DIR})
  include(FeatureSummary)
  feature_summary(WHAT ALL)
endif()
