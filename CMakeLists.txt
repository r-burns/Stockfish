cmake_minimum_required(VERSION 3.13) # needed for if(<var> IN_LIST

include(GNUInstallDirs)

project(stockfish LANGUAGES CXX)

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

include(CPack)
set(CPACK_STRIP_FILES ON CACHE BOOL "")

# Populate interface libraries for setting compiler/linker options

add_library(project_all INTERFACE)
add_library(project_defs INTERFACE)
add_library(project_libs INTERFACE)
add_library(project_options INTERFACE)
add_library(project_sanitize INTERFACE)
add_library(project_warnings INTERFACE)
target_link_libraries(project_all INTERFACE
    project_defs
    project_libs
    project_options
    project_sanitize
    project_warnings
    )

# Universal settings

target_compile_features(project_options INTERFACE cxx_std_11)
target_compile_options(project_options INTERFACE -fno-exceptions)
target_compile_options(project_warnings INTERFACE -Wall -Wcast-qual)

# Compiler-specific settings

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(project_warnings INTERFACE
        -pedantic -Wextra -Wshadow)
    if(APPLE)
        target_link_options(project_options INTERFACE --no-as-needed)
    endif()
elseif(MINGW)
    target_compile_options(project_warnings INTERFACE -Wextra -Wshadow)
    target_link_options(target_options INTERFACE -static)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(project_warnings INTERFACE
        -pedantic -Wextra -Wshadow)
    if(NOT APPLE AND NOT CMAKE_SYSTEM STREQUAL "OpenBSD")
        target_compile_options(project_libs INTERFACE atomic)
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    target_compile_options(project_warnings INTERFACE
        -diag-disable 1476,10120 -Wcheck -Wabi -Wdeprecated -strict-ansi)
endif()

# On mingw use Windows threads, otherwise POSIX
if(NOT MINGW)
    # Haiku has pthreads in its libroot, so only link it in on other platforms
    if(NOT CMAKE_SYSTEM_NAME MATCHES "Haiku")
        target_link_libraries(project_libs INTERFACE pthread)
    endif()
endif()

if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" OR
   CMAKE_SYSTEM_PROCESSOR MATCHES "ppc64")
    target_compile_definitions(project_defs INTERFACE IS_64BIT)
endif()

# User-configurable options

option(SANITIZE_THREAD "Enable threading error checks" OFF)
if(SANITIZE_THREAD)
    target_link_libraries(project_sanitize INTERFACE -fsanitize=thread)
endif()
option(SANITIZE_UNDEFINED "Enable undefined behavior checks" OFF)
if(SANITIZE_UNDEFINED)
    target_link_libraries(project_sanitize INTERFACE -fsanitize=undefined)
endif()

option(USE_PREFETCH "Use prefetch asm-instruction" ON)
if(NOT USE_PREFETCH)
    target_compile_definitions(project_defs INTERFACE NO_PREFETCH)
endif()

option(USE_POPCNT "Use popcnt asm-instruction" ON)
if(USE_POPCNT)
    target_compile_definitions(project_defs INTERFACE USE_POPCNT)
endif()

include(CMakeDependentOption)
cmake_dependent_option(USE_SSE "Use Intel Streaming SIMD Extensions" ON
    "USE_PREFETCH" OFF)
if(USE_SSE)
    target_compile_options(project_defs INTERFACE -msse)
endif()

if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    option(USE_PEXT "Use pext x86_64 asm-instructions" OFF)
    if(USE_PEXT)
        target_compile_definitions(project_defs INTERFACE USE_PEXT)
    endif()
endif()


add_subdirectory(src)
