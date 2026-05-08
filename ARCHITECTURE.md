# Auto_Kaggle Architecture

This document provides a comprehensive overview of the system architecture, design patterns, and how components interact.

## Table of Contents

1. [System Overview](#system-overview)
2. [Core Components](#core-components)
3. [Agent Architecture](#agent-architecture)
4. [Workflow Execution](#workflow-execution)
5. [Data Flow](#data-flow)
6. [Decision Making](#decision-making)
7. [Optimization Loop](#optimization-loop)
8. [Extension Points](#extension-points)

---

## System Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    USER ENTRY POINTS                         │
│  (CLI, Programmatic, Web Interface)                          │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│              🧠 COORDINATOR AGENT (Brain)                    │
│  - Observes system state                                    │
│  - Makes workflow decisions                                 │
│  - Plans next actions dynamically                           │
│  - Learns from history                                      │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
│   Specialist │ │  Specialist │ │  Specialist │
│   Agents     │ │   Agents    │ │   Agents    │
│              │ │             │ │             │
│ • Collect    │ │ • Analyze   │ │ • Engineer  │
│ • Understand │ │ • Plan      │ │ • Train     │
│ • Process    │ │ • Optimize  │ │ • Submit    │
└──────────────┘ └─────────────┘ └─────────────┘
        │                │                │
└───────┴────────────────┴────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│           EXECUTION ENGINE & STATE MANAGER               │
│  - Manages action execution                             │
│  - Tracks system state                                  │
│  - Handles errors and retries                           │
│  - Maintains action history                             │
└────────────────┬─────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│            EXTERNAL SERVICES & DATA                      │
│  - Kaggle API (competitions, data, leaderboard)         │
│  - LLM APIs (Gemini, OpenAI, Anthropic)                │
│  - File System (data, models, cache)                    │
│  - Databases (results, history)                         │
└─────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. **Coordinator Agent** 🧠

**Location:** `src/agents/llm_agents/coordinator_agent.py`

The autonomous brain that makes all workflow decisions.

```python
class CoordinatorAgent:
    """
    Decides which action to take next based on:
    - Current state (what's been done)
    - Goal (reach target percentile)
    - Available actions (next possible steps)
    - History (what worked before)
    """
    
    def __init__(self, llm_config: Dict[str, Any]):
        self.llm = LLMProvider(llm_config)  # Gemini, OpenAI, etc.
        self.memory = ActionMemory()  # Stores action history
        
    async def decide_next_action(
        self, 
        state: SystemState
    ) -> Tuple[str, str]:  # (action, reasoning)
        """
        Input: Current system state (completed actions, metrics, errors)
        Process: 
          - Build context from state and history
          - Send to LLM with decision prompt
          - Parse LLM response
        Output: Next action to execute + reasoning
        """
```

**Key responsibilities:**
- Analyze system state
- Determine appropriate next action
- Provide reasoning for decisions
- Learn from previous iterations

---

### 2. **Specialist Agents** 🤖

**Location:** `src/agents/llm_agents/`

Specialized agents for specific tasks, called by Coordinator:

| Agent | Purpose | Key Methods |
|-------|---------|-------------|
| `DataCollectorAgent` | Download and organize competition data | `collect_data()` |
| `ProblemUnderstandingAgent` | Analyze problem description and requirements | `understand_problem()` |
| `DataAnalysisAgent` | Examine data characteristics | `analyze_data()` |
| `PreprocessingAgent` | Clean and prepare data | `preprocess_data()` |
| `StrategyPlanningAgent` | Create ML strategy | `plan_strategy()` |
| `FeatureEngineeringAgent` | Create and select features | `engineer_features()` |
| `ModelTrainingAgent` | Train ML models | `train_model()` |
| `SubmissionAgent` | Create and submit predictions | `submit_predictions()` |
| `EvaluationAgent` | Check leaderboard and results | `evaluate_results()` |
| `OptimizationAgent` | Suggest improvements | `optimize_strategy()` |

**Common Interface:**

```python
class BaseAgent:
    """Base class for all agents."""
    
    async def execute(self, context: Context) -> Result:
        """Execute the agent's main task."""
        pass
    
    def validate_inputs(self) -> bool:
        """Check prerequisites are met."""
        pass
    
    def generate_report(self) -> Dict:
        """Return execution summary."""
        pass
```

---

### 3. **Agentic Orchestrator** ⚙️

**Location:** `src/agents/orchestrator/orchestrator_agentic.py`

Manages overall workflow execution and state.

```python
class AgenticOrchestrator:
    """
    Orchestrates the entire workflow:
    1. Initialize system
    2. Loop: Coordinator decides → Execute action → Update state
    3. Check if goal achieved
    4. Repeat or finish
    """
    
    async def run(self, config: Dict) -> Results:
        """
        Main execution loop
        """
        while not self.goal_achieved():
            # Get next action from Coordinator
            action, reasoning = await self.coordinator.decide_next_action(
                self.state
            )
            
            # Execute action
            result = await self.execute_action(action, reasoning)
            
            # Update state with result
            self.update_state(action, result)
            
            # Check termination conditions
            if self.should_stop():
                break
                
        return self.generate_results()
```

---

### 4. **State Manager** 📊

**Location:** `src/agents/orchestrator/state_manager.py`

Tracks everything that's happened:

```python
class SystemState:
    """
    Maintains current state:
    - Completed actions (and results)
    - Current metrics (accuracy, percentile, etc.)
    - Available data (train, test, models)
    - Errors and warnings
    """
    
    completed_actions: List[ActionRecord]  # What's been done
    current_metrics: Dict[str, float]      # Performance metrics
    data_artifacts: Dict[str, Path]        # Generated files/models
    context: Dict[str, Any]                # Competition info, data stats
    iteration: int                         # Current iteration number
```

---

## Agent Architecture

### Agent Lifecycle

```
┌──────────────┐
│   CREATED    │
└──────┬───────┘
       │ (receives context)
       ▼
┌──────────────┐
│   VALIDATE   │ Check prerequisites
└──────┬───────┘
       │ (validation passes)
       ▼
┌──────────────┐
│   EXECUTE    │ Main task execution
└──────┬───────┘
       │ (execution succeeds)
       ▼
┌──────────────┐
│  GENERATE    │ Create report/results
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   COMPLETE   │ Return to Orchestrator
└──────────────┘
```

### Agent Communication

**Coordinator → Specialist:**

```python
# Coordinator decides
coordinator.decide_next_action(state)
# Returns: ("engineer_features", "Need more features for better predictions")

# Orchestrator calls specialist
specialist = agents["engineer_features"]
result = await specialist.execute(context)
# Returns: {
#     "success": True,
#     "new_features": ["age_group", "family_size"],
#     "metrics": {"correlation": 0.85}
# }
```

---

## Workflow Execution

### Typical Workflow

```
Iteration 1 (Baseline):
  1. collect_data          → Download from Kaggle
  2. understand_problem    → Read competition description
  3. analyze_data          → Statistics, correlations, missing values
  4. plan_strategy         → What models/approach?
  5. preprocess_data       → Handle missing values, encoding
  6. engineer_features     → Create new features
  7. train_model           → Train LightGBM, XGBoost
  8. submit_predictions    → Submit to leaderboard
  9. evaluate_results      → Check rank/percentile
  10. optimize_strategy    → What improved? What needs work?

Iteration 2+ (Optimization):
  - Coordinator analyzes feedback from previous iteration
  - Skips unnecessary steps (e.g., no preprocessing if no missing values)
  - Repeats beneficial steps (e.g., feature engineering again)
  - Tries new models based on analysis
  - Continues until target is achieved or max actions reached
```

### Dynamic Decision Making

**Example: Data Analysis Results**

```
State: {
  "completed_actions": ["collect_data", "understand_problem", "analyze_data"],
  "metrics": {
    "missing_values_pct": 0,
    "categorical_features": 3,
    "numerical_features": 8,
    "target_type": "binary"
  }
}

Coordinator reasoning:
  "Data has NO missing values (0%) - preprocessing is unnecessary.
   Skip preprocess_data.
   Move directly to feature engineering."

Action: engineer_features (SKIPPED preprocess_data!)
```

This demonstrates true autonomy: **the system adapts to data**, not the reverse.

---

## Data Flow

### Phase 1: Data Collection

```
┌─────────────────┐
│ Kaggle API Call │ (download titanic dataset)
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ Local File System       │
│ data/titanic/           │
│  ├── train.csv          │
│  └── test.csv           │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Pandas DataFrame        │
│ (in-memory)             │
└─────────────────────────┘
```

### Phase 2: Feature Processing

```
┌──────────────────────┐
│ Raw Features         │ age, sex, fare, embarked
│ + Statistics         │ 891 samples, 3% missing values
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Data Preprocessing   │ • Fill missing values
│                      │ • Encode categorical
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Feature Engineering  │ • Create interactions
│                      │ • Polynomial features
│                      │ • Domain-specific features
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Feature Selection    │ • Drop low-importance
│                      │ • Correlation analysis
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Final Features       │ Optimized feature set
│ Ready for Training   │
└──────────────────────┘
```

### Phase 3: Model Training

```
┌──────────────────────┐
│ Prepare Data         │ • Train/validation split
│ cv_folds = 5         │ • Scaling/normalization
└──────┬───────────────┘
       │
       ├─────────┬──────────┬──────────┐
       │         │          │          │
       ▼         ▼          ▼          ▼
┌──────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│LightGBM  │ │XGBoost │ │RandomF │ │LinearM │
│Model     │ │Model   │ │Model   │ │Model   │
└────┬─────┘ └───┬────┘ └───┬────┘ └───┬────┘
     │           │          │          │
     └─────┬─────┴──────┬───┴──────────┘
           │            │
           ▼            ▼
       ┌──────────────────────┐
       │ Ensemble Voting      │ Average predictions
       │ or Blending          │
       └──────┬───────────────┘
              │
              ▼
       ┌──────────────────────┐
       │ Final Predictions    │ Saved for submission
       └──────────────────────┘
```

---

## Decision Making

### Coordinator Prompt Structure

The Coordinator uses a sophisticated prompt system:

```
SYSTEM PROMPT:
  "You are an autonomous Kaggle competition agent..."
  [Instructions on available actions]
  [Decision criteria and constraints]
  [Examples of good decisions]

CONTEXT:
  - Competition info (problem type, evaluation metric)
  - Completed actions with results
  - Current metrics
  - Available resources (time, compute)

TASK:
  "Based on the context above, decide the next action.
   Consider:
   1. What have we accomplished?
   2. What's needed to reach the goal?
   3. What would be most impactful next?"

RESPONSE FORMAT:
  "DECISION: [action_name]
   REASONING: [why this action]
   CONFIDENCE: [1-10]
   ALTERNATIVE: [backup action if this fails]"
```

### Action Selection Logic

```python
async def decide_next_action(state: SystemState) -> str:
    """
    Decision tree (simplified):
    
    IF no data collected:
        → collect_data
    
    ELIF don't understand problem:
        → understand_problem
    
    ELIF haven't analyzed data:
        → analyze_data
    
    ELIF no strategy yet:
        → plan_strategy
    
    ELIF need to preprocess AND data needs it:
        → preprocess_data
    
    ELIF no features engineered:
        → engineer_features
    
    ELIF models not trained:
        → train_model
    
    ELIF haven't submitted:
        → submit_predictions
    
    ELIF need to evaluate:
        → evaluate_results
    
    ELIF below target AND not stuck:
        → optimize_strategy (try again)
    
    ELSE:
        → done (goal achieved or max iterations)
    """
```

---

## Optimization Loop

### Iteration-Based Improvement

```
START
  │
  ├─→ Iteration 1: Baseline
  │     └─→ All phases (1-10)
  │     └─→ Submit baseline solution
  │     └─→ Check metrics: 25% percentile
  │     └─→ Analysis: Need better features
  │
  ├─→ Iteration 2: Feature Optimization
  │     └─→ Skip phases 1-3 (already done)
  │     └─→ Skip phase 4 (strategy unchanged)
  │     └─→ Skip phase 5 (no preprocessing needed)
  │     └─→ Re-run phase 6: engineer_features (new ideas)
  │     └─→ Re-run phase 7: train_model
  │     └─→ Submit improved solution
  │     └─→ Check metrics: 18% percentile ✓ IMPROVED
  │
  ├─→ Iteration 3: Hyperparameter Tuning
  │     └─→ Skip features (already good)
  │     └─→ Re-run training with different hyperparams
  │     └─→ Submit
  │     └─→ Check metrics: 16% percentile ✓ IMPROVED
  │
  └─→ Iteration 4+: Continue until...
        - Target achieved (< 20%) ✅
        - No improvement for 2 iterations
        - Max iterations reached
        
END (Target Met: 16% < 20% ✓)
```

### Feedback Loop

```python
class OptimizationLoop:
    """
    Continuous improvement through feedback
    """
    
    for iteration in range(max_iterations):
        # 1. Execute workflow (phases 1-10)
        results = await orchestrator.run_optimization_iteration()
        
        # 2. Get metrics
        current_percentile = results['percentile']
        
        # 3. Compare with previous
        improvement = previous_percentile - current_percentile
        
        # 4. Update Coordinator with feedback
        state.add_feedback({
            'iteration': iteration,
            'percentile': current_percentile,
            'improvement': improvement,
            'what_worked': ['feature engineering'],
            'what_didnt': ['hyperparameter tuning']
        })
        
        # 5. Next iteration uses this feedback
        # Coordinator will repeat what worked, try new ideas
        
        # 6. Check termination
        if current_percentile < target_percentile:
            break  # Goal achieved!
```

---

## Extension Points

### 1. Adding New Agents

Create a new specialist agent:

```python
# File: src/agents/llm_agents/my_custom_agent.py

from src.agents.llm_agents.base_agent import BaseAgent

class MyCustomAgent(BaseAgent):
    """Custom agent for specific task."""
    
    async def execute(self, context: Context) -> Result:
        """Execute the custom task."""
        # Your implementation
        pass
    
    def validate_inputs(self) -> bool:
        """Check prerequisites."""
        return True
    
    def generate_report(self) -> Dict:
        """Return results."""
        return {}
```

Register in Orchestrator:

```python
# In orchestrator_agentic.py
self.agents['my_custom_action'] = MyCustomAgent(config)
```

### 2. Supporting New Problem Types

The system is designed to work with any problem type through the Coordinator's flexibility:

```python
# Computer Vision (Not yet implemented but architecture ready)
→ Update problem_understanding_agent to detect image classification
→ Update data_analysis_agent to analyze image statistics
→ Update strategy_planning_agent to recommend CNN/ViT
→ Models and training handle the rest automatically

# Time Series (Architecture ready)
→ detect_time_series_problem()
→ analyze_temporal_patterns()
→ plan_time_series_strategy()
→ AutoML handles LSTM/Prophet/ARIMA selection
```

### 3. Custom LLM Provider

Implement custom LLM support:

```python
# File: src/agents/llm_providers/custom_provider.py

class CustomLLMProvider(BaseLLMProvider):
    """Connect to your LLM."""
    
    async def call_llm(self, prompt: str) -> str:
        """Call your LLM."""
        pass
    
    def estimate_tokens(self, text: str) -> int:
        """Calculate token usage."""
        pass
```

---

## Performance Characteristics

### Computational Complexity

| Operation | Complexity | Time (Typical) |
|-----------|-----------|---|
| Data Download | O(n) | < 1 min |
| Data Analysis | O(n) | 2-5 min |
| Feature Engineering | O(n × m²) | 5-15 min |
| Model Training | O(n × m) | 10-30 min |
| Prediction | O(k) | < 1 min |
| Coordinator Decision | O(1) | 10-30 sec |

**Full Iteration (10 phases):** 30-90 minutes
**Test Loop (3 iterations):** 90-270 minutes

### Memory Requirements

- **Minimal:** 2 GB (small datasets, CPU)
- **Recommended:** 4-8 GB (medium datasets, GPU)
- **Optimal:** 16+ GB (large datasets, multi-model)

### API Rate Limits

- **Gemini:** ~60 requests/minute (free tier)
- **Kaggle:** 50 requests/6 hours
- **OpenAI:** Based on plan

---

## Error Handling & Retry Logic

```python
class ErrorHandler:
    """
    Handles failures gracefully
    """
    
    async def execute_with_retry(
        self, 
        action: str, 
        max_retries: int = 3
    ) -> Result:
        """Execute action with automatic retry."""
        
        for attempt in range(max_retries):
            try:
                result = await self.execute_action(action)
                return result
                
            except TemporaryError:
                # Retry: API throttle, timeout, etc.
                await asyncio.sleep(2 ** attempt)  # Exponential backoff
                
            except PermanentError:
                # Don't retry: invalid input, missing data, etc.
                raise
                
            except Exception as e:
                # Log and skip this action
                logger.error(f"Action {action} failed: {e}")
                return Result(success=False, error=str(e))
```

---

## Conclusion

The Auto_Kaggle architecture is built on these principles:

1. **Autonomy:** Coordinator makes all decisions
2. **Flexibility:** Adapts to any problem through agents
3. **Feedback:** Learns from iteration results
4. **Modularity:** Easy to extend with new agents
5. **Transparency:** Explains reasoning for decisions
6. **Resilience:** Handles errors gracefully

This design allows the system to work on competitions it's never seen before, making it truly autonomous.

---

**Last Updated:** May 2026
