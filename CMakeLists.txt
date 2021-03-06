cmake_minimum_required(VERSION 3.2.3)

foreach(policy
    CMP0074 # CMake 3.12
    )
  if(POLICY ${policy})
    cmake_policy(SET ${policy} NEW)
  endif()
endforeach()

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)

set(PROJECT_NAME dumpwow)

project(${PROJECT_NAME})

if ("x_${HADESMEM_ROOT}" STREQUAL "x_")
    message(FATAL_ERROR "HADESMEM_ROOT not set.  ${PROJECT_NAME} requires hadesmem, available at https://bitbucket.org/raptorfactor/hadesmem")
endif()

# build in Release-mode by default if not explicitly set
if( NOT CMAKE_BUILD_TYPE )
  set(CMAKE_BUILD_TYPE "RelWithDebInfo")
endif()

# threading library is required
find_package(Threads REQUIRED)

# currently there is a bug in the cmake included with visual studio where the wrong
# version number is resolved for the compiler.  this should work around it.
# FIXME: this should be removable once visual studio begins using cmake 3.8
if (MSVC)
	if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 19.10)
        set(Boost_COMPILER -vc141)
    endif()
endif()

set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)

find_package(boost)

if (NOT EXISTS "${HADESMEM_ROOT}/include/memory/hadesmem")
    message(FATAL_ERROR "hadesmem not found at ${HADESMEM_ROOT}")
else()
    message(STATUS "hadesmem found at ${HADESMEM_ROOT}")
endif()

if ("x_${CMAKE_BUILD_TYPE}" STREQUAL "x_Debug")
    set(HADESMEM_BUILD "Debug")
else()
    set(HADESMEM_BUILD "Release")
endif()

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(HADESMEM_ARCH "x64")
else()
    set(HADESMEM_ARCH "Win32")
endif()

set(HADESMEM_LIB_DIR "${HADESMEM_ROOT}\\build\\vs\\${HADESMEM_BUILD}\\${HADESMEM_ARCH}")
message(STATUS "hadesmem library directory: ${HADESMEM_LIB_DIR}")

add_definitions(-DUNICODE -D_UNICODE -D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS -DASMJIT_STATIC -DASMJIT_BUILD_X86 -DPUGIXML_WCHAR_MODE)
include_directories(Include
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${Boost_INCLUDE_DIR}"
    "${HADESMEM_ROOT}/include/memory/"
    "${HADESMEM_ROOT}/deps/udis86/udis86"
    "${HADESMEM_ROOT}/deps/asmjit/asmjit/src"
    "${HADESMEM_ROOT}/deps/pugixml/pugixml/src"
)

link_directories("${HADESMEM_LIB_DIR}")

add_subdirectory(dll)
add_subdirectory(src)

install(FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE"
    "${CMAKE_CURRENT_SOURCE_DIR}/README.md"
    DESTINATION "${CMAKE_INSTALL_PREFIX}")