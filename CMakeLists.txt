cmake_minimum_required(VERSION 3.19)
project(SomeLib VERSION 1.0.0)

## C++ language configuration boilerplate

if (NOT DEFINED CMAKE_CXX_VISIBILITY_PRESET AND
    NOT DEFINED CMAKE_VISIBILITY_INLINES_HIDDEN)
  set(CMAKE_CXX_VISIBILITY_PRESET hidden)
  set(CMAKE_VISIBILITY_INLINES_HIDDEN YES)
endif ()

## Let SomeLib_SHARED_LIBS override BUILD_SHARED_LIBS

if (DEFINED SomeLib_SHARED_LIBS)
    set(BUILD_SHARED_LIBS "${SomeLib_SHARED_LIBS}")
endif ()

## Create the main SomeLib library target

add_library(SomeLib src/random.cpp)
add_library(SomeLib::SomeLib ALIAS SomeLib)
set_target_properties(SomeLib PROPERTIES
                      VERSION ${SomeLib_VERSION}
                      SOVERSION ${SomeLib_VERSION_MAJOR})
target_include_directories(
    SomeLib PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>")
target_compile_features(SomeLib PUBLIC cxx_std_17)

## Generate the export header for SomeLib and attach it to the target

include(GenerateExportHeader)
generate_export_header(SomeLib EXPORT_FILE_NAME include/somelib/export.h)
target_compile_definitions(
    SomeLib PUBLIC "$<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:SOMELIB_STATIC_DEFINE>")
target_include_directories(
    SomeLib PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>")

## Include the install rules if the user wanted them (included by default when top-level)

string(COMPARE EQUAL "${CMAKE_SOURCE_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}" is_top_level)
option(SomeLib_INCLUDE_PACKAGING "Include packaging rules for SomeLib" "${is_top_level}")
if (SomeLib_INCLUDE_PACKAGING)
    add_subdirectory(packaging)
endif ()
