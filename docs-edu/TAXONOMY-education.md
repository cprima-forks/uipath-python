# UiPath Python SDK - Educational Taxonomy

Version: v0.1.0

## Overview

This taxonomy organizes learning paths for mastering the UiPath Python SDK, from foundational concepts to advanced agent development and deployment.

## Learning Levels

**L1**: Beginner - No prior UiPath or Python SDK experience
**L2**: Intermediate - Basic Python, some automation experience
**L3**: Advanced - Python proficiency, automation concepts
**L4**: Expert - Advanced agent development and optimization

## 1. Foundations (L1)

### 1.1 Python Prerequisites

#### 1.1.1 Core Python
**Estimated Time**: 8-12 hours
**Prerequisites**: None

- Variables and data types
- Functions and modules
- Error handling (try/except)
- File I/O operations
- Virtual environments
- Package management with `pip` and `uv`

#### 1.1.2 Async Programming
**Estimated Time**: 4-6 hours
**Prerequisites**: Core Python

- Understanding async/await
- Coroutines and futures
- Event loops
- Concurrent execution patterns
- Common pitfalls

#### 1.1.3 Environment Management
**Estimated Time**: 2-3 hours
**Prerequisites**: Core Python

- `.env` files and environment variables
- Configuration management
- Secret handling best practices
- `python-dotenv` usage

### 1.2 UiPath Platform Concepts

#### 1.2.1 UiPath Orchestrator
**Estimated Time**: 4-6 hours
**Prerequisites**: None

- Orchestrator architecture
- Tenants and folders
- Authentication and authorization
- Cloud vs. on-premises

#### 1.2.2 Core Services Overview
**Estimated Time**: 3-4 hours
**Prerequisites**: UiPath Orchestrator

- Processes and jobs
- Assets (variables, credentials)
- Queues and transactions
- Action Center (human-in-the-loop)

## 2. SDK Fundamentals (L2)

### 2.1 Installation and Setup

#### 2.1.1 SDK Installation
**Estimated Time**: 1-2 hours
**Prerequisites**: Python Prerequisites

- Installing with `pip install uipath`
- Installing with `uv add uipath`
- Dependency management
- Version compatibility

#### 2.1.2 Authentication Setup
**Estimated Time**: 2-3 hours
**Prerequisites**: UiPath Orchestrator, SDK Installation

- Using `uipath auth` command
- Manual `.env` configuration
- OAuth token flow
- Token refresh mechanisms

### 2.2 Core Services

#### 2.2.1 Process Management
**Estimated Time**: 3-4 hours
**Prerequisites**: Authentication Setup

**Learning Objectives**:
- Invoke processes programmatically
- Pass input arguments
- Retrieve job results
- Monitor job status

**Key Concepts**:
```python
sdk = UiPath()
job = sdk.processes.invoke(
    name="MyProcess",
    input_arguments={"param": "value"}
)
```

**Practice Projects**:
1. Process invoker script
2. Batch process executor
3. Job status monitor

#### 2.2.2 Asset Management
**Estimated Time**: 3-4 hours
**Prerequisites**: Authentication Setup

**Learning Objectives**:
- Retrieve assets
- Update asset values
- Handle credentials securely
- Asset CRUD operations

**Key Concepts**:
```python
asset = sdk.assets.retrieve(name="MyAsset")
sdk.assets.update(name="MyAsset", value="new_value")
```

**Practice Projects**:
1. Configuration manager using assets
2. Dynamic credential rotator
3. Asset backup utility

#### 2.2.3 Queue Operations
**Estimated Time**: 4-5 hours
**Prerequisites**: Process Management

**Learning Objectives**:
- Add queue items
- Retrieve queue items
- Update transaction status
- Handle queue priorities

**Key Concepts**:
- Queue transaction lifecycle
- Bulk operations
- Error handling in queues

**Practice Projects**:
1. Queue item bulk loader
2. Transaction processor
3. Failed item reprocessor

#### 2.2.4 Storage (Buckets)
**Estimated Time**: 3-4 hours
**Prerequisites**: Authentication Setup

**Learning Objectives**:
- Upload files to buckets
- Download files from buckets
- Manage blob storage
- Handle large files

**Key Concepts**:
```python
sdk.buckets.upload(
    bucket_key="my-bucket",
    blob_file_path="remote/path/file.xlsx",
    source_path="local/file.xlsx"
)
```

**Practice Projects**:
1. File synchronization utility
2. Backup automation
3. Report archiver

### 2.3 CLI Workflows

#### 2.3.1 Project Initialization
**Estimated Time**: 2-3 hours
**Prerequisites**: SDK Installation

**Learning Objectives**:
- Use `uipath init` command
- Understand `uipath.json` structure
- Configure project metadata
- Set up `pyproject.toml`

**Key Concepts**:
- Project structure requirements
- Entrypoint specification
- Metadata best practices

#### 2.3.2 Local Development and Debugging
**Estimated Time**: 3-4 hours
**Prerequisites**: Project Initialization

**Learning Objectives**:
- Use `uipath run` for local testing
- Pass input arguments
- Debug agent behavior
- Handle local errors

**Practice Projects**:
1. Debug workflow for sample agent
2. Input validation script
3. Local test suite

#### 2.3.3 Packaging and Publishing
**Estimated Time**: 4-5 hours
**Prerequisites**: Local Development

**Learning Objectives**:
- Use `uipath pack` command
- Understand `.nupkg` format
- Use `uipath publish` command
- Manage package versions

**Key Concepts**:
- Dependency bundling
- Package validation
- Publishing to Orchestrator
- Version management

**Practice Projects**:
1. Automated packaging script
2. Multi-environment deployer
3. Version control integration

## 3. Advanced Services (L3)

### 3.1 Context Grounding (RAG)

#### 3.1.1 Knowledge Base Management
**Estimated Time**: 5-6 hours
**Prerequisites**: SDK Fundamentals

**Learning Objectives**:
- Create knowledge indexes
- Upload documents
- Manage embeddings
- Configure semantic search

**Key Concepts**:
```python
sdk.context_grounding.create_index(
    name="my-kb",
    description="Knowledge base for invoice processing"
)
```

#### 3.1.2 Semantic Search
**Estimated Time**: 4-5 hours
**Prerequisites**: Knowledge Base Management

**Learning Objectives**:
- Perform semantic searches
- Configure result ranking
- Handle search filters
- Optimize query performance

**Key Concepts**:
```python
results = sdk.context_grounding.search(
    name="my-kb",
    query="How to process invoices?",
    number_of_results=5
)
```

**Practice Projects**:
1. Document Q&A system
2. Contextual automation assistant
3. Knowledge base explorer

### 3.2 Action Center Integration

#### 3.2.1 Human-in-the-Loop Workflows
**Estimated Time**: 5-6 hours
**Prerequisites**: Process Management

**Learning Objectives**:
- Create action items
- Assign tasks to users
- Retrieve action status
- Handle action results

**Key Concepts**:
- Action lifecycle
- Form definitions
- Task assignment rules

**Practice Projects**:
1. Approval workflow system
2. Task assignment automator
3. Action dashboard

## 4. Agent Development (L3-L4)

### 4.1 LangChain Agents

#### 4.1.1 LangChain Fundamentals
**Estimated Time**: 8-10 hours
**Prerequisites**: SDK Fundamentals, Async Programming

**Learning Objectives**:
- Understand LangChain architecture
- Build chains and runnables
- Use chat models
- Implement memory

**Key Concepts**:
- LCEL (LangChain Expression Language)
- Runnable interface
- Chain composition
- Memory types (buffer, summary, etc.)

**Practice Projects**:
1. Simple chatbot
2. Chain-of-thought agent
3. Multi-step reasoning agent

#### 4.1.2 LangGraph Agents
**Estimated Time**: 10-12 hours
**Prerequisites**: LangChain Fundamentals

**Learning Objectives**:
- Build state graphs
- Define nodes and edges
- Implement conditional routing
- Handle agent loops

**Key Concepts**:
```python
from langgraph.graph import StateGraph

graph = StateGraph(AgentState)
graph.add_node("agent", agent_node)
graph.add_node("tools", tool_node)
graph.add_conditional_edges("agent", should_continue)
```

**Practice Projects**:
1. Multi-tool agent
2. Research assistant agent
3. Task orchestration agent

#### 4.1.3 UiPath Tool Integration
**Estimated Time**: 6-8 hours
**Prerequisites**: LangGraph Agents

**Learning Objectives**:
- Wrap UiPath services as LangChain tools
- Handle tool errors
- Implement tool validation
- Optimize tool calls

**Key Concepts**:
- Tool schema definition
- Async tool execution
- Error handling in tools

**Practice Projects**:
1. Asset management tool
2. Process invocation tool
3. Queue operations tool
4. Context grounding tool

### 4.2 LlamaIndex Agents

#### 4.2.1 LlamaIndex Fundamentals
**Estimated Time**: 8-10 hours
**Prerequisites**: SDK Fundamentals

**Learning Objectives**:
- Build indexes
- Create query engines
- Implement retrievers
- Use response synthesizers

**Key Concepts**:
- Document loaders
- Node parsers
- Index types (vector, tree, list)
- Query modes

**Practice Projects**:
1. Document Q&A system
2. Multi-document comparison tool
3. RAG agent

#### 4.2.2 Agent Workflows
**Estimated Time**: 6-8 hours
**Prerequisites**: LlamaIndex Fundamentals

**Learning Objectives**:
- Build agent workflows
- Implement tool calling
- Handle agent loops
- Optimize retrieval

**Practice Projects**:
1. Research agent
2. Data analysis agent
3. Report generation agent

### 4.3 Agent Packaging and Deployment

#### 4.3.1 Agent Configuration
**Estimated Time**: 4-5 hours
**Prerequisites**: Agent Development (LangChain or LlamaIndex)

**Learning Objectives**:
- Configure `uipath.json` for agents
- Define agent metadata
- Specify dependencies
- Configure bindings

**Key Concepts**:
```json
{
  "name": "MyAgent",
  "entrypoint": "main.py",
  "bindings": {
    "version": "2.0",
    "resources": [...]
  }
}
```

#### 4.3.2 Dependency Management
**Estimated Time**: 3-4 hours
**Prerequisites**: Agent Configuration

**Learning Objectives**:
- Manage Python dependencies
- Use `uv` for fast installs
- Create `uv.lock` files
- Handle dependency conflicts

**Key Concepts**:
- Lock file generation
- Dependency resolution
- Cross-platform compatibility

#### 4.3.3 Testing and Validation
**Estimated Time**: 5-6 hours
**Prerequisites**: Agent Packaging

**Learning Objectives**:
- Write unit tests for agents
- Use `uipath run` for local testing
- Validate agent behavior
- Handle edge cases

**Practice Projects**:
1. Test suite for sample agent
2. Input validation framework
3. Mock service testing

## 5. Evaluation and Optimization (L4)

### 5.1 Evaluation Framework

#### 5.1.1 Eval Set Creation
**Estimated Time**: 4-5 hours
**Prerequisites**: Agent Development

**Learning Objectives**:
- Create eval sets (JSON)
- Define test cases
- Write expected outputs
- Organize evaluations

**Key Concepts**:
```json
{
  "version": "1.0",
  "test_cases": [
    {
      "input": "...",
      "expected_output": "...",
      "metadata": {}
    }
  ]
}
```

**Practice Projects**:
1. Eval set for calculator agent
2. Multi-turn conversation eval
3. Tool usage eval set

#### 5.1.2 Evaluator Configuration
**Estimated Time**: 6-8 hours
**Prerequisites**: Eval Set Creation

**Learning Objectives**:
- Configure deterministic evaluators
- Use LLM-as-judge evaluators
- Implement custom evaluators
- Combine multiple evaluators

**Key Concepts**:
- Exact match evaluator
- JSON similarity evaluator
- Semantic similarity (LLM judge)
- Tool call evaluators

**Practice Projects**:
1. Multi-evaluator pipeline
2. Custom evaluator implementation
3. Evaluation dashboard

#### 5.1.3 Running Evaluations
**Estimated Time**: 3-4 hours
**Prerequisites**: Evaluator Configuration

**Learning Objectives**:
- Run evals with CLI
- Interpret eval results
- Analyze failure patterns
- Generate reports

**Practice Projects**:
1. Automated eval runner
2. Result analysis script
3. Regression testing suite

### 5.2 Agent Optimization

#### 5.2.1 Prompt Engineering
**Estimated Time**: 6-8 hours
**Prerequisites**: Evaluation Framework

**Learning Objectives**:
- Optimize agent prompts
- Use few-shot examples
- Handle edge cases
- Reduce token usage

**Key Concepts**:
- Prompt templates
- System message optimization
- Context management

#### 5.2.2 Performance Optimization
**Estimated Time**: 5-6 hours
**Prerequisites**: Agent Development

**Learning Objectives**:
- Optimize tool calls
- Reduce latency
- Handle concurrent operations
- Cache results

**Key Concepts**:
- Async optimization
- Connection pooling
- Retry strategies

#### 5.2.3 Cost Optimization
**Estimated Time**: 4-5 hours
**Prerequisites**: Performance Optimization

**Learning Objectives**:
- Reduce LLM costs
- Optimize token usage
- Use cheaper models where appropriate
- Cache responses

**Practice Projects**:
1. Cost tracking dashboard
2. Token usage analyzer
3. Model selection optimizer

## 6. Production Deployment (L4)

### 6.1 Deployment Strategies

#### 6.1.1 Environment Management
**Estimated Time**: 4-5 hours
**Prerequisites**: Packaging and Publishing

**Learning Objectives**:
- Manage dev/staging/prod environments
- Configure environment-specific settings
- Handle secrets securely
- Implement rollback strategies

#### 6.1.2 Monitoring and Logging
**Estimated Time**: 5-6 hours
**Prerequisites**: Production Deployment

**Learning Objectives**:
- Implement agent logging
- Monitor agent performance
- Track errors and exceptions
- Use UiPath tracing

**Key Concepts**:
- Telemetry integration
- Log aggregation
- Alert configuration

### 6.2 Continuous Integration/Deployment

#### 6.2.1 CI/CD Pipelines
**Estimated Time**: 6-8 hours
**Prerequisites**: Deployment Strategies

**Learning Objectives**:
- Set up GitHub Actions
- Automate testing
- Automate packaging
- Automate publishing

**Practice Projects**:
1. GitHub Actions workflow for agent
2. Automated testing pipeline
3. Multi-environment deployer

## 7. Advanced Topics (L4)

### 7.1 Custom Evaluators
**Estimated Time**: 8-10 hours
**Prerequisites**: Evaluation Framework

**Learning Objectives**:
- Implement custom evaluator logic
- Extend base evaluator classes
- Define custom metrics
- Integrate with eval framework

### 7.2 Multi-Agent Systems
**Estimated Time**: 10-12 hours
**Prerequisites**: LangGraph Agents

**Learning Objectives**:
- Coordinate multiple agents
- Implement agent communication
- Design agent hierarchies
- Handle agent failures

### 7.3 Event-Driven Automation
**Estimated Time**: 6-8 hours
**Prerequisites**: SDK Fundamentals

**Learning Objectives**:
- Subscribe to UiPath events
- Trigger automations on events
- Handle event queues
- Implement event-driven architectures

## Learning Pathways

### Pathway A: CLI User (Quick Start)
**Total Time**: 20-25 hours
**Target**: Developers who want to package and deploy agents quickly

1. Python Prerequisites (1.1.1, 1.1.3)
2. SDK Installation (2.1.1)
3. Authentication Setup (2.1.2)
4. CLI Workflows (2.3)
5. Agent Packaging and Deployment (4.3)

### Pathway B: SDK Developer
**Total Time**: 45-55 hours
**Target**: Developers building automation with UiPath services

1. Foundations (1)
2. SDK Fundamentals (2.1, 2.2)
3. Advanced Services (3)
4. Production Deployment (6)

### Pathway C: Agent Developer (LangChain)
**Total Time**: 70-85 hours
**Target**: Developers building LangChain/LangGraph agents

1. Foundations (1)
2. SDK Fundamentals (2)
3. Advanced Services (3.1)
4. LangChain Agents (4.1)
5. Agent Packaging and Deployment (4.3)
6. Evaluation and Optimization (5)
7. Production Deployment (6)

### Pathway D: Agent Developer (LlamaIndex)
**Total Time**: 65-80 hours
**Target**: Developers building LlamaIndex agents

1. Foundations (1)
2. SDK Fundamentals (2)
3. Advanced Services (3.1)
4. LlamaIndex Agents (4.2)
5. Agent Packaging and Deployment (4.3)
6. Evaluation and Optimization (5)
7. Production Deployment (6)

### Pathway E: Full Stack (Expert)
**Total Time**: 120-150 hours
**Target**: Developers mastering the entire SDK and agent ecosystem

1. All sections (1-7)

## Assessment and Certification

### Knowledge Checks
Each section includes:
- Quiz questions
- Hands-on exercises
- Code review tasks

### Capstone Projects

#### Project 1: Automation Toolkit (L2-L3)
Build a command-line toolkit that:
- Manages assets across environments
- Invokes processes with validation
- Monitors job execution
- Generates reports

#### Project 2: Document Processing Agent (L3-L4)
Build an agent that:
- Processes invoices from storage buckets
- Uses context grounding for classification
- Routes to appropriate queues
- Sends exceptions to Action Center

#### Project 3: Multi-Agent Research System (L4)
Build a multi-agent system that:
- Coordinates research across multiple sources
- Uses UiPath services for data retrieval
- Implements comprehensive evaluation
- Deploys with CI/CD pipeline

## Resources and References

### Official Documentation
- [UiPath Python SDK Docs](https://uipath.github.io/uipath-python/)
- [LangChain Documentation](https://python.langchain.com/)
- [LlamaIndex Documentation](https://docs.llamaindex.ai/)

### Sample Projects
- `samples/calculator`: Basic agent with evals
- `samples/weather_tools`: Multi-tool agent
- `samples/asset-modifier-agent`: Asset management
- `samples/google-ADK-agent`: Google ADK integration

### Community
- GitHub Issues: Report bugs and request features
- GitHub Discussions: Ask questions and share knowledge

## Taxonomy Metadata

**Version**: v0.1.0
**Last Updated**: 2025-11-04
**Target Audience**: Python developers, RPA developers, AI/ML engineers
**Prerequisites**: Basic Python knowledge recommended
**Estimated Total Learning Time**: 20-150 hours (depending on pathway)
**Skill Levels**: L1 (Beginner) → L4 (Expert)
