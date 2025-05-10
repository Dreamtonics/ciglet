cmake_minimum_required(VERSION 3.21)
project(ciglet)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(FATAL_ERROR
    "In-source builds are not allowed."
    "\n Please remove \"${CMAKE_SOURCE_DIR}/CMakeCache.txt\" and try again from another folder"
  )
endif()

set(CIGLET_FP_TYPE "float" CACHE STRING "Floating point type")
set_property(CACHE CIGLET_FP_TYPE PROPERTY STRINGS "float" "double")

option(CIGLET_BUILD_TESTS "Build tests" OFF)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# The static library: libciglet.a
add_library(ciglet STATIC
  ciglet.c
  external/fftsg_h.c
  external/wavfile.c
  external/fast_median.c
)
if (NOT MSVC)
  target_compile_options(ciglet PRIVATE -ffast-math)
else()
  target_compile_options(ciglet PRIVATE /fp:fast)
endif()
target_compile_definitions(ciglet PUBLIC FP_TYPE=${CIGLET_FP_TYPE})
find_library(M_LIBRARY m)
if(M_LIBRARY)
  target_link_libraries(ciglet PRIVATE ${M_LIBRARY})
endif()
include_directories(ciglet PRIVATE external)

install(TARGETS ciglet
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
install(FILES ciglet.h DESTINATION include/ciglet)

# Test
if (CIGLET_BUILD_TESTS)
  message(STATUS "Tests are enabled")
  enable_testing()

  add_executable(ciglet-test test/test.c)
  target_link_libraries(ciglet-test PRIVATE ciglet)
  add_test(NAME ciglet-test COMMAND ciglet-test)
endif()
