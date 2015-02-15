CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
project(time2root)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)
find_package(ROOT REQUIRED)

execute_process(COMMAND ${ROOT_CONFIG_EXECUTABLE} --libdir
    OUTPUT_VARIABLE ROOT_LIB_DIR
    OUTPUT_STRIP_TRAILING_WHITESPACE)
# Add libraries to linker flags
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L${ROOT_LIB_DIR} -Wl,-O1 -Wl,--as-needed")

separate_arguments(ROOT_LIBRARIES)
set(science_libs)
foreach(flag ${ROOT_LIBRARIES})
    if(flag MATCHES "^-l(.*)")
        string(REPLACE "-l" "" _libname ${flag})
        #find_library(lib ${_libname} PATHS ${ROOT_LIB_DIR})
        list(APPEND science_libs ${_libname})
        #unset(lib CACHE)
    endif()
endforeach()
unset(_libname)

find_library(gsl NAMES gsl)
find_library(gslcblas NAMES gslcblas)
find_library(fftw NAMES fftw3)
set(science_libs ${science_libs} ${gsl} ${gslcblas} ${fftw})

message(STATUS "Libs ${science_libs}")

macro(add_executable _name)
    _add_executable(${ARGV})
    if (TARGET ${_name})
        target_link_libraries(${_name} ${science_libs})
    endif()
endmacro()

set(SOURCE
    FritzDPSS.cxx
    iqtdata.cxx
    multitaper.cxx
    header.cxx
    header_dict.cxx
    setinfo.c)
set_source_files_properties(setinfo.c PROPERTIES LANGUAGE CXX)
set(HEADERS
    header.h
    header_linkdef.h)

include_directories(${CMAKE_SOURCE_DIR} ${ROOT_INCLUDE_DIR})
root_generate_dictionary(header_dict header.h LINKDEF header_linkdef.h)

# Build object library with common objects
add_library(iqt_tiq OBJECT ${SOURCE})
# That then gets linked into all executables
add_executable(compare compare.cxx $<TARGET_OBJECTS:iqt_tiq>)
add_executable(read_iqt read_iqt.cxx $<TARGET_OBJECTS:iqt_tiq>)
add_executable(visualizer visualizer.C $<TARGET_OBJECTS:iqt_tiq>)
add_executable(time2root main.cxx $<TARGET_OBJECTS:iqt_tiq>)
