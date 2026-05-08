# Getting Started with Auto_Kaggle

A step-by-step guide to set up and run your first Kaggle competition with Auto_Kaggle.

## Prerequisites

- **Python 3.8+** installed
- **Kaggle account** (free)
- **Google account** for Gemini API (free tier available)
- **Command-line terminal/PowerShell**
- **~15 minutes** to set up

---

## Step 1: Get Your API Keys (5 minutes)

### Google Gemini API Key

1. Visit **https://ai.google.dev/**
2. Click **"Get API Key"** button
3. Select or create a Google Cloud project
4. Enable the Generative AI API
5. Click **"Create API Key"** (free tier)
6. **Copy the key** (you'll need it in Step 3)

**Note:** Free tier includes 60 API calls/minute, which is plenty for our use.

### Kaggle API Key

1. Log in to **https://www.kaggle.com/**
2. Go to **Settings** → **Account** (your profile menu)
3. Scroll to **"API"** section
4. Click **"Create New Token"**
5. This downloads `kaggle.json` file
6. **Save this file** (you'll need it in Step 3)

---

## Step 2: Clone and Install (5 minutes)

### Clone Repository

```bash
# Clone the repository
git clone https://github.com/Nuthan-hue/Auto_kaggle.git
cd Auto_kaggle

# Verify you're in the right directory
ls  # Should see README.md, setup.py, etc.
```

### Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate it
# On Linux/Mac:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Verify activation (should see (venv) prefix in terminal)
```

### Install Dependencies

```bash
# Install required packages
pip install -r requirements.txt

# Verify installation
python -c "import kaggle; import google.generativeai; print('✓ All packages installed')"
```

---

## Step 3: Configure API Keys (3 minutes)

### Option A: Using .env File (Recommended)

```bash
# Copy the template
cp .env.example .env

# Edit .env in your text editor
# Add your API keys:
# GEMINI_API_KEY=your-key-from-step-1
# KAGGLE_USERNAME=your-username
# KAGGLE_KEY=your-key-from-step-1
```

### Option B: Using Kaggle Configuration

```bash
# Create Kaggle config directory
mkdir -p ~/.kaggle

# Copy the kaggle.json file you downloaded
# Linux/Mac:
cp ~/Downloads/kaggle.json ~/.kaggle/

# Windows:
copy %USERPROFILE%\Downloads\kaggle.json %USERPROFILE%\.kaggle\

# Set permissions (Linux/Mac only)
chmod 600 ~/.kaggle/kaggle.json
```

### Verify Configuration

```bash
# Test Kaggle connection
kaggle competitions list

# Test Gemini connection
python -c "
import os
from dotenv import load_dotenv
load_dotenv()
key = os.getenv('GEMINI_API_KEY')
if key:
    print('✓ GEMINI_API_KEY configured')
else:
    print('✗ GEMINI_API_KEY not found')
"
```

---

## Step 4: Run Your First Competition (2 minutes)

### Quick Test (Recommended First)

```bash
# Run a quick 3-iteration test with Titanic
python tests/test_optimization_loop.py titanic

# This will:
# 1. Download Titanic dataset
# 2. Analyze the data
# 3. Run 3 optimization iterations
# 4. Display results
# Total time: ~5-10 minutes
```

### Watch the Output

You should see something like:

```
2026-05-08 15:30:42 - CoordinatorAgent - INFO - 🧠 Coordinator deciding...
2026-05-08 15:30:45 - CoordinatorAgent - INFO - 🎯 Decision: collect_data
2026-05-08 15:30:45 - CoordinatorAgent - INFO - 💭 Reasoning: No data collected yet
2026-05-08 15:30:52 - DataCollector - INFO - Downloaded titanic train.csv (891 rows)
2026-05-08 15:31:05 - CoordinatorAgent - INFO - 🎯 Decision: understand_problem
...
[More decisions and actions]
...
2026-05-08 15:45:32 - AgenticOrchestrator - INFO - ✅ Optimization Complete
Final Percentile: 22.5%
Target Met: False (target was 20%)
```

---

## Step 5: Run Full Competition (Optional)

Once the test passes, try the production run:

```bash
# Full 10-iteration run with target of 20%
python run_optimization_loop.py titanic 0.20

# This will run until:
# - Percentile reaches 20% or better, OR
# - Max iterations (10) reached, OR
# - No improvement for 3 iterations
# Total time: 30-90 minutes depending on competition
```

---

## Understanding the Output

### Decision Logs

```
🧠 Coordinator deciding...
  Looking at state, available actions, and goal
  
🎯 Decision: engineer_features
  Next action to execute
  
💭 Reasoning: Need more features for better predictions
  Why this action was chosen
```

### Action Execution

```
⚙️ Executing action: engineer_features
  Creating new features...
  - Added 5 new features
  - Total features: 15
✅ Action complete
```

### Results

```
📊 Final Results:
├─ Percentile: 18.5%
├─ Rank: 245 / 1,200
├─ Target: 20%
├─ Status: ✅ TARGET MET
└─ Time: 45 minutes
```

---

## Common Issues & Solutions

### Issue: "ModuleNotFoundError: No module named 'kaggle'"

**Solution:** Activate virtual environment and reinstall:
```bash
source venv/bin/activate  # or: venv\Scripts\activate on Windows
pip install -r requirements.txt
```

### Issue: "GEMINI_API_KEY not found"

**Solution:** Check your .env file:
```bash
# Verify .env exists
ls -la .env

# Check if key is set
grep GEMINI_API_KEY .env

# If not set, add it:
echo "GEMINI_API_KEY=your-actual-key" >> .env
```

### Issue: "Kaggle API credentials not found"

**Solution:** Use one of these approaches:

```bash
# Method 1: .env file
echo "KAGGLE_USERNAME=your-username" >> .env
echo "KAGGLE_KEY=your-api-key" >> .env

# Method 2: ~/.kaggle/kaggle.json
mkdir -p ~/.kaggle
echo '{"username":"your-username","key":"your-key"}' > ~/.kaggle/kaggle.json
chmod 600 ~/.kaggle/kaggle.json
```

### Issue: "Out of memory" error

**Solution:** Use data sampling:
```bash
# Edit .env
DATA_SAMPLE_SIZE=1000  # Use first 1000 rows instead of all
```

### Issue: Running very slowly

**Solution:** Enable GPU:
```bash
# Edit .env
ENABLE_GPU=true

# Or run command:
export ENABLE_GPU=true
python run_optimization_loop.py titanic 0.20
```

---

## Next Steps

### 1. Explore Different Competitions

```bash
# List available competitions
kaggle competitions list

# Try other competitions
python tests/test_optimization_loop.py house-prices-advanced-regression-techniques
python tests/test_optimization_loop.py nlp-getting-started
```

### 2. Monitor in Real-Time

```bash
# In a new terminal, watch logs as they're created
tail -f logs/kaggle_agent.log
```

### 3. Understand the Architecture

Read the comprehensive guides:
- **README.md** - Overview and features
- **ARCHITECTURE.md** - Technical deep-dive
- **QUICK_REFERENCE.md** - Command reference

### 4. Customize Behavior

Edit `.env` to customize:
- Target percentile: `DEFAULT_TARGET_PERCENTILE=0.10`
- Max iterations: `DEFAULT_MAX_ITERATIONS=15`
- Model selection: `ENABLE_XGBOOST=true`

### 5. Contribute

See **CONTRIBUTING.md** for:
- How to add new agents
- How to support new problem types
- How to improve the system

---

## Configuration Tips

### For Beginners
```bash
# Conservative settings - slower but more stable
COORDINATOR_TEMPERATURE=0.3
MAX_FEATURES=50
CROSS_VALIDATION_FOLDS=3
```

### For Advanced Users
```bash
# Aggressive settings - faster but might be noisier
COORDINATOR_TEMPERATURE=0.9
MAX_FEATURES=200
CROSS_VALIDATION_FOLDS=10
ENABLE_STACKING=true
```

### For Budget-Conscious Users
```bash
# Minimize API calls
DEFAULT_MAX_ITERATIONS=5
REQUEST_TIMEOUT=60
ENABLE_CACHE=true
```

---

## Project Structure

Once you're familiar, here's what's running:

```
Auto_Kaggle/
├── src/                 # Main source code
│   ├── agents/          # AI agents (coordinator, specialists)
│   ├── prompts/         # LLM prompts
│   └── utils/           # Utilities
├── tests/               # Test files
├── logs/                # Execution logs (created during run)
├── data/                # Downloaded datasets (created during run)
├── models/              # Trained models (created during run)
├── README.md            # Main documentation
├── ARCHITECTURE.md      # Technical details
├── .env                 # Your configuration (created from .env.example)
└── requirements.txt     # Python dependencies
```

---

## Performance Expectations

| Competition | Complexity | Time (test) | Time (full) |
|---|---|---|---|
| Titanic | Easy | 5-10 min | 30-60 min |
| House Prices | Medium | 10-15 min | 60-90 min |
| NLP | Hard | 15-20 min | 90-180 min |

**First run will be slower** (downloads dependencies, models, data)

---

## Getting Help

1. **Check logs:** `tail -f logs/kaggle_agent.log`
2. **Read docs:** Start with README.md, then ARCHITECTURE.md
3. **Check Quick Reference:** QUICK_REFERENCE.md for commands
4. **Open an issue:** GitHub Issues with error logs
5. **Read CONTRIBUTING.md:** For development questions

---

## What's Happening Behind the Scenes?

When you run `python tests/test_optimization_loop.py titanic`:

1. **System Initialization**
   - Load configuration from .env
   - Initialize AI models (Gemini)
   - Connect to Kaggle API

2. **Iteration Loop** (runs 3 times)
   - **Coordinator** decides next action
   - **Specialist agents** execute the action
   - Results stored and evaluated
   - Feedback sent back to Coordinator

3. **Per Action**
   - Download data / Analyze data / Engineer features
   - Train models / Make predictions
   - Submit to leaderboard
   - Check results

4. **Final Report**
   - Display metrics and progress
   - Save results to logs
   - Suggest improvements

---

## Common Next Questions

**Q: Can I use a different LLM instead of Gemini?**
A: Yes! See `.env` - supports OpenAI, Anthropic, or add custom.

**Q: How do I use this in production?**
A: See ARCHITECTURE.md - design is production-ready.

**Q: Can I extend this with custom agents?**
A: Yes! See CONTRIBUTING.md for adding new agents.

**Q: What competitions can it handle?**
A: Tabular and NLP fully supported. CV/Time-series architecture ready.

---

## Success!

You're now ready to run autonomous Kaggle competitions! 🎉

The AI Coordinator will handle all decisions. Just watch the logs and see how it adapts.

---

**Happy Competing!** 🚀

**Last Updated:** May 2026
