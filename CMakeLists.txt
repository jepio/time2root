
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

project(time2root)

# Get root include location
execute_process(COMMAND root-config --incdir OUTPUT_VARIABLE ROOT_DIR
    OUTPUT_STRIP_TRAILING_WHITESPACE)
# Get libraries that need to be linked
execute_process(COMMAND root-config --glibs OUTPUT_VARIABLE GLIBS
    OUTPUT_STRIP_TRAILING_WHITESPACE)
include_directories(${CMAKE_SOURCE_DIR} ${ROOT_DIR})
# Add libraries to linker flags
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${GLIBS} -lgsl -lgslcblas -lfftw3 -lm")

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
# Build dictionary
add_custom_command(OUTPUT header_dict.cxx
    COMMAND rootcint -f ${CMAKE_CURRENT_BINARY_DIR}/header_dict.cxx -c ${HEADERS}
    DEPENDS header.h header_linkdef.h
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

# Build object library with common objects
add_library(iqt_tiq OBJECT ${SOURCE})
# That then gets linked into all executables
add_executable(compare compare.cxx $<TARGET_OBJECTS:iqt_tiq>)
add_executable(read_iqt read_iqt.cxx $<TARGET_OBJECTS:iqt_tiq>)
add_executable(visualizer visualizer.C $<TARGET_OBJECTS:iqt_tiq>)
add_executable(time2root main.cxx $<TARGET_OBJECTS:iqt_tiq>)
