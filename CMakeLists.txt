# Author: Alexander Böhn (with Félix C. Morency)
# © 2011.10 -- GPL, Motherfuckers
cmake_minimum_required(VERSION 3.0)
project(libhiptext LANGUAGES C CXX)

set(libhiptext_VERSION_MAJOR "0")
set(libhiptext_VERSION_MINOR "6")
set(libhiptext_VERSION_PATCH "1")

# If the build script is called from a parent project,
# use the configuration from there.
if(NOT COMMAND if_defined_set)
    set(cmake_directory ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
endif()

# Set the CMake module path to the project cmake directory. This location will
# first be searched for external CMake packages.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${cmake_directory})

# Include the library build configuration. This will also include the custom
# macro defined for the project.
include(build_config)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/gtest)

# Set the location of the library configuration file if it has not already been
# set. This allow the library to be used by an external project without
# overwriting the variable.
if_defined_set(${PROJECT_NAME}_DIR ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Load the Ragel extensions and find the package
include(FindRAGEL)
find_package(RAGEL REQUIRED)

# add ragel targets
RAGEL_TARGET(css_color
             css_color.rl
             ${CMAKE_CURRENT_BINARY_DIR}/css_color.cc)

RAGEL_TARGET(pixel_parse
             pixel_parse.rl
             ${CMAKE_CURRENT_BINARY_DIR}/pixel_parse.cc)

add_custom_target("css_color"
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/css_color.cc)

add_custom_target("pixel_parse"
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/pixel_parse.cc)


# find other packages
# include(FindFFMPEG)
include(FindGflags)
include(FindGlog)
# find_package(FFMPEG REQUIRED)
find_package(Gflags REQUIRED)
find_package(Glog REQUIRED)
find_package(Freetype REQUIRED)
find_package(JPEG REQUIRED)
find_package(PNG REQUIRED)
find_package(ZLIB REQUIRED)
# find_package(${PROJECT_NAME} REQUIRED)

include_directories(
    # ${FFMPEG_INCLUDE_DIR}
    ${GFLAGS_INCLUDE_DIR}
    ${GLOG_INCLUDE_DIR}
    ${FREETYPE_INCLUDE_DIRS}
    ${PNG_INCLUDE_DIR}
    ${JPEG_INCLUDE_DIR}
    ${ZLIB_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    "/usr/include"
    "/usr/local/include"
)

add_definitions(
    -MD -x c++
    -fno-exceptions -fno-rtti
    -Wall -Wextra -fstack-protector-all
    -g -O3 -std=c++11
    -mtune=native -march=native)

set(srcs
    "artiste.cc"
    "charquantizer.cc"
    ${RAGEL_css_color_OUTPUT_SOURCE}
    "font.cc"
    "graphic.cc"
    "hiptext.cc"
    "jpeg.cc"
    "macterm.cc"
    # "movie.cc"
    "pixel.cc"
    ${RAGEL_pixel_parse_OUTPUT_SOURCE}
    ${CMAKE_CURRENT_BINARY_DIR}/pixel_parse.cc
    "png.cc"
    "sixelprinter.cc"
    "termprinter.cc"
    "unicode.cc"
    "xterm256.cc")

set(hdrs
    "artiste.h"
    "charquantizer.h"
    ${RAGEL_css_color_OUTPUT_HEADER}
    "font.h"
    "graphic.h"
    "jpeg.h"
    "macterm.h"
    # "movie.h"
    "pixel.h"
    ${RAGEL_pixel_parse_OUTPUT_HEADER}
    "png.hh"
    "sixelprinter.h"
    "termprinter.h"
    "unicode.h"
    "xterm256.h")

# add_library(libhiptext STATIC ${srcs} ${hdrs})
add_library(hiptext SHARED ${srcs} ${hdrs})

# set_target_properties(libhiptext
#     PROPERTIES ARCHIVE_OUTPUT_NAME "hiptext")
set_target_properties(hiptext
    PROPERTIES LIBRARY_OUTPUT_NAME "hiptext")

# add_dependencies(libhiptext css_color)
# add_dependencies(libhiptext pixel_parse)
add_dependencies(hiptext "css_color")
add_dependencies(hiptext "pixel_parse")

# target_link_libraries(libhiptext)
target_link_libraries(hiptext
    glog gflags
    png z m
    # ${FFMPEG_avformat_LIBRARY}
    # ${FFMPEG_avcodec_LIBRARY}
    # ${FFMPEG_avutil_LIBRARY}
    # ${FFMPEG_swscale_LIBRARY}
    ${FREETYPE_LIBRARIES}
    ${JPEG_LIBRARIES})


# install(TARGETS libhiptext DESTINATION lib)
install(TARGETS hiptext DESTINATION lib)
install(DIRECTORY ./ DESTINATION include/libhiptext
    PATTERN "gtest" EXCLUDE
    FILES_MATCHING PATTERN "./*.h")
