# Development Guide

This guide provides comprehensive technical documentation for developers working on the Optic Library. For contribution guidelines, project vision, and getting started information, see [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Table of Contents

- [Development Environment Setup](#development-environment-setup)
- [Build System](#build-system)
- [Development Tools](#development-tools)
- [Testing and Benchmarking](#testing-and-benchmarking)
- [Documentation](#documentation)
- [Code Quality Standards](#code-quality-standards)
- [Project Architecture](#project-architecture)
- [Development Workflow](#development-workflow)

## Development Environment Setup

### Prerequisites

Ensure you have the following tools installed:

```bash
# macOS (using Homebrew)
brew install cmake ninja doxygen graphviz lcov python

# Ubuntu/Debian
sudo apt-get install cmake ninja-build doxygen graphviz lcov python3-pip

# Windows (using Chocolatey)
choco install cmake ninja doxygen.install python
```

### Compiler Requirements

- **C++23 support required**
- Tested compilers:
  - GCC 12+ (`g++`)
  - Clang 15+ (`clang++`)
  - MSVC 2022+ (`cl`)

### Python Tools

Install development and pre-commit tools:

```bash
pip install pre-commit cmake-format markdownlint-cli
```

### Verification

Verify your environment:

```bash
cmake --version          # Should be 3.21+
ninja --version          # For optimized builds
clang++ --version        # Should support -std=c++2b
doxygen --version        # Should be 1.9+
pre-commit --version     # For automated quality checks
```

## Build System

### CMake Configuration Options

The project uses modern CMake (3.21+) with several configuration options:

```bash
# Basic configuration
cmake -B build

# Recommended optimized build
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release

# Development build with all features
cmake -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Debug \
  -DBUILD_TESTS=ON \
  -DBUILD_BENCHMARKS=ON \
  -DENABLE_SANITIZER_ADDRESS=ON
```

### Build Types

| Build Type | Purpose | Optimizations | Debug Info |
|------------|---------|---------------|------------|
| `Debug` | Development, debugging | None (`-O0`) | Full (`-g`) |
| `Release` | Production, benchmarks | Full (`-O3`) | Minimal |
| `RelWithDebInfo` | Profiling, analysis | Moderate (`-O2`) | Full (`-g`) |
| `Coverage` | Code coverage analysis | None (`-O0`) | Full + coverage |

### Build Commands

```bash
# Configure
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug

# Build
cmake --build build

# Build specific target
cmake --build build --target optic_data_structures

# Build with verbose output
cmake --build build --verbose

# Clean build
cmake --build build --target clean

# Parallel build (automatic with Ninja)
cmake --build build --parallel 8
```

### Advanced Configuration

```bash
# Enable sanitizers
cmake -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Debug \
  -DENABLE_SANITIZER_ADDRESS=ON \
  -DENABLE_SANITIZER_UNDEFINED_BEHAVIOR=ON

# Coverage build
cmake -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Coverage \
  -DBUILD_TESTS=ON

# Cross-compilation example
cmake -B build -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=path/to/toolchain.cmake
```

### Important CMake Notes

**Adding New Source Files**: When adding new source files (`.cpp`, `.hpp`, `.h`), you must re-run the CMake configuration step because the project uses `file(GLOB)` for source discovery. CMake doesn't automatically detect new files added after the initial configuration.

```bash
# After adding new source files, reconfigure:
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug

# Then build as usual:
cmake --build build
```

This is a trade-off for build system simplicity - while `file(GLOB)` makes the CMake files cleaner, it requires manual reconfiguration when the file structure changes.

## Development Tools

### clang-format

Automatic code formatting following project standards:

```bash
# Format single file
clang-format -i src/data_structures/hat_trie.cpp

# Format all source files
find . -name '*.cpp' -o -name '*.hpp' | xargs clang-format -i

# Check formatting (CI mode)
find . -name '*.cpp' -o -name '*.hpp' | xargs clang-format --dry-run --Werror
```

Configuration in `.clang-format`:

- C++23 standard
- 4 spaces indentation
- 100 character line length
- Comprehensive style rules

### clang-tidy

Static analysis and modernization checks:

```bash
# Analyze single file
clang-tidy src/data_structures/hat_trie.cpp -- -std=c++23

# Analyze all files
find . -name '*.cpp' | xargs -I {} clang-tidy {} -- -std=c++23

# Fix issues automatically (use with caution)
clang-tidy -fix src/data_structures/hat_trie.cpp -- -std=c++23
```

Configuration in `.clang-tidy`:

- Modern C++ checks
- Performance analysis
- Readability improvements
- Bug detection

### Pre-commit Hooks

Automated quality checks before commits:

```bash
# Install hooks
pre-commit install

# Run on all files
pre-commit run --all-files

# Update hook versions
pre-commit autoupdate

# Skip hooks for emergency commits
git commit --no-verify -m "Emergency fix"
```

Available hooks:

- clang-format (formatting)
- clang-tidy (static analysis)
- cmake-format (CMake files)
- markdownlint (documentation)
- detect-secrets (security)

## Testing and Benchmarking

### Running Tests

```bash
# Build tests
cmake -B build -G Ninja -DBUILD_TESTS=ON
cmake --build build

# Run all tests
cd build && ctest --output-on-failure

# Run tests with verbose output
cd build && ctest -V

# Run specific test suite
cd build && ctest -R "data_structures"

# Run tests in parallel
cd build && ctest -j8

# Run tests with timeout
cd build && ctest --timeout 30
```

### Test Organization

```text
tests/
├── unit/           # Unit tests for individual components
├── integration/    # Integration tests
├── performance/    # Performance regression tests
└── fixtures/       # Test data and utilities
```

### Writing Tests

Example test structure:

```cpp
#include <gtest/gtest.h>
#include "optic/data_structures/hat_trie.hpp"

namespace optic::data_structures::test {

class HatTrieTest : public ::testing::Test {
protected:
    void SetUp() override {
        // Setup code
    }
    
    void TearDown() override {
        // Cleanup code
    }
    
    HatTrie<std::string, int> trie_;
};

TEST_F(HatTrieTest, InsertAndFind) {
    trie_.insert("hello", 42);
    EXPECT_EQ(trie_.find("hello")->second, 42);
}

TEST_F(HatTrieTest, PerformanceRequirement) {
    // Performance requirements as tests
    auto start = std::chrono::high_resolution_clock::now();
    
    for (int i = 0; i < 10000; ++i) {
        trie_.insert(std::to_string(i), i);
    }
    
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    
    EXPECT_LT(duration.count(), 100) << "Insert performance regression";
}

} // namespace optic::data_structures::test
```

### Benchmarking

```bash
# Build benchmarks
cmake -B build -G Ninja -DBUILD_BENCHMARKS=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Run benchmarks
./build/data_structures/benchmarks/optic_data_structures_benchmarks

# Run specific benchmark
./build/data_structures/benchmarks/optic_data_structures_benchmarks --benchmark_filter=HatTrie

# Output JSON results
./build/data_structures/benchmarks/optic_data_structures_benchmarks --benchmark_format=json > results.json

# Compare benchmark results
./tools/compare_benchmarks.py baseline.json current.json
```

Example benchmark:

```cpp
#include <benchmark/benchmark.h>
#include "optic/data_structures/hat_trie.hpp"

static void BM_HatTrieInsert(benchmark::State& state) {
    optic::data_structures::HatTrie<std::string, int> trie;
    
    for (auto _ : state) {
        for (int i = 0; i < state.range(0); ++i) {
            trie.insert(std::to_string(i), i);
        }
    }
    
    state.SetComplexityN(state.range(0));
}

BENCHMARK(BM_HatTrieInsert)
    ->Range(8, 8<<10)
    ->Complexity(benchmark::oN);

BENCHMARK_MAIN();
```

## Documentation

### Doxygen Generation

```bash
# Generate documentation
cmake --build build --target doc_doxygen

# View documentation
open build/docs/api/html/index.html
```

### Documentation Standards

Use comprehensive Doxygen documentation:

```cpp
/**
 * @brief Memory-efficient trie implementation with hybrid storage.
 * 
 * The HAT-trie combines the benefits of hash tables and tries,
 * providing excellent performance for string operations while
 * maintaining memory efficiency.
 * 
 * @tparam Key Key type (typically std::string)
 * @tparam Value Value type
 * @tparam Hash Hash function type
 * 
 * @complexity 
 * - Insert: O(k) where k is key length
 * - Search: O(k) where k is key length
 * - Memory: ~60% less than standard trie
 * 
 * @threadsafe No. Use external synchronization for concurrent access.
 * 
 * @example
 * ```cpp
 * HatTrie<std::string, int> trie;
 * trie.insert("hello", 42);
 * auto it = trie.find("hello");
 * if (it != trie.end()) {
 *     std::cout << "Found: " << it->second << std::endl;
 * }
 * ```
 */
template<typename Key, typename Value, typename Hash = std::hash<Key>>
class HatTrie {
public:
    /**
     * @brief Insert a key-value pair into the trie.
     * @param key The key to insert
     * @param value The value to associate with the key
     * @return Iterator to inserted element and success flag
     */
    std::pair<iterator, bool> insert(const Key& key, const Value& value);
};
```

### Custom Doxygen Aliases

Available custom aliases:

- `@complexity`: Algorithmic complexity analysis
- `@threadsafe`: Thread safety information
- `@lockfree`: Lock-free implementation notes
- `@benchmark`: Performance benchmark results
- `@educational`: Educational notes and explanations

## Code Quality Standards

### C++23 Features

Encouraged modern C++ features:

```cpp
// Concepts for better APIs
template<std::integral T>
void process_integer(T value) { /* ... */ }

// Ranges for cleaner algorithms
auto result = data 
    | std::views::filter([](auto& x) { return x > 0; })
    | std::views::transform([](auto& x) { return x * 2; });

// std::expected for error handling
std::expected<Value, Error> lookup(const Key& key) noexcept;

// Designated initializers
Config config {
    .max_size = 1000,
    .hash_function = std::hash<std::string>{}
};
```

### Performance Guidelines

1. **Zero-cost abstractions**: Prefer compile-time computation
2. **Move semantics**: Use move constructors and assignment
3. **RAII**: Automatic resource management
4. **Cache-friendly**: Consider data locality
5. **Avoid allocations**: Minimize dynamic memory allocation in hot paths

### Code Style

- **Naming**: `snake_case` for variables/functions, `PascalCase` for types
- **Constants**: `kConstantName` for compile-time constants
- **Members**: Trailing underscore for private members (`member_`)
- **Namespaces**: All code in `optic::` namespace hierarchy

### Error Handling

Prefer modern error handling approaches:

```cpp
// Use std::expected for recoverable errors
std::expected<Value, ErrorCode> safe_operation() noexcept;

// Use exceptions for programming errors
void validate_input(const Input& input) {
    if (!input.is_valid()) {
        throw std::invalid_argument("Invalid input format");
    }
}

// Use assertions for debug checks
assert(index < container.size() && "Index out of bounds");
```

## Project Architecture

### Module Structure

```text
optic-library/
├── data_structures/          # Data structure implementations
│   ├── include/optic/data_structures/
│   │   ├── hat_trie.hpp     # Public API headers
│   │   └── detail/          # Implementation details
│   ├── src/                 # Implementation files
│   ├── tests/               # Unit tests
│   ├── benchmarks/          # Performance tests
│   └── examples/            # Usage examples
├── algorithms/              # Algorithm implementations
│   └── [similar structure]
└── docs/                    # Documentation
    ├── design/              # Design documents
    └── user_guide/          # User guides
```

### Design Principles

1. **Header-only when possible**: Simplify integration
2. **Minimal dependencies**: Reduce external requirements
3. **Generic programming**: Template-based designs
4. **RAII compliance**: Automatic resource management
5. **Exception safety**: Strong exception guarantees

### API Design Guidelines

- **Consistent interfaces**: Follow STL conventions
- **Generic algorithms**: Work with iterators and ranges
- **Customization points**: Support custom allocators and comparators
- **Type safety**: Prefer compile-time checks over runtime

## Development Workflow

### Git Workflow

Follow the contribution process outlined in [`CONTRIBUTING.md`](CONTRIBUTING.md):

1. **Branch naming**: `feature/feature-name`, `fix/bug-description`
2. **Commit messages**: Clear, descriptive commit messages
3. **Pull requests**: One feature per PR, include tests and documentation
4. **Code review**: All changes require review before merging

### Continuous Integration

The CI pipeline includes:

- **Build verification**: Multiple compilers and platforms
- **Static analysis**: clang-tidy and clang-format checks
- **Test execution**: Unit tests, integration tests, benchmarks
- **Coverage analysis**: Code coverage reporting
- **Documentation**: Automatic API documentation generation
- **Security scanning**: Dependency and code security checks

### Local Development Cycle

```bash
# 1. Create feature branch
git checkout -b feature/new-data-structure

# 2. Set up pre-commit hooks
pre-commit install

# 3. Develop with continuous testing
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTS=ON
cmake --build build
cd build && ctest

# 4. Run quality checks
pre-commit run --all-files

# 5. Generate documentation
cmake --build build --target doc_doxygen

# 6. Commit and push
git add .
git commit -m "Add HAT-trie implementation with tests and benchmarks"
git push origin feature/new-data-structure
```

### Performance Optimization Workflow

1. **Profile first**: Use profiling tools to identify bottlenecks
2. **Benchmark**: Establish baseline performance metrics
3. **Optimize**: Make targeted improvements
4. **Verify**: Confirm improvements with benchmarks
5. **Document**: Update performance characteristics in documentation

### Release Process

1. **Version bump**: Update version in `version.cmake`
2. **Changelog**: Update `CHANGELOG.md` with changes
3. **Documentation**: Ensure all documentation is current
4. **Testing**: Run full test suite including performance tests
5. **Tag release**: Create annotated git tag
6. **Deploy**: Automatic deployment through CI

## Troubleshooting

### Common Build Issues

**CMake version too old:**

```bash
# Solution: Upgrade CMake
brew upgrade cmake  # macOS
```

**C++23 not supported:**

```bash
# Check compiler version
clang++ --version
g++ --version

# Solution: Upgrade compiler or use alternative
```

**Ninja not found:**

```bash
# Solution: Install Ninja
brew install ninja  # macOS
sudo apt install ninja-build  # Ubuntu
```

### Performance Issues

**Slow compilation:**

- Use Ninja instead of Make
- Enable parallel builds: `cmake --build build --parallel`
- Use precompiled headers for large projects

**Runtime performance:**

- Build with Release configuration
- Profile with appropriate tools (perf, Instruments, Visual Studio Profiler)
- Check for debug assertions in release builds

For additional help, please open an issue on GitHub or refer to the project documentation.
