# ==================================================================
# 2014 by KjellKod.cc. This is PUBLIC DOMAIN to use at your own
# risk and comes with no warranties.
#
# This code is yours to share, use and modify with no strings attached
# and no restrictions or obligations.
# ===================================================================

cmake_minimum_required (VERSION 2.6)
ENABLE_LANGUAGE(CXX)
#set(CMAKE_BUILD_TYPE Release)

project (DeathKnell)
set(PROJECT_SRC ${DeathKnell_SOURCE_DIR}/src)
set(DIR_UNIT_TEST ${DeathKnell_SOURCE_DIR}/test)
set(DIR_3RDPARTY ${DeathKnell_SOURCE_DIR}/3rdparty)


MESSAGE(" PROJECT_SRC = : ${PROJECT_SRC}")
include_directories(${PROJECT_SRC})
SET(ACTIVE_CPP0xx_DIR "Release")

find_library(G3LOG g3logger PATHS /usr/local/probe/lib )
set(LIBS ${G3LOG})

find_library(G3FILE_ROTATE g3logrotate PATHS /usr/local/probe/lib)
list(APPEND LIBS ${G3FILE_ROTATE})

find_library(FILEIO FileIO PATHS /usr/local/probe/lib)
list(APPEND LIBS ${FILEIO})

find_library(BOOST_THREAD boost_thread PATHS /usr/local/probe/lib )
list(APPEND LIBS ${BOOST_THREAD})

find_library(BOOST_SYSTEM boost_thread PATHS /usr/local/probe/lib )
list(APPEND LIBS ${BOOST_SYSTEM})

find_library(TCMALLOC tcmalloc PATHS /usr/local/probe/lib )
list(APPEND LIBS ${TCMALLOC})


IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
   MESSAGE("")
   MESSAGE("cmake for GCC ")
   SET(PLATFORM_LINK_LIBRIES rt)
   SET(CMAKE_CXX_FLAGS "-Wall -rdynamic -Wunused -std=c++14 -pthread -D_GLIBCXX_USE_NANOSLEEP")
ENDIF()


# GENERIC STEPS
MESSAGE("PROJECT SRC: ${PROJECT_SRC}")
FILE(GLOB SRC_FILES ${PROJECT_SRC}/*.h ${PROJECT_SRC}/*.hpp ${PROJECT_SRC}/*.cpp) 
 

# Setup Library name
# Name for the library
SET(LIBRARY_TO_BUILD DeathKnell)



# Create the DeathKnell library
include_directories(${PROJECT_SRC})
add_library(${LIBRARY_TO_BUILD} SHARED  ${SRC_FILES})
SET(DeathKnell_VERSION_STRING ${VERSION})
SET_TARGET_PROPERTIES(${LIBRARY_TO_BUILD} PROPERTIES LINKER_LANGUAGE CXX SOVERSION ${VERSION})
TARGET_LINK_LIBRARIES(${LIBRARY_TO_BUILD} ${TCMALLOC})
TARGET_LINK_LIBRARIES(${LIBRARY_TO_BUILD} ${LIBS})



# create the unit tests
# =========================
set(GTEST_DIR ${DIR_3RDPARTY}/gtest-1.7.0)
set(GTEST_INCLUDE_DIRECTORIES ${GTEST_DIR}/include ${GTEST_DIR} ${GTEST_DIR}/src)
MESSAGE( "Attempt to build gtest. gtest directory: " ${GTEST_DIR})
include_directories(${GTEST_INCLUDE_DIRECTORIES})
add_library(gtest_170_lib ${GTEST_DIR}/src/gtest-all.cc)
set_target_properties(gtest_170_lib PROPERTIES COMPILE_DEFINITIONS "GTEST_HAS_RTTI=0")
enable_testing(true)

include_directories(test)
include_directories(${PROJECT_SRC})
file(GLOB TEST_SRC_FILES "test/*.cpp")

SET(CMAKE_CXX_FLAGS "-Wall -Wno-sign-compare -rdynamic -Wunused -std=c++14 -pthread -D_GLIBCXX_USE_NANOSLEEP")
add_executable(UnitTestRunner 3rdparty/test_main.cpp ${TEST_SRC_FILES} )
target_link_libraries(UnitTestRunner ${LIBRARY_TO_BUILD} gtest_170_lib ${LIBS})
set_target_properties(${test} PROPERTIES COMPILE_FLAGS "-isystem -pthread ")
