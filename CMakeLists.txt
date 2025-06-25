cmake_minimum_required(VERSION 3.21...3.27)

# Define the project version in a separate file
include(${CMAKE_CURRENT_SOURCE_DIR}/version.cmake)

# Define the project name and version
project(
  OpticLibrary
  VERSION ${PROJECT_VERSION}
  LANGUAGES CXX)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Main project detection - only set these when this is the main project
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
  # Enable compile commands export for better IDE/tooling support
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

  # Set default build type if not specified
  if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE
        "Release"
        CACHE STRING "Choose the type of build" FORCE)
  endif()
  # Add Coverage build type
  set(CMAKE_CXX_FLAGS_COVERAGE
      "-g -O0 --coverage -fprofile-arcs -ftest-coverage"
      CACHE STRING "Flags used by the C++ compiler during coverage builds."
            FORCE)
  set(CMAKE_C_FLAGS_COVERAGE
      "-g -O0 --coverage -fprofile-arcs -ftest-coverage"
      CACHE STRING "Flags used by the C compiler during coverage builds." FORCE)
  set(CMAKE_EXE_LINKER_FLAGS_COVERAGE
      "--coverage"
      CACHE STRING "Flags used for linking binaries during coverage builds."
            FORCE)
  set(CMAKE_SHARED_LINKER_FLAGS_COVERAGE
      "--coverage"
      CACHE STRING
            "Flags used by the shared libraries linker during coverage builds."
            FORCE)
  mark_as_advanced(
    CMAKE_CXX_FLAGS_COVERAGE CMAKE_C_FLAGS_COVERAGE
    CMAKE_EXE_LINKER_FLAGS_COVERAGE CMAKE_SHARED_LINKER_FLAGS_COVERAGE)

  # Options for building tests and benchmarks
  option(BUILD_TESTS "Build the tests" ON)
  option(BUILD_BENCHMARKS "Build the benchmarks" ON)
  # Enable testing
  if(BUILD_TESTS)
    enable_testing()
  endif()

  # Add compile options for different compilers
  if(MSVC)
    add_compile_options(/W4 /WX /EHsc)
  else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
  endif()

  # Sanitizer options
  option(ENABLE_SANITIZER_ADDRESS "Enable address sanitizer" OFF)
  option(ENABLE_SANITIZER_UNDEFINED_BEHAVIOR
         "Enable undefined behavior sanitizer" OFF)
endif()

# Option for building shared or static libraries
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

# Set position-independent code flags for static libraries
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Define minimum versions for dependencies
set(GOOGLETEST_MIN_VERSION "1.14.0")
set(BENCHMARK_MIN_VERSION "1.8.3")

# Find dependencies
find_package(GTest ${GOOGLETEST_MIN_VERSION})
if(NOT GTest_FOUND)
  message(STATUS "GTest not found. Fetching from GitHub...")
  include(FetchContent)
  FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.15.2)
  FetchContent_MakeAvailable(googletest)
endif()

if(BUILD_BENCHMARKS)
  find_package(benchmark ${BENCHMARK_MIN_VERSION})
  if(NOT benchmark_FOUND)
    message(STATUS "Google Benchmark not found. Fetching from GitHub...")
    include(FetchContent)
    FetchContent_Declare(
      benchmark
      GIT_REPOSITORY https://github.com/google/benchmark.git
      GIT_TAG v1.8.5)
    FetchContent_MakeAvailable(benchmark)
  endif()
endif()

# Add subdirectories
add_subdirectory(data_structures)
add_subdirectory(algorithms)

# Main project specific configurations
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
  # Configure sanitizers
  if(ENABLE_SANITIZER_ADDRESS OR ENABLE_SANITIZER_UNDEFINED_BEHAVIOR)
    add_library(sanitizer_options INTERFACE)
    if(ENABLE_SANITIZER_ADDRESS)
      target_compile_options(sanitizer_options
                             INTERFACE $<$<CONFIG:Debug>:-fsanitize=address>)
      target_link_options(sanitizer_options INTERFACE
                          $<$<CONFIG:Debug>:-fsanitize=address>)
    endif()
    if(ENABLE_SANITIZER_UNDEFINED_BEHAVIOR)
      target_compile_options(sanitizer_options
                             INTERFACE $<$<CONFIG:Debug>:-fsanitize=undefined>)
      target_link_options(sanitizer_options INTERFACE
                          $<$<CONFIG:Debug>:-fsanitize=undefined>)
    endif()
  endif()

  # Add Doxygen for API documentation
  find_package(Doxygen)
  if(DOXYGEN_FOUND)
    set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
    set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)
    configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)

    # Ensure the output directory exists
    file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/docs/api)

    add_custom_target(
      doc_doxygen ALL
      COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
      WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
      COMMENT "Generating API documentation with Doxygen"
      VERBATIM)
  else()
    message(
      STATUS "Doxygen not found. API documentation will not be generated.")
  endif()
  # Add coverage target
  if(CMAKE_BUILD_TYPE STREQUAL "Coverage")
    find_program(GCOV_PATH gcov)
    find_program(LCOV_PATH lcov)
    find_program(GENHTML_PATH genhtml)
    if(GCOV_PATH
       AND LCOV_PATH
       AND GENHTML_PATH)
      add_custom_target(
        coverage
        # Cleanup previous coverage data
        COMMAND ${LCOV_PATH} --directory . --zerocounters
        # Run tests to generate coverage data
        COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure
        # Capture coverage data
        COMMAND ${LCOV_PATH} --directory . --capture --output-file coverage.info
        # Remove external libraries and test files from coverage
        COMMAND
          ${LCOV_PATH} --remove coverage.info '/usr/*' '*/build/*' '*/tests/*'
          '*/benchmarks/*' '*/examples/*' '*/_deps/*' --output-file
          coverage.info.cleaned
        # Generate HTML coverage report
        COMMAND ${GENHTML_PATH} -o coverage_html coverage.info.cleaned
        # Output summary
        COMMAND ${LCOV_PATH} --list coverage.info.cleaned
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        COMMENT "Generating coverage report"
        VERBATIM)
      message(
        STATUS
          "Coverage target available. Use 'make coverage' to generate reports.")
    else()
      message(STATUS "Coverage tools not found. Coverage target not available.")
    endif()
  endif()
endif()

# Installation
include(CMakePackageConfigHelpers)
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/OpticLibraryConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/OpticLibraryConfig.cmake"
  INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/OpticLibrary")
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/OpticLibraryConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/OpticLibraryConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/OpticLibraryConfigVersion.cmake"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/OpticLibrary")
