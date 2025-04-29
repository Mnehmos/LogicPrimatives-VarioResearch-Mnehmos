# LogicPrimitives: Complete Guide

This comprehensive guide explains how to build AI research agents using logic primitives, MCP servers, and the multi-agent framework.

## Table of Contents

- [Introduction](#introduction)
- [Core Concepts](#core-concepts)
- [Logic Primitives Framework](#logic-primitives-framework)
- [Multi-Agent System](#multi-agent-system)
- [Implementation Guide](#implementation-guide)
- [Real Example: Research Process in Action](#real-example-research-process-in-action)
- [MCP Server Integration](#mcp-server-integration)
- [Practical Benefits](#practical-benefits)
- [Getting Started](#getting-started)
- [Conclusion](#conclusion)

## Introduction

If you've used tools like Perplexity, ChatGPT, Brave, or Claude for research, you've likely encountered these limitations:

- **One-Shot Generation**: They attempt to solve complex problems in a single prompt
- **Black Box Reasoning**: You can't inspect or verify intermediate steps
- **Contextual Amnesia**: They forget methodologies partway through analysis
- **Opaque Citations**: Sources are referenced but reasoning chains remain hidden
- **Integration Friction**: Research outputs rarely integrate with your workflow

LogicPrimitives solves these problems by breaking research into modular, traceable components that produce verifiable results.

## Core Concepts

The LogicPrimitives framework is built on three core concepts:

### 1. Logic Primitives

Simple cognitive operations that can be composed to perform complex reasoning:

- **observe**: Collect raw data without interpretation
- **define**: Establish frameworks and boundaries
- **distinguish**: Identify differences and classify information
- **infer**: Generate conclusions from evidence
- **reflect**: Question assumptions and identify gaps
- **synthesize**: Combine findings into coherent wholes
- **decide**: Make clear choices based on analysis
- **adapt**: Modify approaches based on new information

### 2. Multi-Agent Architecture

Specialized agents with distinct roles that work together:

- **Orchestrator**: Task decomposition & coordination (Strategic Planning, Problem-Solving)
- **Research Agent**: Information gathering & synthesis (Evidence Triangulation, Synthesizing Complexity)
- **Architect Agent**: Research framework design (Strategic Planning, Complex Decision-Making)
- **Debug Agent**: Verification & gap analysis (Root Cause Analysis, Hypothesis Testing)
- **Ask Agent**: Focused information retrieval (Fact-Checking, Critical Review)
- **Code Agent**: Technical implementation (Problem-Solving, Operational Optimization)

### 3. Boomerang Logic

A task delegation and completion protocol where:
1. Tasks originate from the Orchestrator with clear parameters
2. Tasks are processed by specialist agents
3. Tasks return results in a structured format to the Orchestrator

## Logic Primitives Framework

The power of this approach comes from breaking down complex research into primitive cognitive operations. Each operation becomes a separate function with its own prompt template and produces explicit artifacts.

### Observe

**Purpose**: Collect raw data without interpretation
**Input**: Direct sources or raw observations
**Output**: Structured collection of unprocessed information
**Example Prompt**: "Collect raw data points about European renewable energy market from 2020-2024"

### Define

**Purpose**: Establish frameworks, categories, and boundaries
**Input**: Raw observations requiring structure
**Output**: Clear analytical framework with defined boundaries
**Example Prompt**: "Define key market segments and metrics for renewable energy analysis"

### Distinguish

**Purpose**: Identify differences and classify information
**Input**: Defined categories and items to classify
**Output**: Clear distinctions between categories with identified patterns
**Example Prompt**: "Identify differences between solar, wind, and hydrogen growth patterns"

### Infer

**Purpose**: Generate conclusions from evidence
**Input**: Observations, definitions, and distinctions
**Output**: Evidence-based projections and conclusions with confidence levels
**Example Prompt**: "Project likely trends for 2025 based on historical data patterns"

### Reflect

**Purpose**: Question assumptions and identify gaps
**Input**: The entire research process so far
**Output**: Identified gaps, biases, and refinement priorities
**Example Prompt**: "Evaluate confidence levels and potential gaps in the analysis"

### Synthesize

**Purpose**: Combine findings into coherent wholes
**Input**: All previous cognitive operations
**Output**: Comprehensive analysis with integrated insights
**Example Prompt**: "Combine all findings into a comprehensive market analysis with traceable reasoning"

### Decide

**Purpose**: Make clear choices based on analysis
**Input**: Synthesized analysis with options
**Output**: Decision with justification and confidence assessment
**Example Prompt**: "Determine the most promising renewable energy sector based on the analysis"

### Adapt

**Purpose**: Modify approaches based on new information
**Input**: Decisions and new observations
**Output**: Refined approach or updated analysis
**Example Prompt**: "Adjust projections based on identified policy changes and market factors"

## Multi-Agent System

### Agent Role Definitions

For each agent, create a role definition with these standard sections:

```yaml
role_definition:
  mode_name: Research
  identity: >
    You are the investigation specialist focused on gathering reliable, 
    traceable evidence through systematic research...
  process_guidelines:
    - Phase 1: Initial exploration using observe primitive
    - Phase 2: Framework development using define primitive
    # Additional phases...
  communication_protocols:
    - Citation standards
    - Output formats
```

### Agent Interaction

Agents interact through the boomerang logic system:

```json
{
  "task_id": "renewable_energy_trends_001",
  "origin_mode": "Research",
  "destination_mode": "Orchestrator",
  "status": "complete",
  "result": "Completed European renewable energy market analysis in research/energy_markets_report.md"
}
```

### Research Task Format

Format every research subtask with these sections:

```markdown
# [Task Title]

## Context
[Background information and relationship to larger project]

## Scope
[Specific requirements and boundaries]
[Step-by-step instructions when appropriate]

## Expected Output
[Detailed description of deliverables]
[Format specifications]
[Quality criteria]

## [Optional] Additional Resources
[Relevant tips or examples]
[Links to reference materials]
```

## Implementation Guide

### 1. Directory Structure

Create this folder structure for your research:

```
/projects/research_[topic]/
├── raw/                 # Raw research outputs 
├── synthesis/           # Integrated analyses
├── final/               # Polished deliverables
├── .roo/                # Process tracking 
│   └── boomerang-state.json
└── README.md            # Project overview
```

### 2. Configure Agent Modes

For each agent, create a mode configuration file with the role definition, process guidelines, and communication protocols outlined above.

### 3. Set Up Boomerang Logic

Implement a tracking system for task delegation and completion based on the boomerang payload format.

### 4. Connect MCP Servers

Configure connections to the required MCP servers for extended capabilities.

## Real Example: Research Process in Action

Let's see how this works with a concrete research question: "How will chip shortages affect the EV market in 2026?"

### Traditional Approach (Perplexity/ChatGPT/etc.)

```
"Analyze how semiconductor shortages will impact the electric vehicle market in 2026"
→ [Single long response with mixed-quality information and uncertain sources]
```

### LogicPrimitives Approach

1. **Orchestrator**: Decomposes task into research components:
   ```
   Task ID: chip_ev_impact_001
   Subtasks:
   - Define semiconductor dependencies in EVs (to Architect)
   - Research current chip production capacity (to Research)
   - Analyze EV market growth forecasts (to Research)
   - Identify potential mitigation strategies (to Ask)
   - Synthesize impact scenarios (to Research)
   ```

2. **Architect Agent**: Uses define primitive
   ```
   // Framework for Chip Dependency Analysis
   EV Chip Dependency Framework {
     Critical Components: Battery Management, Autonomous Driving, Infotainment
     Supply Chain Tiers: Chip Design, Fabrication, Assembly, Integration
     Impact Categories: Production Delays, Feature Reduction, Cost Increases
   }
   ```

3. **Research Agent**: Uses observe primitive
   ```
   // Raw Data Collection
   Chip Production Capacity {
     Current Global Capacity: 22.3M wafers/month
     Planned Expansion: +18% by 2025 (TSMC, Intel, Samsung)
     EV-Specific Allocation: Currently 8.2% of chip production
     Production Bottlenecks: 3nm process, Packaging, Testing
   }
   ```

4. **Research Agent**: Uses distinguish primitive
   ```
   // Comparative Analysis
   EV Chip Requirement Segmentation {
     Economy EVs: 384 chips/vehicle (±42)
     Mid-Market EVs: 712 chips/vehicle (±85)
     Premium & Autonomous: 1,400+ chips/vehicle (±120)
     
     Critical Difference: ADAS systems account for 65% chip variation
     Supply Sensitivity: Premium segment 3.2x more vulnerable to shortages
   }
   ```

5. **Debug Agent**: Uses reflect primitive
   ```
   // Gap Analysis
   Research Limitations {
     Temporal Gap: Production projections beyond 2025 highly uncertain
     Data Gap: Actual chip allocations by manufacturers not publicly disclosed
     Assumption Risk: Analysis assumes current EV architecture persistence
     Confidence Assessment: Medium confidence in basic trends, low confidence in specific numbers
   }
   ```

6. **Research Agent**: Uses synthesize primitive
   ```
   // Integrated Analysis
   2026 EV Market Impact Projection {
     Projected Impact Severity: Moderate (confidence: medium)
     Primary Effect: 12-18% production shortfall for premium segment
     Secondary Effect: Feature reduction in 38% of mid-market models
     Price Impact: 4-7% MSRP increase for feature-complete vehicles
     Regional Variation: Asia-based manufacturers less affected (-40% impact)
     
     Critical Dependencies: Power management chips remain primary constraint
     Key Uncertainty: New fabs coming online in late 2025
   }
   ```

7. **Orchestrator**: Validates and integrates all components into final research report

## MCP Server Integration

### Core MCP Servers

1. **logic-mcp-primitives**: Provides the core cognitive operations
   - observe, define, distinguish, infer, reflect, synthesize, decide, adapt

2. **computer-use-primitives**: File and system operations
   - read_file, write_file, list_directory, run_command

3. **brave-search**: External information retrieval
   - brave_web_search, brave_local_search

4. **context7**: Documentation lookup
   - resolve-library-id, get-library-docs

5. **perplexity**: Enhanced search capabilities
   - search, fact_check, code_assistant

## Practical Benefits

| Benefit | Traditional AI | LogicPrimitives |
|---------|---------------|-------------------|
| **Citation Quality** | Links to sources without explaining relevance | Clear connection between sources and claims |
| **Research Reproducibility** | Different results each time | Consistent, traceable methodology |
| **Complex Problem Handling** | Struggles with multi-faceted questions | Systematically decomposes and reintegrates |
| **Work Integration** | Outputs rarely match existing workflows | Produces artifacts in your preferred formats |
| **Confidence Assessment** | Typically overconfident with no nuance | Explicit uncertainty tracking |
| **Reasoning Transparency** | Black box thinking | Visible intermediate steps |

## Getting Started

To implement your first LogicPrimitives research project:

1. Install the LogicPrimitives package
2. Set up your project directory structure
3. Configure your specialized agent roles
4. Implement boomerang logic for task tracking
5. Connect to required MCP servers
6. Start with a small research question and expand

## Conclusion

The LogicPrimitives approach represents a fundamental shift in how we can use AI for serious research. Rather than treating AI as a magic answer box, this system makes AI a true research partner—providing structure, traceability, and reliability that one-shot tools simply cannot match.

By breaking complex thinking into primitive operations, connecting specialized agents through a coordinator, and maintaining meticulous documentation, you can build a research system that truly deserves to be called intelligent.

---

*This guide was created using the LogicPrimitives methodology it describes—decomposing the writing process into observe, define, distinguish, infer, reflect, synthesize, decide, and adapt operations.*