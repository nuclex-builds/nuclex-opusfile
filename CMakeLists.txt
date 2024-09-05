#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  OpusFile
  VERSION 0.12
  DESCRIPTION "Add-on for libopus to decode a single OPUS stream inside an .opus or .ogg file"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# Add OPUS as a sub-project, this is a library built on top of libopus
if(NOT (TARGET Opus))
  add_subdirectory(
    ${PROJECT_SOURCE_DIR}/../nuclex-opus
    ${CMAKE_BINARY_DIR}/nuclex-opus
  )
endif()

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/xiph/opusfile/releases/download/v0.12/opusfile-0.12.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/opusfile-0.12.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=118d8601c12dd6a44f52423e68ca9083cc9f2bfe72da7a8c1acb22a80ae3550b
    STATUS DOWNLOAD_STATUS
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    list(GET DOWNLOAD_STATUS 2 HTTP_CODE)
    message(
      FATAL_ERROR
      "Could not download OposFile sources: ${ERROR_MESSAGE} (HTTP result ${HTTP_CODE})"
    )
  endif()

  message(STATUS "Extracting OpusFile sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/opusfile-0.12.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/opusfile-0.12
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

configure_file(
  "${PROJECT_SOURCE_DIR}/config.h"
  "${PROJECT_SOURCE_DIR}/build/src/config.h"
  COPYONLY
)

# -------------------------------------------------------------------------------------------------

set(
  sourceFiles
  "build/src/info.c"
  "build/src/stream.c"
  "build/src/opusfile.c"
  "build/src/http.c"
  "build/src/wincerts.c"
  "build/src/internal.c"
)
set(
  headerFiles
  "build/src/internal.h"
  "build/src/winerrno.h"
  "build/include/opusfile.h"
)

# -------------------------------------------------------------------------------------------------

add_library(OpusFile STATIC)
add_library(OpusFile::Static ALIAS OpusFile)

target_compile_definitions(
  OpusFile
  PRIVATE HAVE_CONFIG_H
)

target_include_directories(
  OpusFile
  PUBLIC "build/include"
)

target_link_libraries(
  OpusFile
  PRIVATE Opus
)

target_sources(
  OpusFile
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

set_target_properties(OpusFile PROPERTIES PREFIX "")
set_target_properties(OpusFile PROPERTIES OUTPUT_NAME "opusfile")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS OpusFile
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)
