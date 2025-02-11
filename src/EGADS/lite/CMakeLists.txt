# ------------------------------------------------------------------------------
# Compilation
# ------------------------------------------------------------------------------
set(src_dir      ${PROJECT_SOURCE_DIR}/src/)
set(src_dir_util ${PROJECT_SOURCE_DIR}/util/)
set(src_dir_lite ${PROJECT_SOURCE_DIR}/lite/)
set(include_dir  ${PROJECT_SOURCE_DIR})


file(GLOB_RECURSE src_files_util
  CONFIGURE_DEPENDS "${src_dir_util}/uvmap/*.c"
  CONFIGURE_DEPENDS "${src_dir_util}/egadsUVmap.c"
)

file(GLOB_RECURSE src_files_lite
  CONFIGURE_DEPENDS "${src_dir_lite}/*.c"
  CONFIGURE_DEPENDS "${src_dir}/egadsTess.c"
  CONFIGURE_DEPENDS "${src_dir}/egadsTris.c"
  CONFIGURE_DEPENDS "${src_dir}/egadsQuads.c"
  CONFIGURE_DEPENDS "${src_dir}/egadsTessInp.c"
  CONFIGURE_DEPENDS "${src_dir}/egadsRobust.c"
  CONFIGURE_DEPENDS "${src_dir}/egadsEffect.c"
  CONFIGURE_DEPENDS "${src_dir_util}/emp.c"
  CONFIGURE_DEPENDS "${src_dir_util}/evaluate_lite.c"
  CONFIGURE_DEPENDS "${src_dir_util}/regQuads.c"
  CONFIGURE_DEPENDS "${src_dir_util}/rational_lite.c"
)


list(FILTER src_files_lite EXCLUDE REGEX ".*liteDevice.c$")
list(FILTER src_files_lite EXCLUDE REGEX ".*liteTest.c$")

add_library(egadslite SHARED ${src_files_lite})

target_compile_definitions(egadslite PRIVATE -DLITE)
set_source_files_properties("${src_dir_util}/evaluate.c" TARGET_DIRECTORY egadslite PROPERTIES LANGUAGE C)
set_source_files_properties("${src_dir_util}/rational.c" TARGET_DIRECTORY egadslite PROPERTIES LANGUAGE C)

target_include_directories(egadslite
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/../../include>
    $<INSTALL_INTERFACE:include/>
  PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/util/uvmap>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/util/>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/lite/>
    $<BUILD_INTERFACE:${include_dir}/include>
    ${OpenCASCADE_INCLUDE_DIR}
)


target_link_libraries(egadslite
  PUBLIC
  ${OpenCASCADE_LIBRARIES}
)


# ------------------------------------------------------------------------------
# Installation
# ------------------------------------------------------------------------------
# Installation paths
install(TARGETS egadslite EXPORT egadsliteTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# Install headers
# install(DIRECTORY ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}
install(DIRECTORY ${include_dir}/include
  DESTINATION include
  FILES_MATCHING
    PATTERN "*.h"
    PATTERN "*.hxx"
)

# Install cmake package files (${target}Targets.cmake and ${target}Config.cmake)
install(EXPORT egadsliteTargets
  FILE egadsConfig.cmake
  NAMESPACE egads::
  DESTINATION lib/cmake/egads
)

add_library(egads::egadslite ALIAS egadslite)