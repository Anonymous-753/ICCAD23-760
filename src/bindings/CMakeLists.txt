project(aiger LANGUAGES C CXX)
cmake_minimum_required(VERSION 3.4...3.18)

#set (CMAKE_CXX_STANDARD 17)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

if (CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR ) 
  message (FATAL_ERROR
    "In-source builds are not allowed. Please clean your source tree and try again.")  
endif()

add_library(aiger_library aiger/aiger.h aiger/aiger.c)
target_compile_options(aiger_library PRIVATE -fPIC)

FILE(GLOB CadicalSources cadical/src/*.cpp cadical/src/*.hpp)
add_library (cadical_library cadical.h ${CadicalSources})
target_compile_options(cadical_library PRIVATE -fPIC)

add_library(rsynth_library relationSynthesiser.h relationSynthesiser.cc)
target_link_libraries(rsynth_library PRIVATE cadical_library)
target_compile_options(rsynth_library PRIVATE -fPIC)


find_package(pybind11 CONFIG)
message(STATUS "Found pybind11 v${pybind11_VERSION}: ${pybind11_INCLUDE_DIRS}")
if (pybind11_FOUND) 
  pybind11_add_module(aiger SHARED aiger_bindings.cc)
  target_link_libraries(aiger PRIVATE aiger_library)
  target_include_directories(aiger PUBLIC "aiger")
  
  pybind11_add_module(cadical SHARED cadical_bindings.cc)
  target_link_libraries(cadical PRIVATE cadical_library)
  
  pybind11_add_module(relationSynthesiser SHARED relationSynthesiser_bindings.cc)
  target_link_libraries(relationSynthesiser PRIVATE rsynth_library)
  
endif()
