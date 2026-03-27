---
name: appendix-ai-cli
description: This skill should be used when the user wants to use "AI CLI agent", "Claude Code", "Claude CLI", "Gemini CLI", "Aider", "GitHub Copilot CLI", "AI terminal agent", "AI coding assistant CLI", "Terminal-Bench", "AI command line coding tool", "agentic coding assistant", "AI pair programmer terminal", "large-scale refactoring with AI", "codebase understanding AI", "AI coding tool", "terminal AI assistant", "AI code generator CLI", "code with AI in terminal", "AI shell assistant", or compare AI-powered command-line coding tools. Also responds to Korean: "AI CLI 에이전트", "Claude Code 사용법", "Gemini CLI 사용법", "AI 터미널 에이전트", "AI 코딩 어시스턴트 CLI", "AI 명령줄 코딩 도구", "클로드 코드 사용법", "제미나이 CLI 사용법", "AI CLI 써보고 싶어", "터미널에서 AI 쓰기", "CLI로 코딩해줘", "AI가 코드 짜주는 CLI". Also responds to Japanese: "AI CLIエージェント", "Claude Codeの使い方", "Gemini CLIの使い方", "AIターミナルエージェント", "AIコーディングアシスタントCLI", "ターミナルでAIを使いたい", "CLIでコーディングしたい", "AIでコードを書いてほしい", "Aiderの使い方", "AI開発ツール" Also responds to Chinese: "AI命令行工具", "Claude Code使用方法", "Gemini CLI使用方法", "AI终端智能体", "AI编程助手CLI", "在终端使用AI", "用CLI编程", "让AI帮我写代码", "Aider使用方法", "AI开发工具".. Apply this skill to select, configure, and effectively use AI CLI agents for software development tasks.
version: 1.0.0
---

# Appendix E - AI Agents on the CLI

## Overview

**AI CLI Agents** represent a new class of tools that transform the developer's terminal from a command executor into an intelligent, collaborative workspace. These agents understand natural language, maintain context about entire codebases, and perform complex multi-step development tasks autonomously — from architectural refactoring to test generation to documentation.

The command-line interface is an ideal environment for AI agents: text-based, sandboxed, and structured. Unlike GUI agents that must interpret visual layouts, CLI agents work directly with code, files, and system commands, enabling precise and auditable automation.

**Core Principle:** Choose the CLI agent based on your workflow: Claude Code for architectural work, Gemini CLI for multimodal and cloud tasks, Aider for git-centric TDD, GitHub Copilot CLI for GitHub-integrated workflows.

## When This Skill Applies

Activate this pattern when:
- Performing large-scale refactoring across many files
- Generating comprehensive test suites for existing code
- Generating or updating technical documentation
- Implementing features described in natural language
- Integrating with CI/CD pipelines for automated code review
- Comparing AI CLI tools for a team's development workflow
- Setting up CLAUDE.md / GEMINI.md project memory files

## CLI Agent Comparison

| Agent | Provider | Core Strength | Best For |
|-------|----------|---------------|----------|
| **Claude Code** | Anthropic | Deep codebase understanding, architectural reasoning | Large-scale refactoring, multi-file changes, architectural tasks |
| **Gemini CLI** | Google | Open-source, multimodal, large context window, Google Cloud integration | Multimodal tasks, Google Cloud workflows, broad accessibility |
| **Aider** | Open Source | Git-centric, auto-commit, TDD workflow, model-agnostic | Test-driven development, precise bug fixes, auditable changes |
| **GitHub Copilot CLI** | GitHub/Microsoft | Deep GitHub ecosystem integration, issue→PR workflow | GitHub-integrated teams, automated issue resolution |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map CLI agent requirements:
1. What is the primary task type? (Refactoring, testing, documentation, feature implementation)
2. What is the scope? (Single file, module, or full codebase)
3. Is git commit history important? (Aider auto-commits; others require explicit commits)
4. Are there Google Cloud or GitHub-specific integrations needed?
5. Is multimodal input needed? (Design mockups → code: use Gemini CLI)

### PLAN
Select and configure the agent:
1. Choose the agent based on strength mapping above
2. Create a project memory file (CLAUDE.md or GEMINI.md) with codebase context
3. Define the task in clear, specific natural language with success criteria
4. Identify which files/directories are in scope
5. Plan validation: how will you verify the agent's output is correct?

### ACTION
Execute with the chosen CLI agent:
1. Navigate to the project directory
2. Invoke the agent with the task description
3. Review the agent's plan before it executes (Claude Code always explains first)
4. Approve or refine the plan
5. Validate output with tests, review, or both
6. Commit the changes with appropriate commit message

## Claude Code (Claude CLI)

### Overview
Anthropic's Claude CLI is engineered as a high-level coding agent with deep, holistic understanding of a project's architecture. Its defining feature is its "agentic" nature — it creates a mental model of your entire repository before acting, making it ideal for tasks with broad architectural impact.

**Key strengths:**
- Reads all relevant files to understand existing patterns before making changes
- Plans before executing — explains its approach and asks for confirmation
- MCP extensibility — connect custom tools (databases, APIs, scripts)
- Deep Git integration for branch and commit management

### Setup and Configuration
```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code

# Authenticate
claude auth login

# Navigate to your project
cd /path/to/your/project

# Launch Claude Code
claude
```

### CLAUDE.md — Project Memory File
```markdown
# Project: E-Commerce API

## Architecture
- FastAPI backend with PostgreSQL (SQLAlchemy ORM)
- Redis for session caching
- Celery for async task processing
- JWT authentication (no session cookies)

## Code Conventions
- All API endpoints use Pydantic v2 models for request/response
- Database models in `app/models/`, schemas in `app/schemas/`
- Tests use pytest with real test database (no mocks for DB layer)
- All functions must have type hints

## Key Files
- `app/main.py` — FastAPI app setup and middleware
- `app/core/security.py` — JWT auth logic
- `app/api/v1/` — All API endpoints

## Do Not Touch
- `migrations/` — Alembic migrations, only add new ones
- `.env.example` — Update when adding new env vars
```

### Example Use Cases
```bash
# Large-scale refactoring
claude "Our authentication uses session cookies. Refactor the entire codebase
to use stateless JWTs. Update login/logout endpoints, session middleware,
and frontend token handling."

# API integration
claude "We have an OpenAPI spec at docs/weather-api.yaml. Integrate this API:
create a service module, add a new /weather endpoint, update the dashboard."

# Documentation generation
claude "Analyze ./src/utils/data_processing.py. Generate comprehensive
TSDoc comments for every function explaining purpose, parameters, and return value."
```

## Gemini CLI

### Overview
Google's Gemini CLI is a versatile, open-source AI agent with a massive context window (up to 1M tokens), multimodal capabilities, and a transparent "Reason and Act" loop. It's accessible to hobbyists (generous free tier) and enterprise developers alike.

**Key strengths:**
- Open-source and free tier available
- Multimodal: process images, screenshots, design files alongside text
- Massive context window for entire codebase analysis
- Built-in Google Cloud integration (GKE, Cloud SQL, GCS)
- MCP servers for custom tool integration

### Setup and Configuration
```bash
# Install Gemini CLI
npm install -g @google/gemini-cli

# Authenticate with Google
gemini auth login

# Navigate to your project
cd /path/to/your/project

# Launch Gemini CLI
gemini
```

### GEMINI.md — Project Memory File
```markdown
# Project Context for Gemini

## Stack
- Java Spring Boot microservices
- PostgreSQL primary database, Redis cache
- Docker + Kubernetes deployment on GKE

## Development Guidelines
- All services use OpenAPI 3.0 specifications
- Tests: JUnit 5 + Mockito, minimum 80% coverage
- Logging: structured JSON via SLF4J + Logback

## Google Cloud Resources
- GKE Cluster: prod-cluster-1 (us-central1)
- Cloud SQL: postgres-prod (PostgreSQL 15)
- GCS Bucket: my-app-artifacts

## Key Directories
- `services/` — Individual microservices
- `infra/` — Terraform configs
- `k8s/` — Kubernetes manifests
```

### Example Use Cases
```bash
# Multimodal: image to code
gemini describe component.png
# "Write React HTML/CSS that looks exactly like this component. Make it responsive."

# Google Cloud management
gemini "Find all GKE clusters running Kubernetes < 1.28 in the production project.
Generate gcloud commands to upgrade them one by one."

# Enterprise tool integration via MCP
gemini "Use the get-employee-details --id=E90210 tool to fetch their name and team,
then populate the welcome_template.md with that information."

# Large-scale refactoring
gemini "Read all *.java files in src/main/java. Replace all org.apache.log4j imports
and Logger class with org.slf4j.Logger and LoggerFactory. Rewrite all .info(),
.debug(), .error() calls to use structured key-value pairs."
```

## Aider

### Overview
Aider is an open-source AI coding assistant that acts as a true pair programmer by working directly on files and committing changes to Git. Its defining feature is directness: it applies edits, runs tests to validate them, and automatically commits every successful change.

**Key strengths:**
- Model-agnostic: works with Claude, GPT-4, Gemini, and local models
- Git-centric: every successful change is committed with a descriptive message
- Test-aware: runs tests after each change to validate correctness
- Transparent: clear audit trail of all code modifications

### Setup and Configuration
```bash
# Install Aider
pip install aider-chat

# Configure with your preferred model
export ANTHROPIC_API_KEY="your-key"
# or
export OPENAI_API_KEY="your-key"

# Start Aider in your project
cd /path/to/project
aider --model claude-opus-4-6

# Add specific files to context
aider --model claude-opus-4-6 src/billing.py tests/test_billing.py
```

### Example Aider Workflow (TDD)
```bash
# Step 1: Write a failing test first
> Create a failing test for a function that calculates compound interest.
# Aider writes the test, runs it (fails), commits: "Add failing test for compound_interest"

# Step 2: Implement to make it pass
> Now write the compound_interest function to make the test pass.
# Aider implements, runs test (passes), commits: "Implement compound_interest function"

# Step 3: Bug fix with verification
> The calculate_total function in billing.py fails on leap years.
  Fix the bug and verify against the existing test suite.
# Aider reads the file, identifies the issue, fixes it, runs all tests, commits

# Step 4: Dependency update
> Our project uses requests 2.28. Update all Python files to use the
  new requests 2.32 API, fix deprecated function calls, update requirements.txt
# Aider scans all .py files, applies changes, runs tests, commits each change
```

## GitHub Copilot CLI

### Overview
GitHub Copilot CLI extends AI pair programming into the terminal with native, deep GitHub ecosystem integration. It understands the context of a project within GitHub — issues, PRs, CI status — enabling autonomous issue-to-PR workflows.

**Key strengths:**
- Full GitHub context: issues, PRs, repository structure, CI/CD status
- Automated issue resolution: reads a GitHub issue, writes code, opens a PR
- Repository-aware Q&A: answers questions about the codebase with file paths
- Shell command helper (`gh?`): generates precise shell commands

### Setup and Configuration
```bash
# Requires GitHub CLI and Copilot subscription
gh extension install github/gh-copilot

# Authenticate
gh auth login

# Use Copilot in the CLI
gh copilot suggest "find all files larger than 50MB and compress them"
gh copilot explain "git rebase -i HEAD~5"
```

### Example Use Cases
```bash
# Automated issue resolution
gh copilot "Fix issue #123: off-by-one error in pagination.
Check out a branch, write the fix, run tests, open a PR referencing the issue."

# Repository-aware Q&A
gh copilot "Where in this repository is the database connection logic defined?
What environment variables does it require?"

# Shell command generation
gh? "find all files larger than 50MB, compress them, and place in /archive"
# → find . -size +50M -exec gzip -9 {} \; -exec mv {}.gz /archive/ \;
```

## Terminal-Bench: Evaluating CLI Agents

**Terminal-Bench** is a benchmark framework for evaluating AI agent proficiency in CLI tasks. It identifies the terminal as optimal for AI evaluation due to its text-based, sandboxed nature.

- **Terminal-Bench-Core-v0**: 80 manually curated tasks spanning scientific workflows and data analysis
- **Terminus**: Minimalistic baseline agent as a standardized testbed
- Tasks can be added via containerization or direct connections
- Enables massively parallel evaluation across models

Use Terminal-Bench to compare CLI agents on your specific task types before selecting one for your team.

## Key Takeaways

- **Right tool for the task**: Claude Code for architecture, Gemini CLI for multimodal/cloud, Aider for TDD/git workflow, Copilot CLI for GitHub integration
- **Project memory files matter**: CLAUDE.md and GEMINI.md give agents critical context about your codebase — invest in writing them well
- **No single "best" CLI agent**: The ecosystem is diverse; each agent has specialized strengths
- **Validate with tests**: Always run your test suite after AI-generated changes — the agent may not catch all edge cases
- **MCP extends capabilities**: Both Claude Code and Gemini CLI support MCP servers — use them to connect internal APIs and databases
- **Aider for auditability**: When every change must be tracked with rationale, Aider's auto-commit workflow provides the cleanest audit trail

## Anti-Patterns to Avoid

- **Unlimited scope**: "Refactor the entire codebase" without file scope constraints can cause unintended changes — always specify scope
- **No validation step**: Agent-generated code must be validated with tests and code review — never push directly without verification
- **Ignoring project memory files**: Without CLAUDE.md or GEMINI.md, the agent lacks project context and makes generic choices that violate your conventions
- **Using GUI models for CLI tasks**: CLI agents are optimized for code and terminal tasks — general-purpose chatbots are not substitutes

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| AI CLI + specialist coding agent team | AI CLI + **Coding Agents** |
| Framework selection for CLI workflows | AI CLI + **Agentic Frameworks** |
| CLI agent with tool integration | AI CLI + **Tool Use** + **MCP** |
| Evaluated CLI agent performance | AI CLI + **Evaluation** |

## References

- Anthropic Claude Code: https://docs.anthropic.com/en/docs/claude-code/cli-reference
- Google Gemini CLI: https://github.com/google-gemini/gemini-cli
- Aider: https://aider.chat/
- GitHub Copilot CLI: https://docs.github.com/en/copilot/github-copilot-enterprise/copilot-cli
- Terminal Bench: https://www.tbench.ai/
