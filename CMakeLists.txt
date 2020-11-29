cmake_minimum_required(VERSION 3.7...3.19)

if(${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

#####
# Enables link_directories() treat paths relative
# to the source dir.
if(POLICY CMP0015)
    cmake_policy(SET CMP0015 NEW)
endif(POLICY CMP0015)

#####
# Enables project() command manages VERSION variables.
if(POLICY CMP0048)
    cmake_policy(SET CMP0048 NEW)
endif(POLICY CMP0048)

set(VECTORCLASS_VERSION_MAJOR 2)
set(VECTORCLASS_VERSION_MINOR 0)
set(VECTORCLASS_VERSION_PATCH 0)
set(VECTORCLASS_VERSION_NOTE "")

set(VECTORCLASS_VERSION ${VECTORCLASS_VERSION_MAJOR}.${VECTORCLASS_VERSION_MINOR}.${VECTORCLASS_VERSION_PATCH})
math(EXPR VECTORCLASS_VERSION_NUMBER "10000*${VECTORCLASS_VERSION_MAJOR} + 100*${VECTORCLASS_VERSION_MINOR} + ${VECTORCLASS_VERSION_PATCH}")

project(vectorclass VERSION ${VECTORCLASS_VERSION} LANGUAGES CXX)

INCLUDE(GNUInstallDirs)
INCLUDE(CheckCSourceCompiles)
INCLUDE(CheckCXXSourceCompiles)

#####
# Change the default build type from Debug to Release, while still
# supporting overriding the build type.
#
# The CACHE STRING logic here and elsewhere is needed to force CMake
# to pay attention to the value of these variables.
if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type specified; defaulting to CMAKE_BUILD_TYPE=Release.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
else(NOT CMAKE_BUILD_TYPE)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        message("==========================================================================================")
        message(STATUS "Build type: Debug. Performance will be terrible!")
        message(STATUS "Add -DCMAKE_BUILD_TYPE=Release to the CMake command line to get an optimized build.")
        message("==========================================================================================")
    endif(CMAKE_BUILD_TYPE STREQUAL "Debug")
endif(NOT CMAKE_BUILD_TYPE)

option(BUILD_SHARED_LIBS "Build shared library" OFF)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

MACRO(VECTORCLASS_TARGET_ENABLE_CXX17 target)
  if(WIN32 OR MSVC)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
      # warning level 4 and all warnings as errors
      TARGET_COMPILE_OPTIONS(${target} PRIVATE /W4 /fp:fast /arch:AVX2 /openmp:experimental)
    else()
      TARGET_COMPILE_OPTIONS(${target} PRIVATE /W3 /O2 /Ob2 /Oi /Ot /GL /GF /Gy /fp:fast /arch:AVX2 /openmp:experimental)
    endif()
  else()
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
      # warning level 4 and all warnings as errors
      TARGET_COMPILE_OPTIONS(${target} PRIVATE -g -Wextra -mavx2 -mfma -march=znver2 -fopenmp -fPIC)
    else()
      if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        TARGET_COMPILE_OPTIONS(${target} PRIVATE -Ofast -funroll-loops -ftree-vectorize -Wno-error -mavx2 -mfma -march=znver2 -fopenmp -ffast-math -fPIC)
      else()
        TARGET_COMPILE_OPTIONS(${target} PRIVATE -Ofast -funroll-loops -unroll-aggressive -ftree-vectorize -Wno-error -mavx2 -mfma -march=znver2 -fopenmp -ffast-math -fPIC)
      endif()
    endif()
  endif()
  SET_TARGET_PROPERTIES(${target} PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS ON)
ENDMACRO()

# Files
#--------------------------------------
set(SRC_FILES
	instrset.h	
	instrset_detect.cpp
	vectorclass.h
	vectorf128.h
	vectorf256.h
	vectorf256e.h
	vectorf512.h
	vectorf512e.h
	vectori128.h
	vectori256.h
	vectori256e.h
	vectori512.h
	vectori512e.h
	vectori512s.h
	vectori512se.h
	vectormath_common.h
	vectormath_exp.h
	vectormath_hyp.h
	vectormath_lib.h
	vectormath_trig.h
	vector_convert.h
	add-on/random/ranvec1.h
	add-on/random/ranvec1.cpp
	add-on/random/physseed.cpp
)

# Library
#--------------------------------------
add_library(${PROJECT_NAME} STATIC ${SRC_FILES})
VECTORCLASS_TARGET_ENABLE_CXX17(${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
     PUBLIC 
        $<INSTALL_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>    
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)