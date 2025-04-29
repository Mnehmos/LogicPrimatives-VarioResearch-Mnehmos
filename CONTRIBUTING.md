# Contributing to LogicPrimitives

Thank you for your interest in contributing to LogicPrimitives! This document provides guidelines and instructions for contributing to this project.

## Code of Conduct

This project adheres to the Contributor Covenant code of conduct. By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## How to Contribute

### Reporting Bugs

Before submitting a bug report:
- Check the issue tracker to see if the bug has already been reported
- Collect information about the bug (OS, version, steps to reproduce, etc.)

When reporting bugs, please include:
- A clear and descriptive title
- Detailed steps to reproduce the bug
- Expected vs. actual behavior
- Screenshots if applicable
- Environment information
- Any additional context

### Suggesting Enhancements

When suggesting enhancements, please include:
- A clear and descriptive title
- A detailed description of the proposed enhancement
- An explanation of why this enhancement would be useful
- Any relevant examples or mockups

### Pull Requests

1. Fork the repository
2. Create a new branch (`git checkout -b feature/your-feature-name`)
3. Make your changes
4. Run tests and ensure they pass
5. Commit your changes with clear, descriptive commit messages
6. Push to your branch (`git push origin feature/your-feature-name`)
7. Open a Pull Request

#### Pull Request Guidelines

- Follow the coding style and standards used in the project
- Include tests for new features or bug fixes
- Update documentation as necessary
- Keep pull requests focused on a single change
- Link any relevant issues

## Development Setup

### Prerequisites

- Python 3.9 or higher
- pip
- virtualenv (recommended)

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/your-username/logic-primitives.git
cd logic-primitives

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install development dependencies
pip install -r requirements-dev.txt
```

### Running Tests

```bash
pytest
```

## Documentation

Good documentation is essential for this project. Please document:
- New functions, classes, and methods
- Changes to the API
- Updates to the cognitive primitives or agent behaviors
- Complex algorithms or logic

## Style Guide

This project follows:
- PEP 8 for Python code
- Google docstring format for documentation
- Markdown for documentation files

## Extending Logic Primitives

### Adding New Primitives

To add a new cognitive primitive:
1. Create a new file in the `/logic_primitives/primitives` directory
2. Implement the primitive using the standard interface
3. Add tests in the `/tests/primitives` directory
4. Update the documentation

### Creating New Agents

To create a new specialized agent:
1. Create a new class in the `/logic_primitives/agents` directory
2. Implement the agent using the Agent base class
3. Add tests in the `/tests/agents` directory
4. Update the documentation

## License

By contributing to LogicPrimitives, you agree that your contributions will be licensed under the project's MIT license.

## Questions?

If you have any questions, feel free to open an issue or contact the maintainers.

Thank you for contributing to LogicPrimitives!