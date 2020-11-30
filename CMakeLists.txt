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

option(BUILD_SHARED_LIBS "Build shared library" OFF)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
 
if(WIN32 OR MSVC)
  set(flags /arch:AVX2 /openmp:experimental)
  add_compile_options(
    "$<$<COMPILE_LANGUAGE:CXX>:${flags}>"
    "$<$<COMPILE_LANGUAGE:C>:${flags}>")
  set(debug_flags /W4 /Od)
  set(release_flags /W3 /O2 /Ob2 /Oi /Ot /GL /GF /Gy /fp:fast)
else()
  set(flags -mfma -mavx2 -march=znver2 -fopenmp -fPIC)
  add_compile_options(
    "$<$<COMPILE_LANGUAGE:CXX>:${flags}>"
    "$<$<COMPILE_LANGUAGE:C>:${flags}>")
  set(debug_flags -g -Wextra)
  set(release_flags -Ofast -funroll-loops -ftree-vectorize -Wno-error -ffast-math)
endif()

MACRO(VECTORCLASS_TARGET_ENABLE_CXX17 target)
  SET_TARGET_PROPERTIES(${target} PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS ON)
  target_compile_options(${target} PRIVATE
    "$<$<CONFIG:RELEASE>:${release_flags}>"
    "$<$<CONFIG:DEBUG>:${debug_flags}>")
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