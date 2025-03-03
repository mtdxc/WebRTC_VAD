cmake_minimum_required(VERSION 2.4)
project(rtc_apm)
if (MSVC)
    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ")
    else ()
        set(CMAKE_BUILD_TYPE "Release")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  /fp:fast /Gy /Oi /Oy /O2 /Ot /Zi /EHsc ")
    endif ()
    ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
else ()
    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g -Wall -Wno-unused-variable")
    else (CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_BUILD_TYPE "Release")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2")
    endif (CMAKE_BUILD_TYPE STREQUAL "Debug")
endif ()

set(CMAKE_CXX_STANDARD 14)

include_directories(include .)
add_library(test_common STATIC test_common.cc dr_wav.h timing.h)

add_library(signal_processing STATIC signal_processing_library.c)

aux_source_directory(vad VAD_SRC)
add_library(vad STATIC ${VAD_SRC})
target_link_libraries(vad signal_processing)

add_executable(vad_main vad_main.cc)
target_link_libraries(vad_main vad test_common)

aux_source_directory(aecm AECM_SRC)
list(FILTER AECM_SRC EXCLUDE REGEX ".*aecm_core_neon.cc$")
list(FILTER AECM_SRC EXCLUDE REGEX ".*aecm_core_mips.cc$")
add_library(aecm STATIC ${AECM_SRC})
target_link_libraries(aecm signal_processing)

add_executable(aecm_main aecm_main.cc)
target_link_libraries(aecm_main aecm test_common)

add_executable(ns_main ns_main.c src/noise_suppression.c include/noise_suppression.h)
target_link_libraries(ns_main test_common)

add_executable(cng_main cng_main.cpp src/cng.cpp include/cng.h)
target_link_libraries(cng_main test_common)

add_executable(agc_main agc_main.c src/agc.c include/agc.h)
target_link_libraries(agc_main test_common)

aux_source_directory(ns NS_SRC)
add_library(ns STATIC ${NS_SRC})
add_executable(nsp_main nsp_main.cc)
target_link_libraries(nsp_main test_common ns)