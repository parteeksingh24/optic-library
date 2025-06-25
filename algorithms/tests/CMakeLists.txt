# Collect all test source files
file(GLOB TEST_SOURCES "*.cpp")

# Only create the target if there are source files
if(TEST_SOURCES)
  add_executable(optic_algorithms_tests ${TEST_SOURCES})
  target_link_libraries(optic_algorithms_tests PRIVATE optic_algorithms gtest
                                                       gtest_main)
  gtest_discover_tests(optic_algorithms_tests)
else()
  message(STATUS "No test sources found for optic_algorithms_tests")
endif()
