# Flowise AgentFlow v2 Pattern Library

Complete collection of production-ready workflow patterns for Flowise AgentFlow v2.

📋 **[View Project Progress & Next Steps](PROJECT_PROGRESS.md)** - Development roadmap and session plan

---

## 📚 Pattern Index

### Core Patterns (1-6)

| Pattern | Name | Description | Repository |
|---------|------|-------------|------------|
| **#1** | **Chaining** | Sequential 3-agent chain with HIL approval gate | [afv2-pattern-01-chaining](https://github.com/snedea/afv2-pattern-01-chaining) |
| **#2** | **Parallel** | Multi-source concurrent execution with aggregation | [afv2-pattern-02-parallel](https://github.com/snedea/afv2-pattern-02-parallel) |
| **#3** | **Routing** | Intent-based routing to domain-specific agents | [afv2-pattern-03-routing](https://github.com/snedea/afv2-pattern-03-routing) |
| **#4** | **Iteration** | Quality-driven iterative refinement loop | [afv2-pattern-04-iteration](https://github.com/snedea/afv2-pattern-04-iteration) |
| **#5** | **Looping** | Validation-driven retry loop with fix generation | [afv2-pattern-05-looping](https://github.com/snedea/afv2-pattern-05-looping) |
| **#6** | **Hierarchy** | Supervisor-orchestrated task delegation | [afv2-pattern-06-hierarchy](https://github.com/snedea/afv2-pattern-06-hierarchy) |

### Advanced Patterns (7-9)

| Pattern | Name | Description | Repository |
|---------|------|-------------|------------|
| **#7** | **Batch Processing** | Iteration Node for-each array processing | [afv2-pattern-07-batch](https://github.com/snedea/afv2-pattern-07-batch) |
| **#8** | **Conditional Retry** | Score-based validation with retry loop | [afv2-pattern-08-retry](https://github.com/snedea/afv2-pattern-08-retry) |
| **#9** | **API Integration** | HTTP Request Node with error handling | [afv2-pattern-09-api](https://github.com/snedea/afv2-pattern-09-api) |

### Specialized Node Patterns (10-13)

| Pattern | Name | Description | Repository |
|---------|------|-------------|------------|
| **#10** | **RAG** | Retriever + LLM + Agent for document Q&A | [afv2-pattern-10-rag](https://github.com/snedea/afv2-pattern-10-rag) |
| **#11** | **Smart Calculator** | Tool + LLM + Custom Function routing | [afv2-pattern-11-calculator](https://github.com/snedea/afv2-pattern-11-calculator) |
| **#12** | **Document Q&A** | Retriever + LLM + Condition with confidence | [afv2-pattern-12-doc-qa](https://github.com/snedea/afv2-pattern-12-doc-qa) |
| **#13** | **Data Pipeline** | Custom Function ETL with validation | [afv2-pattern-13-pipeline](https://github.com/snedea/afv2-pattern-13-pipeline) |

---

## 🎯 Pattern Selection Guide

### By Use Case

**Sequential Processing**
- ✅ Pattern #1 (Chaining) - Linear workflow with approval gates
- ✅ Pattern #6 (Hierarchy) - Multi-step delegation with reviews

**Parallel Execution**
- ✅ Pattern #2 (Parallel) - Multi-source data gathering
- ✅ Pattern #7 (Batch Processing) - Array iteration

**Decision Making**
- ✅ Pattern #3 (Routing) - Intent classification
- ✅ Pattern #11 (Smart Calculator) - Cost optimization routing

**Quality Improvement**
- ✅ Pattern #4 (Iteration) - Progressive refinement
- ✅ Pattern #5 (Looping) - Test-driven fixes
- ✅ Pattern #8 (Conditional Retry) - Score-based validation

**External Integration**
- ✅ Pattern #9 (API Integration) - HTTP requests
- ✅ Pattern #13 (Data Pipeline) - ETL workflows

**Knowledge Retrieval**
- ✅ Pattern #10 (RAG) - Full RAG with citations
- ✅ Pattern #12 (Document Q&A) - Simple Q&A

### By Node Type

**Agent Nodes**
- All patterns use Agent nodes as primary building blocks

**Condition Nodes**
- Pattern #3 (Routing), #8 (Conditional Retry), #11 (Smart Calculator), #12 (Document Q&A)

**Iteration Nodes**
- Pattern #7 (Batch Processing)

**HTTP Request Nodes**
- Pattern #9 (API Integration)

**Retriever Nodes**
- Pattern #10 (RAG), #12 (Document Q&A)

**LLM Nodes**
- Pattern #10 (RAG), #11 (Smart Calculator)

**Custom Function Nodes**
- Pattern #11 (Smart Calculator), #13 (Data Pipeline)

**Human Input Nodes**
- Pattern #1 (Chaining)

**Direct Reply Nodes**
- All patterns terminate with Direct Reply

---

## 📦 What's Included in Each Pattern

Every pattern repository includes:

- **`[NN]-[name].json`** - Complete Flowise workflow (ready to import)
- **`README.md`** - Pattern overview, use cases, configuration
- **`TESTING.md`** - Comprehensive test cases with validation checklists
- **`INTEGRATION_GUIDE.md`** - Step-by-step setup instructions *(patterns 10-13)*

---

## 🚀 Quick Start

### 1. Choose a Pattern

Browse the index above and select a pattern that matches your use case.

### 2. Import into Flowise

1. Click the repository link for your chosen pattern
2. Download the JSON file (e.g., `01-chaining.json`)
3. Open Flowise UI → **Agentflows** → **Add New** → **Import Agentflow**
4. Upload the JSON file
5. Configure API keys (see pattern's README.md)

### 3. Test the Pattern

Follow the test cases in `TESTING.md` to validate the workflow.

### 4. Customize

Modify agents, add tools, adjust prompts to fit your specific needs.

---

## 📊 Pattern Statistics

- **Total Patterns**: 13
- **Total Nodes**: 100+ across all patterns
- **Node Type Coverage**: 14/14 (100%)
- **Lines of Code**: ~12,000 (JSON workflows)
- **Documentation**: ~20,000 lines (README + TESTING + GUIDES)
- **Test Cases**: 40+ comprehensive scenarios

---

## 🏗️ Pattern Architecture

### Flow Types

| Flow Type | Patterns | Description |
|-----------|----------|-------------|
| **Linear** | #1, #10, #12, #13 | Sequential A→B→C execution |
| **Parallel** | #2, #7 | Concurrent branch execution |
| **Conditional** | #3, #8, #11, #12 | Decision-based routing |
| **Loop** | #4, #5, #6, #8 | Iterative refinement |
| **Hybrid** | #6, #7, #9 | Combination of above |

### Complexity Levels

| Level | Patterns | Nodes | Use When |
|-------|----------|-------|----------|
| **Simple** | #1, #12 | 6-8 | Learning, prototyping |
| **Moderate** | #3, #7, #9, #10, #11, #13 | 8-10 | Production workflows |
| **Complex** | #2, #4, #5, #6, #8 | 10+ | Advanced orchestration |

---

## 🔧 Technical Details

### Flowise Compatibility

- **Flowise Version**: v2.0+
- **AgentFlow Version**: v2.2
- **Node API**: Latest Flowise node structure
- **Validation**: Passes `validate_workflow.py` (13 patterns)

### Model Configuration

All patterns use:
- **Model**: Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`)
- **Temperature**: 0.1-0.4 (varies by role)
- **Streaming**: Enabled
- **Memory**: Enabled for stateful agents

### State Management

Patterns demonstrate:
- Flow state variables
- Agent state updates
- Cross-agent data sharing
- Iteration counters
- Loop control flags

---

## 📖 Documentation

### Pattern Documentation

Each pattern includes detailed documentation:
- **Overview**: Flow structure and key features
- **Use Cases**: Real-world applications
- **Configuration**: API keys, tools, settings
- **Testing**: Manual test cases with expected results
- **Troubleshooting**: Common issues and solutions

### Additional Resources

- **Flowise Documentation**: https://docs.flowiseai.com/
- **AgentFlow v2 Docs**: https://docs.flowiseai.com/agentflows
- **Anthropic Claude**: https://docs.anthropic.com/

---

## 🤝 Contributing

These patterns are production-ready templates maintained by Context Foundry.

**Feedback & Issues**:
- Report issues in individual pattern repositories
- Suggest improvements via GitHub Issues
- Share use cases and customizations

---

## 📜 License

All patterns are open-source and free to use.

---

## 🏆 Built By

**Context Foundry** - Autonomous AI development system

- Patterns 1-6: Core workflow patterns
- Patterns 7-9: Advanced node demonstrations
- Patterns 10-13: Specialized node templates

All patterns built autonomously with comprehensive testing and documentation.

---

## 🔗 Links

- **GitHub Organization**: https://github.com/snedea
- **Pattern Index**: You are here
- **Context Foundry**: https://github.com/context-foundry/context-foundry

---

🤖 Built with Context Foundry
