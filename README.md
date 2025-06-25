# Optic Library: A modern C++ toolkit

<!-- Badges -->
[![Optic Library CI/CD](https://github.com/parteeksingh24/optic-library/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/parteeksingh24/optic-library/actions/workflows/ci-cd.yml)
[![codecov](https://codecov.io/gh/parteeksingh24/optic-library/branch/main/graph/badge.svg)](https://codecov.io/gh/parteeksingh24/optic-library)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## Overview
The Optic Library is a comprehensive collection of modern Data Structures and Algorithms (DSA), implemented in C++. Primarily designed for educational use, it explores efficient and robust implementations that go beyond the typical curriculum for Computer Science students. Whenever possible, this project aims to enhance traditional DSA components, focusing on improved performance and usability.

## Project Structure
- **data_structures/**: Contains various custom data structures.
- **algorithms/**: Includes a wide range of algorithms, from basic to advanced.

Both the `data_structures` and `algorithms` modules contain the same overall components:
- *benchmarks/*: For performance benchmarking (using Google Benchmark), as well as results and graphs
- *include/optic/*: Module header files
- *src/*: Implementation files
- *tests/*: Unit tests (using Google Test)
- *examples/*: Example usage of module implementations

## Getting Started

### Prerequisites
- **CMake**: This project uses CMake as the build system. Ensure that you have CMake (version 3.21 or higher) installed on your system.
- **Compiler**: A C++ compiler supporting C++23 (such as `g++`, `clang`, or MSVC).
- **Ninja** (optional but recommended): For optimized build performance.
- **Dependencies**: The project uses GoogleTest and Google Benchmark for unit tests and benchmarking. CMake will handle fetching and configuring these libraries automatically.

### Building the Project

1. Clone the repository:
    ```bash
    git clone https://github.com/parteeksingh24/optic-library.git
    cd optic-library
    ```

2. Create a build directory and configure the project using CMake:
    ```bash
    mkdir build && cd build
    cmake ..
    ```
    
    For optimal build performance, use Ninja (if installed):
    ```bash
    cmake -B build -G Ninja
    ```

3. Build the project:
    ```bash
    cmake --build build
    ```

4. Run the tests:
    ```bash
    cd build
    ctest --output-on-failure
    ```

### Usage
Once the project is built, you can run the example programs in the `examples/` directory or use the libraries in your own projects.

```bash
./examples/example_program
```

## Contribution
Contributions are welcome! Please read through the contribution guidelines in `CONTRIBUTING.md` before making a pull request.

## License
This project is licensed under the MIT License - see the `LICENSE` file for details.
