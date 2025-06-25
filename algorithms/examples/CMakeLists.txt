# Collect all example files in the examples directory
set(EXAMPLE_FILES # Example source files to be added here
)

# Check if there are any example source files before creating the executables
if(EXAMPLE_FILES)
  foreach(EXAMPLE ${EXAMPLE_FILES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE})
    # Link the example executable with the algorithms library
    target_link_libraries(${EXAMPLE_NAME} PRIVATE optic_algorithms)
  endforeach()
else()
  message(STATUS "No example files found in examples directory.")
endif()
