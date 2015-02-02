CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
project(time2root)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)
find_package(ROOT REQUIRED)

# Add libraries to linker flags
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${ROOT_LIBRARIES} -lgsl -lgslcblas -lfftw3 -lm")

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
