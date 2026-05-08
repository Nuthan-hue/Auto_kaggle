# Contributing to Auto_Kaggle

Thank you for your interest in contributing to the **Auto_Kaggle** project! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Making Changes](#making-changes)
- [Coding Standards](#coding-standards)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Reporting Issues](#reporting-issues)

---

## Code of Conduct

We are committed to providing a welcoming and inspiring community for all. Please read and adhere to our Code of Conduct:

- Be respectful and inclusive
- Welcome diverse perspectives
- Focus on constructive feedback
- Report unacceptable behavior to the maintainers

---

## Getting Started

### Prerequisites

- Python 3.8+
- Git
- A Kaggle account
- Google Gemini API key (free tier available)

### Fork and Clone

```bash
# Fork the repository on GitHub
# Clone your fork
git clone https://github.com/YOUR_USERNAME/Auto_kaggle.git
cd Auto_kaggle

# Add upstream remote
git remote add upstream https://github.com/Nuthan-hue/Auto_kaggle.git
```

---

## Development Setup

### 1. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 2. Install Dependencies

```bash
# Install core dependencies
pip install -r requirements.txt

# Install development dependencies
pip install pytest pytest-cov pytest-asyncio black isort flake8 mypy
```

### 3. Setup Environment Variables

```bash
# Copy the example file
cp .env.example .env

# Edit .env with your credentials
GEMINI_API_KEY=your-api-key
KAGGLE_USERNAME=your-username
KAGGLE_KEY=your-kaggle-key
```

### 4. Verify Setup

```bash
# Run a quick test
python -m pytest tests/ -v
```

---

## Making Changes

### Create a Feature Branch

```bash
# Update main branch
git fetch upstream
git checkout master
git merge upstream/master

# Create feature branch
git checkout -b feature/your-feature-name
# or for bug fixes:
git checkout -b fix/bug-description
```

### Naming Conventions

- Feature branches: `feature/descriptive-name`
- Bug fixes: `fix/bug-description`
- Documentation: `docs/topic-name`
- Refactoring: `refactor/component-name`

---

## Coding Standards

### Python Style

We follow **PEP 8** with the following tools:

```bash
# Format code with Black
black src/ tests/

# Sort imports
isort src/ tests/

# Check code style
flake8 src/ tests/

# Type checking
mypy src/
```

### Formatting Rules

**Line Length:** 100 characters (Black default)

**Example:**

```python
# ✅ Good
async def run_optimization_loop(
    competition_name: str,
    target_percentile: float,
    max_iterations: int = 10
) -> Dict[str, Any]:
    """Run the optimization loop for a competition.
    
    Args:
        competition_name: Name of the Kaggle competition
        target_percentile: Target percentile (0.0-1.0)
        max_iterations: Maximum iterations allowed
        
    Returns:
        Dictionary with results and metrics
    """
    # Implementation
    pass

# ❌ Avoid
async def run_optimization_loop(competition_name: str, target_percentile: float, max_iterations: int = 10) -> Dict[str, Any]:
    pass
```

### Docstring Format

Use Google-style docstrings:

```python
def analyze_data(filepath: str, target_column: str = None) -> pd.DataFrame:
    """Analyze dataset characteristics and return statistics.
    
    Performs comprehensive analysis including:
    - Missing value detection
    - Data type inference
    - Statistical summaries
    - Outlier detection
    
    Args:
        filepath: Path to the data file (CSV, Parquet, etc.)
        target_column: Optional target column for correlation analysis
        
    Returns:
        DataFrame with analysis results
        
    Raises:
        FileNotFoundError: If file doesn't exist
        ValueError: If target_column not found in data
        
    Example:
        >>> results = analyze_data('data.csv', 'target')
        >>> print(results['missing_values'])
    """
```

### Commenting Guidelines

```python
# ✅ Good comments (Why, not What)
# Skip preprocessing if data has no missing values
if data.isnull().sum().sum() == 0:
    return data

# ❌ Avoid (restating the code)
# Set x to 5
x = 5
```

---

## Testing

### Writing Tests

Place tests in the `tests/` directory with the naming convention `test_*.py`:

```python
# tests/test_data_analysis.py
import pytest
from src.agents.data_analysis import analyze_data

class TestDataAnalysis:
    """Test suite for data analysis module."""
    
    @pytest.fixture
    def sample_data(self):
        """Fixture providing sample data."""
        return pd.DataFrame({
            'feature1': [1, 2, 3, 4, 5],
            'feature2': [10, 20, 30, 40, 50]
        })
    
    def test_analyze_data_returns_dataframe(self, sample_data):
        """Test that analyze_data returns a DataFrame."""
        result = analyze_data(sample_data)
        assert isinstance(result, pd.DataFrame)
    
    @pytest.mark.asyncio
    async def test_async_operation(self):
        """Test async operations."""
        result = await some_async_function()
        assert result is not None
```

### Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/test_data_analysis.py -v

# Run with coverage
pytest tests/ --cov=src --cov-report=html

# Run async tests
pytest tests/ -v -m asyncio
```

### Test Coverage

Aim for **at least 80% code coverage**:

```bash
pytest tests/ --cov=src --cov-report=term-missing
```

---

## Submitting Changes

### Commit Guidelines

```bash
# Make focused commits
git commit -m "feat: Add new feature description"

# Good commit message structure
# <type>(<scope>): <subject>
#
# <body>
#
# <footer>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Examples:**

```
feat(coordinator): Add decision logging for debugging
fix(data-analysis): Prevent division by zero in outlier detection
docs: Update installation instructions
refactor(agents): Simplify agent initialization
test(orchestrator): Add integration tests for optimization loop
```

### Push Changes

```bash
# Push to your fork
git push origin feature/your-feature-name

# Create Pull Request on GitHub
# - Write clear PR title and description
# - Reference any related issues (#123)
# - Include screenshots/logs if relevant
```

### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix (fixes #123)
- [ ] New feature (related to #123)
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Added tests
- [ ] Tests pass locally
- [ ] Coverage is maintained

## Checklist
- [ ] Code follows style guidelines
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings generated
```

---

## Reporting Issues

### Before Reporting

1. Search existing issues to avoid duplicates
2. Check the FAQ and troubleshooting guide in README
3. Try reproducing with latest code

### Issue Template

```markdown
## Description
Clear description of the issue

## To Reproduce
Steps to reproduce:
1. 
2. 
3. 

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: 
- Python version: 
- Auto_Kaggle version: 
- Kaggle competition: 

## Error Logs
```
Paste error logs here
```

## Screenshots
(if applicable)
```

---

## Contribution Areas

### 🎯 High Priority

- **Computer Vision Support** - Implement CV agent
- **Time Series Support** - Add time series handling
- **Performance Optimization** - Speed up model training
- **Bug Fixes** - Address open issues

### 📚 Documentation

- Add tutorials
- Improve API docs
- Create video guides
- Add more examples

### 🧪 Testing

- Increase test coverage
- Add integration tests
- Performance benchmarks
- Edge case tests

### 🔧 Tooling

- Improve logging
- Add monitoring
- Enhance CLI
- Build dashboards

---

## Questions?

- Open an issue with the `question` label
- Check existing discussions
- Review documentation
- Ask in project discussions

---

## Recognition

All contributors will be recognized in:
- `CONTRIBUTORS.md`
- Project acknowledgments
- Release notes

Thank you for contributing! 🙏

---

**Last Updated:** May 2026
**Maintainer:** Nuthan-hue
