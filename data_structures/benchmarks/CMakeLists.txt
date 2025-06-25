# Collect all benchmark source files
file(GLOB BENCHMARK_SOURCES "*.cpp")

# Only create the target if there are source files
if(BENCHMARK_SOURCES)
    add_executable(optic_data_structures_benchmarks ${BENCHMARK_SOURCES})
    # Include headers for header-only implementations
    target_include_directories(optic_data_structures_benchmarks PRIVATE 
        ${CMAKE_CURRENT_SOURCE_DIR}/../include
    )
    target_compile_features(optic_data_structures_benchmarks PRIVATE cxx_std_23)
    
    target_link_libraries(optic_data_structures_benchmarks PRIVATE
        benchmark::benchmark
    )
    
    # Link with library if it exists
    if(TARGET optic_data_structures)
        target_link_libraries(optic_data_structures_benchmarks PRIVATE optic_data_structures)
    endif()
else()
    message(STATUS "No benchmark sources found for optic_data_structures_benchmarks")
endif()
