# CONTRIBUTING.md

## Contributing to AICRUSH

We appreciate your interest in contributing to AICRUSH! This document provides guidelines for contributing to our AI coding assistant project.

## 🎯 Project Mission

AICRUSH aims to build a **reliable, practical AI coding assistant** that generates accurate, production-ready code. We focus on:

- **Quality over quantity**: Well-tested, reliable features
- **Security first**: All contributions must maintain security standards
- **Practical solutions**: Code that solves real-world problems
- **Incremental improvement**: Steady, measurable progress

## 🚀 Getting Started

### Prerequisites

- Python 3.8+
- Google Cloud SDK
- Docker (for containerized development)
- VS Code (recommended IDE)

### Development Setup

1. **Fork and Clone**
   ```bash
   git clone https://github.com/nibertinvestments/AICRUSH.git
   cd AICRUSH
   ```

2. **Environment Setup**
   ```bash
   # Copy environment template
   cp .env.template .env
   # Add your configuration to .env (never commit this file)
   ```

3. **Install Dependencies**
   ```bash
   # Install Python dependencies
   pip install -r requirements.txt
   
   # Verify Google Cloud connection
   gcloud auth application-default login
   ```

4. **Run Tests**
   ```bash
   # Run test suite
   python -m pytest tests/
   
   # Run security checks
   bandit -r src/
   ```

## 📝 Development Workflow

### 1. Create a Feature Branch
```bash
git checkout -b feature/your-feature-name
```

### 2. Make Your Changes
- Follow our [coding standards](#coding-standards)
- Write tests for new functionality
- Update documentation as needed
- Ensure security best practices

### 3. Test Your Changes
```bash
# Run unit tests
pytest tests/unit/

# Run integration tests
pytest tests/integration/

# Check code quality
flake8 src/
black --check src/
mypy src/
```

### 4. Submit a Pull Request
- Use our [PR template](#pull-request-template)
- Provide clear description of changes
- Link to any related issues
- Request review from maintainers

## 🔧 Coding Standards

### Python Code Style
We follow PEP 8 with these specific guidelines:

```python
# Use type hints
def process_data(input_files: List[str], config: Dict[str, Any]) -> ProcessingResult:
    """Process training data files.
    
    Args:
        input_files: List of file paths to process
        config: Configuration dictionary
        
    Returns:
        ProcessingResult with status and metrics
    """
    
# Use dataclasses for structured data
@dataclass
class ProcessingResult:
    success: bool
    files_processed: int
    errors: List[str]
    processing_time: float
```

### Documentation Standards
- **Docstrings**: All public functions and classes must have docstrings
- **Type Hints**: Use type hints for all function parameters and returns
- **Comments**: Explain complex logic, not obvious code
- **README Updates**: Update documentation for user-facing changes

### Testing Requirements
- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test component interactions
- **Coverage**: Maintain >80% code coverage
- **Security Tests**: Test for common vulnerabilities

## 🔒 Security Guidelines

### Never Commit Secrets
```bash
# ✅ Good: Use environment variables
api_key = os.environ.get('API_KEY')

# ❌ Bad: Hardcoded secrets
api_key = "sk-1234567890abcdef"
```

### Google Cloud Security
- Use service accounts with minimal permissions
- Enable audit logging for all services
- Regular security reviews and key rotation
- Follow Google Cloud security best practices

### Code Security
- Validate all user inputs
- Use parameterized queries for databases
- Implement proper error handling
- Regular dependency security updates

## 🧪 Testing Guidelines

### Test Structure
```
tests/
├── unit/              # Fast, isolated tests
├── integration/       # Service interaction tests
├── security/          # Security validation tests
└── fixtures/          # Test data and mocks
```

### Writing Tests
```python
import pytest
from unittest.mock import Mock, patch

class TestDataProcessor:
    """Test suite for data processing functionality."""
    
    def setup_method(self):
        """Set up test fixtures before each test."""
        self.processor = DataProcessor(test_config)
    
    def test_process_valid_data(self):
        """Test processing with valid input data."""
        # Arrange
        test_data = create_test_data()
        
        # Act
        result = self.processor.process(test_data)
        
        # Assert
        assert result.success is True
        assert len(result.errors) == 0
```

## 📋 Pull Request Process

### PR Template
When creating a PR, please include:

- **Description**: Clear summary of changes
- **Type**: Bug fix, feature, documentation, etc.
- **Testing**: How changes were tested
- **Security**: Any security considerations
- **Documentation**: Updates made to docs

### Review Process
1. **Automated Checks**: All CI checks must pass
2. **Security Review**: Security team review for sensitive changes
3. **Code Review**: At least one maintainer approval required
4. **Testing**: Manual testing for complex features
5. **Merge**: Squash and merge after approval

### Review Checklist
- [ ] Code follows project standards
- [ ] Tests pass and provide adequate coverage
- [ ] Documentation is updated
- [ ] Security considerations addressed
- [ ] Performance impact evaluated
- [ ] Cloud cost impact considered

## 🌟 Types of Contributions

### 🐛 Bug Reports
Use our bug report template with:
- Clear description of the issue
- Steps to reproduce
- Expected vs actual behavior
- Environment details
- Logs or error messages

### 💡 Feature Requests
Use our feature request template with:
- Problem statement
- Proposed solution
- Alternative solutions considered
- Additional context

### 📚 Documentation
- Fix typos or unclear instructions
- Add examples or use cases
- Improve API documentation
- Create tutorials or guides

### 🔧 Code Contributions
- Bug fixes
- Performance improvements
- New features
- Refactoring for maintainability

## 🏷️ Issue Labels

We use these labels to organize issues:

- `bug`: Something isn't working
- `enhancement`: New feature or improvement
- `documentation`: Documentation updates
- `security`: Security-related issues
- `performance`: Performance improvements
- `good first issue`: Good for newcomers
- `help wanted`: Extra attention needed

## 📊 Performance Guidelines

### Cloud Cost Awareness
- Monitor resource usage in development
- Optimize algorithms for efficiency
- Use appropriate instance types
- Implement auto-scaling where possible

### Code Performance
- Profile code for bottlenecks
- Use async/await for I/O operations
- Batch operations when possible
- Cache frequently accessed data

## 🎓 Learning Resources

### Project-Specific
- [Architecture Documentation](docs/architecture/)
- [API Documentation](docs/api/)
- [Deployment Guides](docs/deployment/)

### General Resources
- [Google Cloud AI/ML Documentation](https://cloud.google.com/ai-platform)
- [Python Best Practices](https://docs.python.org/3/tutorial/)
- [Docker for ML](https://docs.docker.com/)

## 🤝 Community

### Communication Channels
- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: Architecture and design discussions
- **Pull Requests**: Code review and collaboration

### Code of Conduct
We follow the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). Please read it to understand our community standards.

## 📞 Getting Help

### Before Asking for Help
1. Check existing documentation
2. Search through issues
3. Review pull requests
4. Read the code

### How to Ask for Help
- Create a GitHub Discussion for general questions
- Open an issue for specific bugs or features
- Provide as much context as possible
- Be respectful and patient

## 🎉 Recognition

Contributors are recognized through:
- Attribution in release notes
- Contributor sections in documentation
- GitHub contributor graphs
- Special recognition for significant contributions

---

Thank you for contributing to AICRUSH! Together, we're building practical AI tools that help developers write better code.

For questions about contributing, please create a GitHub Discussion or contact the maintainers.