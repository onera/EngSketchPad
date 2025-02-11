# ------------------------------------------------------------------------------
# Compilation
# ------------------------------------------------------------------------------
set(src_dir      ${PROJECT_SOURCE_DIR}/src/)
set(src_dir_util ${PROJECT_SOURCE_DIR}/util/)
set(include_dir  ${PROJECT_SOURCE_DIR})

file(GLOB_RECURSE src_files
  CONFIGURE_DEPENDS "${src_dir}/*.cxx" "${src_dir}/*.cpp" "${src_dir}/*.c"
  CONFIGURE_DEPENDS "${src_dir_util}/emp.c"
  CONFIGURE_DEPENDS "${src_dir_util}/evaluate.c"
  CONFIGURE_DEPENDS "${src_dir_util}/regQuads.c"
  CONFIGURE_DEPENDS "${src_dir_util}/rational.c"
)

file(GLOB_RECURSE src_files_util
  CONFIGURE_DEPENDS "${src_dir_util}/uvmap/*.c"
  CONFIGURE_DEPENDS "${src_dir_util}/egadsUVmap.c"
)

list(FILTER src_files EXCLUDE REGEX ".*fgadsHLevel.c$")
list(FILTER src_files EXCLUDE REGEX "./OCC/*.cpp$")

list(FILTER src_files_util EXCLUDE REGEX ".*evaluateNR.c$")

add_library(egads SHARED ${src_files})
add_library(uvmap SHARED ${src_files_util})

set_source_files_properties("${src_dir_util}/evaluate.c" TARGET_DIRECTORY egads PROPERTIES LANGUAGE CXX)
set_source_files_properties("${src_dir_util}/rational.c" TARGET_DIRECTORY egads PROPERTIES LANGUAGE CXX)


target_include_directories(egads
  PUBLIC
    $<BUILD_INTERFACE:${include_dir}/include>
    $<INSTALL_INTERFACE:include/>
  PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/util/uvmap>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/util/>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/>
    ${OpenCASCADE_INCLUDE_DIR}
)

target_include_directories(uvmap PRIVATE
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/util/uvmap>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/util/>
  ${OpenCASCADE_INCLUDE_DIR}
)


target_link_libraries(egads
  PUBLIC
  uvmap
  ${OpenCASCADE_LIBRARIES}
)


# ------------------------------------------------------------------------------
# Installation
# ------------------------------------------------------------------------------
# Installation paths
install(TARGETS egads EXPORT egadsTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)
# Installation paths
install(TARGETS uvmap EXPORT uvmapTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# Install headers
install(DIRECTORY ${include_dir}/include
  DESTINATION include
  FILES_MATCHING
    PATTERN "*.h"
    PATTERN "*.hxx"
)

# Install cmake package files (${target}Targets.cmake and ${target}Config.cmake)
install(EXPORT egadsTargets
  FILE egadsConfig.cmake
  NAMESPACE egads::
  DESTINATION lib/cmake/egads
)
install(EXPORT uvmapTargets
  FILE egadsConfig.cmake
  NAMESPACE egads::
  DESTINATION lib/cmake/egads
)

add_library(EGADS::EGADS ALIAS egads)