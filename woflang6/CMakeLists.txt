cmake_minimum_required(VERSION 3.16)
project(woflang VERSION 3.0.0 LANGUAGES CXX)

# Set C++23 standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Create plugins directory
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/bin/plugins)

# Platform detection
if(WIN32)
    add_definitions(-DWOFLANG_PLATFORM_WINDOWS)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
else()
    add_definitions(-DWOFLANG_PLATFORM_UNIX)
endif()

# Compiler settings
if(MSVC)
    add_compile_options(/W4 /permissive- /utf-8)
    add_compile_options(/std:c++latest)
else()
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# SIMD detection
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-mavx2" COMPILER_SUPPORTS_AVX2)
if(COMPILER_SUPPORTS_AVX2)
    add_compile_options(-mavx2)
    add_definitions(-DWOFLANG_USE_AVX2)
endif()

# Core library
add_library(woflang_core STATIC
    src/core/woflang.cpp
    src/io/tokenizer.cpp
)

target_include_directories(woflang_core PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/src/core
    ${CMAKE_CURRENT_SOURCE_DIR}/src/io
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# Link system libraries
if(NOT WIN32)
    target_link_libraries(woflang_core PUBLIC ${CMAKE_DL_LIBS})
endif()

# Main executable
add_executable(woflang main.cpp)
target_link_libraries(woflang PRIVATE woflang_core)

# Simple plugin building - just the ones that exist
function(add_woflang_plugin name source)
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/plugins/${source}")
        add_library(${name} SHARED "plugins/${source}")
        target_include_directories(${name} PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/src/core
            ${CMAKE_CURRENT_SOURCE_DIR}/src/io
            ${CMAKE_CURRENT_SOURCE_DIR}/src
        )
        target_link_libraries(${name} PRIVATE woflang_core)
        set_target_properties(${name} PROPERTIES
            LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/plugins"
            RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/plugins"
        )
        if(WIN32)
            set_target_properties(${name} PROPERTIES SUFFIX ".dll")
        endif()
    endif()
endfunction()

# Only build plugins that actually exist
add_woflang_plugin(math_ops math_ops.cpp)
add_woflang_plugin(logic_ops logic_ops.cpp)
add_woflang_plugin(crypto_ops crypto_ops.cpp)
add_woflang_plugin(fractal_ops fractal_ops.cpp)
add_woflang_plugin(quantum_ops quantum_ops.cpp)
add_woflang_plugin(chemistry_ops chemistry_ops.cpp)
add_woflang_plugin(kanji_ops kanji_ops.cpp)

# Summary
message(STATUS "")
message(STATUS "Woflang Configuration Summary:")
message(STATUS "  Version:       ${PROJECT_VERSION}")
message(STATUS "  C++ Standard:  ${CMAKE_CXX_STANDARD}")
message(STATUS "  Build Type:    ${CMAKE_BUILD_TYPE}")
message(STATUS "  Platform:      ${CMAKE_SYSTEM_NAME}")
if(COMPILER_SUPPORTS_AVX2)
    message(STATUS "  SIMD Support:  AVX2 enabled")
else()
    message(STATUS "  SIMD Support:  Disabled")
endif()
message(STATUS "")