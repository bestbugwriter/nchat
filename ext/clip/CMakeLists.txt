# Clip Library
# Copyright (c) 2015-2021 David Capello

cmake_minimum_required(VERSION 3.16)

project(clip LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

option(CLIP_EXAMPLES "Compile clip examples" off)
option(CLIP_TESTS "Compile clip tests" off)
if(UNIX AND NOT APPLE)
  option(CLIP_X11_WITH_PNG "Compile with libpng to support copy/paste image in png format" on)
endif()

set(CLIP_SOURCES clip.cpp image.cpp)

if(WIN32)
  if(MSVC)
    add_definitions(-D_SCL_SECURE_NO_WARNINGS)
  endif()
  list(APPEND CLIP_SOURCES clip_win.cpp)
elseif(APPLE)
  if(CMAKE_C_COMPILER_ID MATCHES "Clang")
    add_definitions(-fobjc-arc)
  endif()

  execute_process(COMMAND sw_vers -productVersion OUTPUT_VARIABLE OS_VERSION OUTPUT_STRIP_TRAILING_WHITESPACE)
  if(OS_VERSION VERSION_GREATER_EQUAL "10.10.0")
    find_library(COCOA_LIBRARY Cocoa)
  endif()
  if(COCOA_LIBRARY)
    list(APPEND CLIP_SOURCES clip_osx.mm)
  else()
    list(APPEND CLIP_SOURCES clip_none.cpp)
  endif()
elseif(UNIX)
  include(CheckIncludeFiles)
  check_include_files(xcb/xcb.h HAVE_XCB_XLIB_H)

  if(HAVE_XCB_XLIB_H)
    add_definitions(-DHAVE_XCB_XLIB_H)

    if(CLIP_X11_WITH_PNG)
      check_include_files(png.h HAVE_PNG_H)
      if(CLIP_X11_PNG_LIBRARY)
        set(PNG_LIBRARY ${CLIP_X11_PNG_LIBRARY})
      else()
        find_library(PNG_LIBRARY png)
      endif()
      if(HAVE_PNG_H AND PNG_LIBRARY)
        add_definitions(-DHAVE_PNG_H)
      endif()
    endif()

    list(APPEND CLIP_SOURCES clip_x11.cpp)
  else()
    list(APPEND CLIP_SOURCES clip_none.cpp)
  endif()
else()
  list(APPEND CLIP_SOURCES clip_none.cpp)
endif()

add_library(clip ${CLIP_SOURCES})

if(WIN32)
  target_link_libraries(clip shlwapi)

  # MinGW requires the windowscodecs just because CLSIDs are defined
  # in the windowscodecs.a file instead of the wincodec.h file (?!)
  if(MINGW)
    find_library(CLIP_WINDOWSCODECS_LIBRARY windowscodecs)
    if(CLIP_WINDOWSCODECS_LIBRARY)
      target_link_libraries(clip ${CLIP_WINDOWSCODECS_LIBRARY})
    endif()
  endif()
elseif(APPLE)
  if(COCOA_LIBRARY)
    target_link_libraries(clip ${COCOA_LIBRARY})
  endif()
elseif(UNIX)
  if(HAVE_XCB_XLIB_H)
    target_link_libraries(clip xcb pthread)
    if(CLIP_X11_WITH_PNG AND HAVE_PNG_H AND PNG_LIBRARY)
      target_link_libraries(clip ${PNG_LIBRARY})
    endif()
  endif()
endif()

if(CLIP_EXAMPLES)
  add_subdirectory(examples)
endif()

if(CLIP_TESTS)
  enable_testing()
  add_subdirectory(tests)
endif()
