# Quick Reference Guide

Fast lookup for common tasks in Auto_Kaggle.

## Table of Contents

- [Installation](#installation)
- [Running Competitions](#running-competitions)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Common Commands](#common-commands)
- [File Locations](#file-locations)

---

## Installation

```bash
# Clone and enter directory
git clone https://github.com/Nuthan-hue/Auto_kaggle.git
cd Auto_kaggle

# Create virtual environment
python -m venv venv && source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure API keys
cp .env.example .env
# Edit .env and add your keys
```

---

## Running Competitions

### Quick Test (3 iterations, ~10 min)

```bash
python tests/test_optimization_loop.py titanic
```

### Full Run (10 iterations, ~60 min)

```bash
python run_optimization_loop.py titanic 0.20
```

### With Custom Target

```bash
python run_optimization_loop.py titanic 0.10 --max-actions 100
```

### Test Individual Phase

```bash
python tests/test_phase_1_data_collection.py titanic
python tests/test_phase_2_problem_understanding.py titanic
# ... continue for phases 3-10
```

---

## Configuration

### Environment Variables (.env)

```bash
# Required
GEMINI_API_KEY=your-key
KAGGLE_USERNAME=your-username
KAGGLE_KEY=your-key

# Optional but recommended
LOG_LEVEL=INFO
ENABLE_GPU=true
DEFAULT_TARGET_PERCENTILE=0.20

# Advanced
COORDINATOR_TEMPERATURE=0.7
BATCH_SIZE=32
CROSS_VALIDATION_FOLDS=5
```

### Kaggle Setup

```bash
# Download kaggle.json from: https://www.kaggle.com/settings/account
mkdir -p ~/.kaggle
mv ~/Downloads/kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: No module named 'src'` | Run from project root: `cd Auto_kaggle` |
| `GEMINI_API_KEY not found` | Create `.env` file: `cp .env.example .env` then edit |
| `Kaggle API credentials not found` | Setup `~/.kaggle/kaggle.json` or add to `.env` |
| Out of memory | Reduce `BATCH_SIZE=16` or `DATA_SAMPLE_SIZE=1000` |
| Coordinator repeats same action | Increase `COORDINATOR_TEMPERATURE=0.8` |
| Slow model training | Enable GPU: `export ENABLE_GPU=true` |
| API rate limit errors | Reduce `MAX_CONCURRENT_REQUESTS=5` |

---

## Common Commands

### Run & Monitor

```bash
# Run with test competition
python tests/test_optimization_loop.py titanic

# View logs in real-time
tail -f logs/kaggle_agent.log

# Run and save output
python run_optimization_loop.py titanic 0.20 | tee run.log

# Check GPU usage (during training)
watch nvidia-smi  # Linux/Mac
gpustat                # Alternative
```

### Interactive Mode

```bash
# Launch interactive CLI
python src/main.py

# Follow prompts to configure and run
```

### Testing & Validation

```bash
# Run all tests
pytest tests/ -v

# Run specific test
pytest tests/test_phase_1_data_collection.py -v

# With coverage
pytest tests/ --cov=src --cov-report=html
```

### Development

```bash
# Format code
black src/ tests/

# Check code style
flake8 src/ tests/

# Sort imports
isort src/ tests/

# Type checking
mypy src/
```

---

## File Locations

### Entry Points

```
src/main.py                    # Interactive CLI entry point
src/cli.py                     # Command-line interface
run_agentic.py                # Quick launcher
run_optimization_loop.py       # Optimization run script
```

### Core Agents

```
src/agents/orchestrator/
  ├── orchestrator_agentic.py  # Main orchestrator
  └── state_manager.py         # State tracking

src/agents/llm_agents/
  ├── coordinator_agent.py     # Autonomous brain
  ├── data_collection_agent.py
  ├── problem_understanding_agent.py
  ├── data_analysis_agent.py
  ├── preprocessing_agent.py
  ├── strategy_planning_agent.py
  ├── feature_engineering_agent.py
  ├── model_training_agent.py
  ├── submission_agent.py
  ├── evaluation_agent.py
  └── optimization_agent.py
```

### Configuration & Prompts

```
src/prompts/
  ├── coordinator_agent.txt   # Coordinator instructions
  ├── data_analysis.txt       # Data analysis prompt
  ├── feature_engineering.txt # Feature engineering guide
  └── ... (other prompts)

.env                          # Your API keys (create from .env.example)
.env.example                  # Template for environment variables
```

### Data & Results

```
data/
  └── <competition_name>/    # Downloaded competition data
      ├── train.csv
      ├── test.csv
      └── ...

models/
  └── <competition_name>/    # Trained models
      ├── model_1.pkl
      ├── model_2.pkl
      └── ...

submissions/
  └── <competition_name>/    # Submission files
      ├── submission_1.csv
      ├── submission_2.csv
      └── ...

logs/
  └── kaggle_agent.log        # System logs
```

### Tests

```
tests/
  ├── test_phase_1_data_collection.py
  ├── test_phase_2_problem_understanding.py
  ├── ... (phases 3-10)
  ├── test_optimization_loop.py
  └── conftest.py             # Pytest configuration
```

### Documentation

```
README.md                      # Main documentation
GETTING_STARTED.md            # Step-by-step guide
ARCHITECTURE.md               # Technical architecture
CONTRIBUTING.md               # Development guidelines
QUICK_REFERENCE.md            # This file
.env.example                  # Environment variable template
```

---

## Competition Selection

### Recommended for Testing

```bash
# Easy (5-15 min)
python tests/test_optimization_loop.py titanic
python tests/test_optimization_loop.py iris

# Medium (15-30 min)
python tests/test_optimization_loop.py house-prices-advanced-regression-techniques
python tests/test_optimization_loop.py nlp-getting-started

# Challenging (30-60+ min)
python tests/test_optimization_loop.py digit-recognizer
python tests/test_optimization_loop.py jigsaw-toxic-comment-classification-challenge
```

### Get Competition Names

```bash
# List available Kaggle competitions
kaggle competitions list

# List top active competitions
kaggle competitions list --page 1
```

---

## Performance Tips

### For Faster Results

```bash
# Reduce iterations
python run_optimization_loop.py titanic 0.30 --max-actions 30

# Sample data for testing
export DATA_SAMPLE_SIZE=1000

# Disable GPU if it's slow
export ENABLE_GPU=false
```

### For Better Results

```bash
# Increase iterations
python run_optimization_loop.py titanic 0.10 --max-actions 100

# Use all data
export DATA_SAMPLE_SIZE=None

# Enable GPU
export ENABLE_GPU=true

# Increase CV folds
export CROSS_VALIDATION_FOLDS=10

# Increase timeout for long-running competitions
export SUBMISSION_TIMEOUT=600
```

---

## API Keys Management

### Getting Keys

**Google Gemini:**
1. Visit https://ai.google.dev/
2. Click "Get API Key"
3. Create/select project
4. Copy key

**Kaggle:**
1. Visit https://www.kaggle.com/settings/account
2. Scroll to "API" section
3. Click "Create New API Token"
4. Downloads `kaggle.json`

### Storing Securely

```bash
# Option 1: .env file (local development)
cp .env.example .env
# Edit and add your keys
# ⚠️ Never commit .env to git

# Option 2: Environment variables (production)
export GEMINI_API_KEY="your-key"
export KAGGLE_USERNAME="your-username"
export KAGGLE_KEY="your-key"

# Option 3: System keychain (macOS)
security add-generic-password -a "gemini" -s "api_key" -w "your-key"
```

---

## Development Workflow

### Create Feature Branch

```bash
git checkout -b feature/your-feature
```

### Make Changes

```bash
# Edit files...

# Format code
black src/ tests/

# Run tests
pytest tests/ -v

# Commit changes
git commit -m "feat: Description of feature"
```

### Submit PR

```bash
# Push to fork
git push origin feature/your-feature

# Create Pull Request on GitHub
# - Write clear description
# - Reference related issues
# - Wait for review
```

---

## Useful Links

- **[GitHub Repository](https://github.com/Nuthan-hue/Auto_kaggle)**
- **[Kaggle Competitions](https://www.kaggle.com/competitions)**
- **[Google Gemini API](https://ai.google.dev/)**
- **[Full Documentation](README.md)**
- **[Architecture Details](ARCHITECTURE.md)**
- **[Getting Started Guide](GETTING_STARTED.md)**

---

## Still Need Help?

1. **Check documentation** - README.md, ARCHITECTURE.md, GETTING_STARTED.md
2. **Review logs** - `tail -f logs/kaggle_agent.log`
3. **Search issues** - GitHub Issues
4. **Open an issue** - With error logs and steps to reproduce

---

**Last Updated:** May 2026
