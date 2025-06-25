# Collect all example files in the examples directory
file(GLOB EXAMPLE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
     "${CMAKE_CURRENT_SOURCE_DIR}/*.cxx")

# Check if there are any example source files before creating the executables
if(EXAMPLE_FILES)
  foreach(EXAMPLE ${EXAMPLE_FILES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE})
    # Link the example executable with the data structures library (if
    # available) For header-only implementations, just include the headers
    target_include_directories(${EXAMPLE_NAME}
                               PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/../include)
    target_compile_features(${EXAMPLE_NAME} PRIVATE cxx_std_23)
    if(TARGET optic_data_structures)
      target_link_libraries(${EXAMPLE_NAME} PRIVATE optic_data_structures)
    endif()
  endforeach()
else()
  message(STATUS "No example files found in examples directory.")
endif()
