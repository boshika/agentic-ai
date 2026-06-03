# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Udacity course exercise repository: **"Prompting for Effective LLM Reasoning with Gemini"**. It contains five progressive lessons on prompt engineering techniques using Google Vertex AI (Gemini 2.5 Flash). Each lesson builds on the previous one, culminating in a production-grade AI system in Lesson 5.

## Environment Setup

All exercises require a GCP project with Vertex AI enabled:

```bash
export PROJECT_ID=your-gcp-project-id
gcloud auth application-default login
pip install -r requirements.txt
```

Key dependencies: `google-genai>=1.62.0`, `python-dotenv`, `pytest`, `pytest-asyncio`.

## Running Tests

Each lesson has its own test runner located in `lesson-N-*/exercises/`:

```bash
# Run all tests verbosely
python test_<lesson>.py --verbose

# Run a specific component
python test_<lesson>.py --component cot       # lesson 2 example
python test_<lesson>.py --component react

# Run tests for a specific TODO
python test_<lesson>.py --todo 11

# Integration test across all lessons (lesson 5)
python test_feedback_loops.py --integration --all-lessons
```

Run a single exercise module directly to see its output:
```bash
python personas.py
python react_agent.py
python sequential_chain.py
```

## Architecture

The five lessons form a progressive pipeline — each lesson's outputs are consumed as inputs by later lessons:

```
Lesson 1 (Personas) → Lesson 2 (CoT/ReACT) → Lesson 3 (Optimization)
                                                       ↓
                             Lesson 5 (Feedback Loops) ← Lesson 4 (Chaining)
```

**Lesson 1 — Role-Based Prompting** (`lesson-1-role-based-prompting/exercises/`)  
Two files: `personas.py` (text-only persona design) and `personas_with_ai.py` (live Vertex AI testing). Defines `BUSINESS_ANALYST_PERSONA`, `MARKET_RESEARCHER_PERSONA`, and `STRATEGIC_CONSULTANT_PERSONA` constants reused in all later lessons.

**Lesson 2 — CoT & ReACT** (`lesson-2-cot-react-prompting/exercises/`)  
`cot_prompting.py` implements step-by-step Chain-of-Thought prompting. `react_agent.py` implements the Reason→Act→Observe loop with Gemini function calling (tools: `calculate`, `get_market_data`, `get_competitor_analysis`). Max 5 iterations per loop.

**Lesson 3 — Prompt Optimization** (`lesson-3-prompt-optimization/exercises/`)  
`prompt_analyzer.py` scores existing prompts (clarity, specificity, completeness, structure). `vertex_optimizer.py` calls the Vertex AI Prompt Optimizer API to apply systematic improvements and produce before/after metrics.

**Lesson 4 — Prompt Chaining** (`lesson-4-prompt-chaining/exercises/`)  
`sequential_chain.py` runs a 4-step Business Intelligence Chain: Market Overview → Competitive Analysis → Risk Assessment → Strategic Recommendations. Key dataclasses: `SequentialChain`, `ChainContext`, `ChainStep`, `StepResult`, `ChainResult`. Each step has a configurable `quality_threshold` (recommended: 0.55) and `ValidationLevel` (BASIC/STANDARD/STRICT). Enable `CHAIN_DEBUG=true` for step-by-step logging.

**Lesson 5 — Feedback Loops** (`lesson-5-feedback-loops/exercises/`)  
Capstone that imports all prior lessons. Three components: `self_validator.py` (confidence scoring, error detection, quality gates), `improvement_engine.py` (retry loops with parameter adjustment, trend analysis), `monitoring_dashboard.py` (real-time metrics, alerting). Cross-lesson import pattern:
```python
from lesson_1_personas import PersonaManager
from lesson_2_reasoning import CoTAgent, ReACTAgent
from lesson_3_optimization import VertexPromptOptimizer
from lesson_4_chaining import BusinessIntelligenceChain
```

## Lesson File Structure

Each lesson follows the same layout:
```
lesson-N-*/exercises/
  starter/       # Student files with TODO placeholders — edit these
  solution/      # Reference implementation — do not copy blindly
  README.md      # Detailed task descriptions and expected outputs
  test_*.py      # Test runner (runs against starter/ by default)
  requirements.txt
```

## Vertex AI Client Pattern

All lessons use this initialization:
```python
import vertexai
from google import genai
from google.genai.types import GenerateContentConfig

vertexai.init(project=project_id, location="us-central1")
client = genai.Client(vertexai=True)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=prompt,
    config=GenerateContentConfig(temperature=0.7, max_output_tokens=800)
)
```
