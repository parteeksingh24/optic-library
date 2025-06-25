# Collect all benchmark source files
file(GLOB BENCHMARK_SOURCES "*.cpp")

# Only create the target if there are source files
if(BENCHMARK_SOURCES)
  add_executable(optic_algorithms_benchmarks ${BENCHMARK_SOURCES})
  target_link_libraries(optic_algorithms_benchmarks
                        PRIVATE optic_algorithms benchmark::benchmark)
else()
  message(STATUS "No benchmark sources found for optic_algorithms_benchmarks")
endif()
