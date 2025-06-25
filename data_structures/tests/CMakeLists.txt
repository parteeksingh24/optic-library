# Collect all test source files
file(GLOB TEST_SOURCES "*.cpp")

# Only create the target if there are source files
if(TEST_SOURCES)
  add_executable(optic_data_structures_tests ${TEST_SOURCES})
  # Include headers for header-only implementations
  target_include_directories(optic_data_structures_tests
                             PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/../include)
  target_compile_features(optic_data_structures_tests PRIVATE cxx_std_23)
  target_link_libraries(optic_data_structures_tests PRIVATE GTest::gtest
                                                            GTest::gtest_main)
  # Link with library if it exists
  if(TARGET optic_data_structures)
    target_link_libraries(optic_data_structures_tests
                          PRIVATE optic_data_structures)
  endif()
  gtest_discover_tests(optic_data_structures_tests)
else()
  message(STATUS "No test sources found for optic_data_structures_tests")
endif()
