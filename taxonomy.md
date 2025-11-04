# UiPath Python SDK - Content Taxonomy

Version: v0.1.0

## 1. Core Architecture

### 1.1 SDK Services
**Location**: `src/uipath/_services/`

#### 1.1.1 Process Management
- Process invocation and execution
- Job lifecycle management
- Input/output argument handling

#### 1.1.2 Asset Management
- Asset CRUD operations
- Credential handling
- Variable storage and retrieval

#### 1.1.3 Storage Services
- **Buckets**: Cloud storage containers
  - File upload/download
  - Blob management
  - Path operations

#### 1.1.4 Queue Services
- Transaction queue management
- Queue item processing
- Status tracking

#### 1.1.5 Context Grounding
- Semantic search
- Knowledge base management
- Vector embeddings
- RAG (Retrieval-Augmented Generation) support

#### 1.1.6 Action Center
- Human-in-the-loop workflows
- Task assignment and tracking
- Form handling

#### 1.1.7 Connections
- External system integrations
- Connection management
- Authentication handling

### 1.2 CLI Components
**Location**: `src/uipath/_cli/`

#### 1.2.1 Project Lifecycle
- `init`: Project initialization
- `pack`: Package creation (.nupkg)
- `publish`: Deployment to Orchestrator
- `run`: Local debugging and execution

#### 1.2.2 Authentication
- `auth`: OAuth flow handling
- Environment variable management
- Token storage and refresh

#### 1.2.3 Evaluation System
**Location**: `src/uipath/eval/`

##### 1.2.3.1 Evaluator Types
- **Deterministic Evaluators**
  - Exact match
  - Contains evaluator
  - JSON similarity
  - Output evaluator

- **LLM-as-Judge Evaluators**
  - Semantic similarity
  - Trajectory evaluation
  - Output evaluation with LLM

- **Tool-Specific Evaluators**
  - Tool call count
  - Tool call arguments
  - Tool call output
  - Tool call order

##### 1.2.3.2 Evaluation Workflow
- Eval set management
- Test case execution
- Metric calculation and reporting

### 1.3 Agent Framework
**Location**: `src/uipath/agent/`

#### 1.3.1 Agent Models
- Agent configuration and metadata
- LangChain integration
- LlamaIndex integration

#### 1.3.2 Agent Packaging
- Dependency management
- Entrypoint handling
- Configuration validation

## 2. Integration Layers

### 2.1 LangChain Integration
**Related Repository**: `uipath-langchain-python`

#### 2.1.1 Components
- Custom LangChain runnables
- Chain composition
- Memory integration
- Tool wrapping

#### 2.1.2 LangGraph Support
- State graph agents
- Node definition
- Edge configuration
- Conditional routing

### 2.2 LlamaIndex Integration
**Related Repository**: `uipath-llamaindex-python`

#### 2.2.1 Components
- Query engines
- Index management
- Retrieval configuration
- Response synthesis

## 3. Development Infrastructure

### 3.1 Testing Framework
**Location**: `tests/`

#### 3.1.1 Test Categories
- **Unit Tests**: `tests/cli/unit/`, `tests/sdk/`
  - Service method testing
  - CLI command testing
  - Model validation

- **Integration Tests**: `tests/cli/integration/`
  - End-to-end workflows
  - Service interaction testing
  - Authentication flows

- **Contract Tests**: `tests/cli/contract/`
  - API contract validation
  - Schema verification

#### 3.1.2 Test Fixtures
- Mock data structures
- Sample projects
- Test evaluators

### 3.2 Documentation
**Location**: `docs/`

#### 3.2.1 User Documentation
- Getting started guides
- API reference
- CLI command reference
- Examples and tutorials

#### 3.2.2 Developer Documentation
- Contributing guidelines
- Release policy
- Architecture decisions
- FAQ

#### 3.2.3 Sample Projects
**Location**: `samples/`

- **calculator**: Basic agent with evaluation examples
- **weather_tools**: Multi-tool agent example
- **asset-modifier-agent**: Asset manipulation example
- **google-ADK-agent**: Google Agent Development Kit integration
- **event-trigger**: Event-driven automation example

## 4. Configuration and Metadata

### 4.1 Project Configuration
- `pyproject.toml`: Python project metadata, dependencies, build configuration
- `uipath.json`: UiPath-specific project configuration
  - Entrypoint definition
  - Binding specifications
  - Resource declarations

### 4.2 Evaluation Configuration
**Format**: JSON

#### 4.2.1 Eval Set Structure
```json
{
  "version": "1.0",
  "name": "Eval Set Name",
  "test_cases": [...]
}
```

#### 4.2.2 Evaluator Structure
```json
{
  "version": "1.0",
  "name": "Evaluator Name",
  "type": "evaluator_type",
  "config": {...}
}
```

## 5. Data Models

### 5.1 Agent Models
**Location**: `src/uipath/models/`

- Agent metadata
- Tool definitions
- Input/output schemas
- Evaluation results

### 5.2 Service Models
- API request/response models
- Resource representations
- Error models

## 6. Utilities

### 6.1 Core Utilities
**Location**: `src/uipath/_utils/`

- File operations
- JSON handling
- Path management
- Validation helpers

### 6.2 CLI Utilities
**Location**: `src/uipath/_cli/`

- Command parsing
- Output formatting
- Progress tracking
- Error handling

## 7. Telemetry and Tracing

### 7.1 Telemetry
**Location**: `src/uipath/telemetry/`

- Usage tracking
- Error reporting
- Performance metrics

### 7.2 Tracing
**Location**: `src/uipath/tracing/`

- Execution tracing
- Debug logging
- Span management

## 8. CI/CD and Automation

### 8.1 GitHub Workflows
**Location**: `.github/workflows/`

- Test execution pipelines
- Package publishing
- Code quality checks

### 8.2 Development Scripts
**Location**: `scripts/`

- Automation helpers
- Build tools
- Maintenance scripts

## 9. Claude Code Integration

### 9.1 Agents
**Location**: `.claude/agents/`

- `command-tester.md`: CLI command testing automation

### 9.2 Commands
**Location**: `.claude/commands/`

- Custom slash commands for development workflows

## 10. Cross-Cutting Concerns

### 10.1 Error Handling
- Hierarchical error types
- Error context preservation
- User-friendly error messages

### 10.2 Authentication & Authorization
- OAuth 2.0 flow
- Token management
- Environment-based configuration

### 10.3 Async/Await Patterns
- Asynchronous service calls
- Concurrent operations
- Event loop management

### 10.4 Dependency Management
- `uv` for fast dependency resolution
- Lock file management (`uv.lock`)
- Version pinning strategies

## Taxonomy Metadata

**Version**: v0.1.0
**Last Updated**: 2025-11-04
**Repository**: https://github.com/UiPath/uipath-python
**Python Versions**: 3.10, 3.11, 3.12, 3.13
**License**: See repository LICENSE file
