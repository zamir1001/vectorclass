cmake_minimum_required(VERSION 3.1)

if("${CMAKE_BUILD_TYPE}" STREQUAL "")
    message(STATUS "CMAKE_BUILD_TYPE is unset, defaulting to Release")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel RelWithAssert ...")
endif()

set(VECTORCLASS_VERSION_MAJOR 2
set(VECTORCLASS_VERSION_MINOR 0)
set(VECTORCLASS_VERSION_PATCH 0)
set(VECTORCLASS_VERSION_NOTE "")

set(VECTORCLASS_VERSION ${VECTORCLASS_VERSION_MAJOR}.${VECTORCLASS_VERSION_MINOR}.${VECTORCLASS_VERSION_PATCH})
math(EXPR VECTORCLASS_VERSION_NUMBER "10000*${VECTORCLASS_VERSION_MAJOR} + 100*${VECTORCLASS_VERSION_MINOR} + ${VECTORCLASS_VERSION_PATCH}")

message(STATUS "Using CMake version ${CMAKE_VERSION}")

INCLUDE(GNUInstallDirs)
INCLUDE(CheckCSourceCompiles)

project(vectorclass VERSION ${VECTORCLASS_VERSION} LANGUAGES CXX)

MACRO(VECTORCLASS_TARGET_ENABLE_CXX17 target)
  IF(MSVC)
    TARGET_COMPILE_OPTIONS(${target} PRIVATE /arch:AVX2 /openmp:experimental)
  ELSE()
    TARGET_COMPILE_OPTIONS(${target} PRIVATE -mavx2 -mfma -march=znver2 -fopenmp)
  ENDIF()
  SET_TARGET_PROPERTIES(${target} PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS OFF)
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
add_library(${PROJECT_NAME} STATIC $(SRC_FILES})
VECTORCLASS_TARGET_ENABLE_CXX17(${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
     PUBLIC 
        $<INSTALL_INTERFACE:include>    
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)
