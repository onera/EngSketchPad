# ------------------------------------------------------------------------------
# General CMake settings
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
cmake_policy(SET CMP0074 NEW) # force find_package to take <PackageName>_ROOT variables into account

if(NOT DEFINED PROJECT_ROOT)
  set(PROJECT_ROOT ${CMAKE_SOURCE_DIR} CACHE PATH "Root directory, where the submodules are populated")
endif()

# Bootstrap project_utils
set(sub_repo_path "${PROJECT_ROOT}/external/project_utils")
file(GLOB sub_repo_files ${sub_repo_path}/*)
list(LENGTH sub_repo_files sub_repo_nb_files)
if(sub_repo_nb_files EQUAL 0)
  message(FATAL_ERROR
    "${sub_repo_path} is empty.
    Maybe you forgot to initialize it with \"git submodule update --init\""
  )
endif()
set(PROJECT_UTILS_DIR ${PROJECT_ROOT}/external/project_utils)
set(PROJECT_UTILS_CMAKE_DIR ${PROJECT_UTILS_DIR}/cmake)
list(APPEND CMAKE_MODULE_PATH "${PROJECT_UTILS_CMAKE_DIR}/find_package") # for custom Find*.cmake files
include(${PROJECT_UTILS_CMAKE_DIR}/dependency_management.cmake) # project_add_subdirectory, project_find_package and target_install

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")


# ------------------------------------------------------------------------------
# Project
# ------------------------------------------------------------------------------
project(
  EngSketchPad VERSION 1.24
  DESCRIPTION " "
  LANGUAGES CXX C
)


## Compiler flags
### C++ standard
set(SUPPORTED_CXX_STANDARDS 17 20)
if(NOT DEFINED CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20 CACHE STRING "C++ standard to compile against")
endif()
if(NOT CMAKE_CXX_STANDARD IN_LIST SUPPORTED_CXX_STANDARDS)
  message(FATAL_ERROR "Supported CXX standards are: ${SUPPORTED_CXX_STANDARDS}.")
endif()

set(CMAKE_CXX_EXTENSIONS        OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON )
### fPIC
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
### Compiler-dependent flags
include(${PROJECT_UTILS_CMAKE_DIR}/default_flags.cmake)
### Default build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE) # default to Release
endif()
### Additionnal build types
include(${PROJECT_UTILS_CMAKE_DIR}/additional_build_types.cmake)


# ------------------------------------------------------------------------------
# Dependencies
# ------------------------------------------------------------------------------
# ## System dependencies ###
# if (NOT TARGET Python::Python OR NOT TARGET Python::NumPy)
#   project_find_package(Python REQUIRED COMPONENTS Development NumPy)
# endif()
# if (NOT TARGET MPI::MPI_CXX)
#   project_find_package(MPI REQUIRED COMPONENTS CXX)
# endif()
# if (NOT TARGET Mpi4Py::Mpi4Py)
#   project_find_package(Mpi4Py REQUIRED)
# endif()
# if (NOT TARGET EGADS::EGADS)
#   project_find_package(EGADS REQUIRED)
# endif()
# if (NOT TARGET tetgen::tetgen)
#   project_find_package(tetgen REQUIRED CONFIG)
# endif()

## Dependencies built from source ##
include(${PROJECT_UTILS_CMAKE_DIR}/find_or_fetch.cmake)
find_or_fetch_pybind11()


# ------------------------------------------------------------------------------
# Compilation
# ------------------------------------------------------------------------------
set(src_dir ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}/src/EGADS/src)
message(${PROJECT_SOURCE_DIR})
message(${PROJECT_NAME})
# set(test_dir ${PROJECT_SOURCE_DIR}/test)
set(include_dir ${PROJECT_SOURCE_DIR})

file(GLOB_RECURSE all_src_files
  CONFIGURE_DEPENDS "${src_dir}/*.cxx" "${src_dir}/*.cpp" "${src_dir}/*.c"
)
message(${src_dir})
foreach(X $all_src_files)
  message(${X})
endforeach()

set(src_files ${all_src_files})
# list(FILTER src_files EXCLUDE REGEX ".*\\.test\\.cpp$")
# list(FILTER src_files EXCLUDE REGEX ".*\\.pybind\\.cpp$")

add_library(${PROJECT_NAME} SHARED ${src_files})


target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${include_dir}>
  $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)

# target_link_libraries(${PROJECT_NAME}
#   PUBLIC
#     MPI::MPI_CXX
#     Mpi4Py::Mpi4Py
#     # EGADS::EGADS
#     # pdm::pdm_shared
#     # pdma::pdma_shared
#     # maia::maia
#     # hrewrite::hrewrite
#     # sonics::sonics
#     # refine::refine_core
#     # refine::refine_with_mpi
#     # refine::refine_with_egads
#     # refine::refine_with_egadslite
#     # tetgen::tetgen
# )

# ------------------------------------------------------------------------------
# Create file to source build environnement
# ------------------------------------------------------------------------------
include(${PROJECT_UTILS_CMAKE_DIR}/write_build_env_file.cmake)
write_build_env_file()
# SoNICS specific: define SONICS_TEST_CASES
file(
  APPEND
  ${PROJECT_BINARY_DIR}/source.sh
)
# file(CREATE_LINK ${PROJECT_SOURCE_DIR}/external/sonics_test_suite/sonics_test_suite/cases ${PROJECT_BINARY_DIR}/cases SYMBOLIC)

# # ----------------------------------------------------------------------
# # Copy logging.conf example to build folder
# # ----------------------------------------------------------------------
# if (EXISTS "${PROJECT_BINARY_DIR}/logging.conf")
#   message("File ${PROJECT_BINARY_DIR}/logging.conf already present")
# else()
#   configure_file(
#     ${PROJECT_SOURCE_DIR}/share/logging.conf.example
#     ${PROJECT_BINARY_DIR}/logging.conf
#     @ONLY
#   )
# endif()

# ------------------------------------------------------------------------------
# Installation
# ------------------------------------------------------------------------------
target_install(${PROJECT_NAME})


# ------------------------------------------------------------------------------
# Python and wrapping
# ------------------------------------------------------------------------------
include(${PROJECT_UTILS_CMAKE_DIR}/python_wrapping.cmake)
compile_install_pybind_module(${PROJECT_NAME})
install_python_modules(${PROJECT_NAME})


# ------------------------------------------------------------------------------
# Build documentation
# ------------------------------------------------------------------------------
if(${PROJECT_NAME}_ENABLE_DOCUMENTATION)
  include(${PROJECT_UTILS_CMAKE_DIR}/build_documentation.cmake)
  file(
    APPEND
    ${PROJECT_BINARY_DIR}/source.sh
    "export PYTHONPATH=${PROJECT_SOURCE_DIR}/doc/external/onera-sphinx-theme/source/:$PYTHONPATH"
  )
  build_sphinx_documentation()
endif()


# ------------------------------------------------------------------------------
# Summary
# ------------------------------------------------------------------------------
get_directory_property(top_lvl_cmakelists PARENT_DIRECTORY)
if(top_lvl_cmakelists STREQUAL ${PROJECT_SOURCE_DIR})
  include(FeatureSummary)
  feature_summary(WHAT ALL)
endif()
