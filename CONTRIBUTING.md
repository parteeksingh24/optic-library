# Contributing to the Optic Library

Welcome to the Optic Library project! This educational toolkit explores advanced Data Structures and Algorithms (DSA) implementations in modern C++, going beyond traditional computer science curriculum. While this is primarily an educational project, contributions are welcomed!

## How to Contribute

### Feedback and Suggestions

Feedback and suggestions are always welcome. They can help improve the project's quality and usability:

- **Providing Feedback**: If you have any comments or suggestions, please feel free to open an issue on the GitHub repository with your feedback.

### Reporting Issues

If you notice any bugs or issues, please report them by opening an issue on GitHub. Include a clear title and description, and as much information as you feel is necessary.

### Feature Requests

If you have ideas for new features or improvements, I'd love to hear about them! Please open an issue on GitHub to discuss your ideas.

## Pull Requests

If you'd like to get involved on a particular improvement or feature, here's how you can make a contribution:

1. **Fork the Repository**: Start by forking the repository on GitHub.
2. **Clone Your Fork**: Clone your fork to your local machine.

    ```bash
    git clone https://github.com/yourusername/optic-library.git
    cd optic-library
    ```

3. **Create a Branch**: Create a new branch for your changes.

    ```bash
    git checkout -b feature/your-feature-name
    ```

4. **Make Your Changes**: Implement your feature or fix and thoroughly test your changes.
5. **Build the Project**: Ensure that your changes build properly using CMake.

    ```bash
    # Standard build
    cmake -B build
    cmake --build build
    
    # Optimized build (recommended)
    cmake -B build -G Ninja
    cmake --build build
    ```

6. **Run Tests**: Verify your changes don't break existing functionality.

    ```bash
    cd build
    ctest --output-on-failure
    ```

7. **Commit Your Changes**: Ensure your commit messages are clear.

    ```bash
    git commit -m "Add a concise description of your change"
    ```

8. **Push to Your Fork** and submit a pull request. All submissions will be reviewed thoughtfully and considered for inclusion in the project.

## Development Guidelines

### Technical Requirements

- C++23 compiler support required
- All code must include comprehensive unit tests
- Performance-critical implementations should include benchmarks
- Documentation using Doxygen conventions

### Code Quality

The project maintains high code quality standards using automated tooling:

- **clang-format**: Automatic code formatting
- **clang-tidy**: Static analysis and modern C++ checks
- **pre-commit hooks**: Automated quality checks before commits

For detailed technical setup and development workflows, see [`DEVELOPMENT.md`](DEVELOPMENT.md).

## What Makes a Good Contribution

### Data Structures and Algorithms

This project focuses on advanced, educational implementations:

- **Beyond Basic**: Implementations that go beyond typical CS curriculum
- **Educational Value**: Clear documentation explaining algorithms and design choices
- **Performance**: Efficient implementations with benchmarks comparing to standard library
- **Modern C++**: Leverage C++23 features appropriately

### Examples of Valuable Contributions

- Advanced data structures (HAT-tries, Fibonacci heaps, lock-free structures)
- Optimized algorithms (cache-oblivious, parallel, streaming)
- Performance improvements and benchmarks
- Documentation and educational examples
- Bug fixes and code quality improvements

## License

By contributing, you agree that your contributions will be licensed under the MIT License. See the `LICENSE` file for details.

Thank you for your interest in the Optic Library! Your contributions help make advanced computer science concepts more accessible for everyone.
