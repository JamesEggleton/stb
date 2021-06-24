# Copyright (c) 2017-2021, Ruslan Baratov, Rahul Sheth
# All rights reserved.

cmake_minimum_required(VERSION 3.2)
project(stb VERSION 0.0.1)

set(STBCC_GRID_COUNT_X_LOG2 "10" CACHE STRING "STBCC_GRID_COUNT_X_LOG2")
set(STBCC_GRID_COUNT_Y_LOG2 "10" CACHE STRING "STBCC_GRID_COUNT_Y_LOG2")

set(STBDIV_C_INTEGER_DIVISION_TRUNCATES OFF CACHE BOOL "C_INTEGER_DIVISION_TRUNCATES")
set(STBDIV_C_INTEGER_DIVISION_FLOORS OFF CACHE BOOL "C_INTEGER_DIVISION_FLOORS")

set(STBVOX_CONFIG_MODE "0" CACHE STRING "STBVOX_CONFIG_MODE")
set(STBVOX_CONFIG_PREFER_TEXBUFFER OFF CACHE BOOL "STBVOX_CONFIG_PREFER_TEXBUFFER")

add_library(stb STATIC)

# Installation (https://github.com/forexample/package-example) {

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use:
#   * 'TARGETS_EXPORT_NAME'
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

function(GenerateHeader header_name defines)
    file(READ "${header_name}" HEADER_STRING)
    get_filename_component(header_prefix ${header_name} NAME_WE)
    set(GENERATED_HEADER "${generated_dir}/${header_prefix}.h")
    set(HEADER_DEFS "")
    set(HEADER_UNDEFS "")
    foreach(define IN LISTS defines)
        string(APPEND HEADER_DEFS "#define ${define}\n")
        string(REGEX REPLACE "(.*)\ .*" "\\1" undefine "${define}")
        string(APPEND HEADER_UNDEFS "#undef ${undefine}\n")
    endforeach()
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/header.template.in" "${GENERATED_HEADER}")
    install(FILES "${GENERATED_HEADER}" DESTINATION "${include_install_dir}/${PROJECT_NAME}")

    list(LENGTH defines define_count)
    if(define_count GREATER 0)
        string(PREPEND HEADER_STRING "${HEADER_DEFS}")
        file(WRITE "${generated_dir}/${header_prefix}.c" "${HEADER_STRING}")
        target_sources(stb PRIVATE "${generated_dir}/${header_prefix}.c")
    endif()
endfunction()

GenerateHeader(stb.h "STB_DEFINE")
GenerateHeader(stb_c_lexer.h "STB_C_LEXER_IMPLEMENTATION")
GenerateHeader(stb_connected_components.h "STBCC_GRID_COUNT_X_LOG2 ${STBCC_GRID_COUNT_X_LOG2};STBCC_GRID_COUNT_Y_LOG2 ${STBCC_GRID_COUNT_Y_LOG2};STB_CONNECTED_COMPONENTS_IMPLEMENTATION")
set(STB_DIVIDE_DEFINES "STB_DIVIDE_IMPLEMENTATION")
if(STBDIV_C_INTEGER_DIVISION_TRUNCATES)
  list(APPEND STB_DIVIDE_DEFINES "C_INTEGER_DIVISION_TRUNCATES")
  target_compile_definitions(stb PUBLIC "C_INTEGER_DIVISION_TRUNCATES")
endif()
if(STBDIV_C_INTEGER_DIVISION_FLOORS)
  list(APPEND STB_DIVIDE_DEFINES "C_INTEGER_DIVISION_FLOORS")
  target_compile_definitions(stb PUBLIC "C_INTEGER_DIVISION_FLOORS")
endif()
GenerateHeader(stb_divide.h "${STB_DIVIDE_DEFINES}")
GenerateHeader(stb_ds.h "STB_DS_IMPLEMENTATION")
GenerateHeader(stb_dxt.h "STB_DXT_IMPLEMENTATION")
GenerateHeader(stb_easy_font.h "")
GenerateHeader(stb_herringbone_wang_tile.h "STB_HERRINGBONE_WANG_TILE_IMPLEMENTATION")
GenerateHeader(stb_hexwave.h "STB_HEXWAVE_IMPLEMENTATION")
GenerateHeader(stb_image.h "STB_IMAGE_IMPLEMENTATION")
GenerateHeader(stb_image_resize.h "STB_IMAGE_RESIZE_IMPLEMENTATION")
GenerateHeader(stb_image_write.h "STB_IMAGE_WRITE_IMPLEMENTATION")
GenerateHeader(stb_include.h "STB_INCLUDE_IMPLEMENTATION")
GenerateHeader(stb_leakcheck.h "STB_LEAKCHECK_IMPLEMENTATION")
GenerateHeader(stb_perlin.h "STB_PERLIN_IMPLEMENTATION")
GenerateHeader(stb_rect_pack.h "")
GenerateHeader(stb_sprintf.h "")
# stb_textedit has way too many configurable options, bypass implementation generation for now.
install(FILES stb_textedit.h DESTINATION "${include_install_dir}/${PROJECT_NAME}")
# stb_tilemap_editor requires functions to be defined
install(FILES stb_tilemap_editor.h DESTINATION "${include_install_dir}/${PROJECT_NAME}")
GenerateHeader(stb_truetype.h "STB_TRUETYPE_IMPLEMENTATION")
GenerateHeader(stb_vorbis.h "")

set(STBVOX_DEFINES "STB_VOXEL_RENDER_IMPLEMENTATION;STBVOX_CONFIG_MODE ${STBVOX_CONFIG_MODE}")
if(STBVOX_CONFIG_PREFER_TEXBUFFER)
  list(APPEND STBVOX_DEFINES "STBVOX_CONFIG_PREFER_TEXBUFFER")
  target_compile_definitions(stb PUBLIC "STBVOX_CONFIG_PREFER_TEXBUFFER")
endif()
GenerateHeader(stb_voxel_render.h "${STBVOX_DEFINES}")

target_compile_definitions(stb PUBLIC STB_USE_HUNTER)

install(
    TARGETS stb
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }
