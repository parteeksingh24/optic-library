# Collect all source files in the src directory
file(GLOB_RECURSE SRC_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx"
)

if(SRC_FILES)
    # Add the data structures library
    add_library(optic_data_structures STATIC ${SRC_FILES})

    # Specify the include directories for header files
    target_include_directories(optic_data_structures
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<INSTALL_INTERFACE:include>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/src
    )

    # Specify C++23 feature requirements
    target_compile_features(optic_data_structures PUBLIC cxx_std_23)

    # Specify installation targets
    install(TARGETS optic_data_structures
        EXPORT OpticDataStructuresTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    # Install the headers
    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ 
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/optic/data_structures)

else()
    message(STATUS "No source files found in src directory.")
endif()

# Add subdirectories for examples, tests, and benchmarks
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/examples")
    add_subdirectory(examples)
else()
    message(STATUS "No examples directory found.")
endif()

if(BUILD_TESTS AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests")
    add_subdirectory(tests)
endif()

if(BUILD_BENCHMARKS AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/benchmarks")
    add_subdirectory(benchmarks)
endif()
