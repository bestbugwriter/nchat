# Project
cmake_minimum_required(VERSION 3.16 FATAL_ERROR)
project(libduchat LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 14)

# Ccache
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

# Library
add_library(duchat ${LIBRARY_LINKING_TYPE}
  src/duchat.cpp
  src/duchat.h
)
install(TARGETS duchat DESTINATION ${CMAKE_INSTALL_LIBDIR})

# Common
target_common_config(duchat)

# Headers
target_include_directories(duchat PRIVATE "../common/src")
target_include_directories(duchat PRIVATE "../ncutil/src")

# Compiler flags
set_target_properties(duchat PROPERTIES COMPILE_FLAGS
                      "-Wall -Wextra -Wpedantic -Wshadow -Wpointer-arith \
                       -Wcast-qual -Wno-missing-braces -Wswitch-default \
                       -Wunreachable-code -Wundef -Wuninitialized \
                       -Wcast-align")

# Linking
target_link_libraries(duchat PUBLIC ncutil)
