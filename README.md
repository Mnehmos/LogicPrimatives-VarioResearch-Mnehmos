# LogicPrimitives: Building Research Agents with Modular Cognitive Operations

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Version: 2.0.0](https://img.shields.io/badge/Version-2.0.0-green.svg)](https://github.com/username/LogicPrimitives)

**LogicPrimitives** is a conceptual framework for creating AI research agents that produce transparent, traceable, and reliable results by breaking complex analysis into primitive cognitive operations. This approach dramatically outperforms one-shot generations used by mainstream AI tools.

> **Note**: This repository contains conceptual guides and documentation only, not installable software.

## ⚠️ Implementation Disclaimer

LogicPrimitives was originally developed as a custom MCP server, but that implementation is not release-ready as it was an initial version with API key management issues. There are two main approaches to implement this framework:

1. **Build your own Logic MCP primitive**: Create a server that calls models outside the chat to make these decisions in isolation, providing a cleaner separation of cognitive operations.

2. **Document each primitive in Markdown**: Use sequential .md files to document each primitive step, allowing for structured thinking that can be traced and verified.

The demonstration in this repository uses the Markdown approach to show the concept in action.

## 🔍 The Problem with Traditional AI Research

Standard AI tools like Perplexity, ChatGPT, Claude, or Brave typically:
- Generate conclusions without showing intermediate reasoning
- Produce inconsistent outputs with each run
- Create phantom citations without proper evidence
- Deliver uniform confidence across all conclusions
- Cannot identify their own biases and limitations

## 💡 Our Solution: Logic Primitives with MCP Servers

LogicPrimitives breaks research into modular cognitive operations:

```
observe → define → distinguish → infer → reflect → synthesize → decide → adapt
```

Each operation produces a separate, verifiable research artifact connected through Model Context Protocol (MCP) servers and specialized agent roles.

## 🧩 Core Components

### 1. Logic Primitives Library

- **observe**: Collect raw data without interpretation
- **define**: Establish frameworks, categories, and boundaries
- **distinguish**: Identify differences and classify information
- **infer**: Generate conclusions with explicit confidence levels
- **reflect**: Critically evaluate the analysis process
- **synthesize**: Integrate all components into a coherent whole
- **decide**: Make clear choices based on analysis
- **adapt**: Modify approaches based on new information

### 2. Multi-Agent Framework

- **Orchestrator**: Task decomposition & coordination
- **Research Agent**: Information gathering & synthesis
- **Architect Agent**: Framework design
- **Debug Agent**: Verification & gap analysis
- **Ask Agent**: Focused information retrieval
- **Code Agent**: Technical implementation

### 3. Boomerang Logic & Task Tracking

Every task:
1. Originates from the Orchestrator with clear parameters
2. Gets processed by a specialist agent
3. Returns results in a standardized format to the Orchestrator

### 4. MCP Server Integration

- **logic-mcp-primitives**: Core thinking operations
- **computer-use-primitives**: File operations and storage
- **brave-search**: External information retrieval
- **context7**: Documentation lookup
- **perplexity**: Enhanced search capabilities

## 📚 Documentation

- [Complete Guide](docs/guide.md): Comprehensive explanation of the LogicPrimitives approach
- [Multi-Agent Architecture](docs/multi-agent-architecture.md): Detailed explanation of specialized agents
- [MCP Server Integration](docs/mcp-server-integration.md): How the framework integrates with MCP servers
- [Process Patterns](docs/process-patterns.md): Common primitive chains for specific use cases
- [Real-World Application](docs/real_world_application.md): How this framework integrates with external data sources
- [MCP Server Setup Guide](docs/mcp-server-setup-guide.md): Technical implementation guide for setting up MCP servers
- [Research Agents Guide](docs/research-agents-guide.md): Comprehensive guide to building your own AI research team

## 🔬 Demonstration

Explore our [AI Election Impact Demo](demo/README.md) to see logic primitives in action. This demonstration analyzes how AI might impact democratic elections in 2026 using the complete cognitive workflow:

1. [Observe](demo/1-observe.md): Raw data collection about AI election technologies
2. [Define](demo/2-define.md): Framework definition for analyzing impacts
3. [Distinguish](demo/3-distinguish.md): Differentiation between threats and safeguards
4. [Infer](demo/4-infer.md): Evidence-based projections with confidence levels
5. [Reflect](demo/5-reflect.md): Critical evaluation of the research process
6. [Synthesize](demo/6-synthesize.md): Integration of all components into final analysis

## 📊 Key Advantages

| Traditional AI | LogicPrimitives |
|-----------------|----------------|
| Black box reasoning | Traceable cognitive path |
| One-shot generation | Modular, verifiable steps |
| Phantom citations | Evidence-based conclusions |
| Uniform confidence | Explicit confidence levels |
| Invisible assumptions | Surfaced assumptions and biases |

## 💪 Use Cases

- **Academic Research**: Transparent methodology for literature review and analysis
- **Policy Analysis**: Evidence-based policy recommendations with clear reasoning
- **Strategic Planning**: Decision frameworks with explicit confidence assessment
- **Complex Problem Solving**: Decomposed approach to intractable challenges
- **Intelligence Analysis**: Structured reasoning with bias mitigation

## 🤝 Contributing

We welcome contributions to these concepts and documentation! See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get started.

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

## 📆 Changelog

### Version 2.0.0 (April 29, 2025)
- Added comprehensive technical implementation guide for MCP servers
- Added detailed research agents guide for building your own AI research team
- Updated documentation with more practical examples
- Improved conceptual framework with clearer integration points

### Version 1.0.0 (April 1, 2025)
- Initial release of conceptual framework
- Basic documentation of logic primitives
- Demo implementation using Markdown

## ⚠️ Research Disclaimer

The example research in this repository is demonstrative only and not synthesized from actual sources. In real research applications, tools like Brave Search, Perplexity, arXiv, and Alpha Vantage work in tandem to create sources from raw data.

The LogicPrimitives approach addresses not just the challenge of documenting sources, but documenting the chain of thought and second-order reasoning to move beyond black box reports.