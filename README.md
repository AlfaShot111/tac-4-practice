# Natural Language SQL Interface

A web application that converts natural language queries to SQL using AI, built with FastAPI and Vite + TypeScript.

## Features

- 🗣️ Natural language to SQL conversion using OpenAI or Anthropic
- 📁 Drag-and-drop file upload (.csv and .json)
- 📊 Interactive table results display
- 🔒 SQL injection protection
- ⚡ Fast development with Vite and uv

## Prerequisites

- Python 3.10+
- Node.js 18+
- OpenAI API key and/or Anthropic API key
- 'gh' github cli
- astral uv

## Setup

### 1. Install Dependencies

```bash
# Backend
cd app/server
uv sync --all-extras

# Frontend
cd app/client
npm install
```

### 2. Environment Configuration

Set up your API keys in the server directory:

```bash
cp .env.sample .env
```

and

```bash
cd app/server
cp .env.sample .env
# Edit .env and add your API keys
```

## Quick Start

Use the provided script to start both services:

```bash
./scripts/start.sh
```

Press `Ctrl+C` to stop both services.

The script will:
- Check that `.env` exists in `app/server/`
- Start the backend on http://localhost:8000
- Start the frontend on http://localhost:5173
- Handle graceful shutdown when you exit

## Manual Start (Alternative)

### Backend
```bash
cd app/server
# .env is loaded automatically by python-dotenv
uv run python server.py
```

### Frontend
```bash
cd app/client
npm run dev
```

## Usage

1. **Upload Data**: Click "Upload Data" to open the modal
   - Use sample data buttons for quick testing
   - Or drag and drop your own .csv or .json files
   - Uploading a file with the same name will overwrite the existing table
2. **Query Your Data**: Type a natural language query like "Show me all users who signed up last week"
   - Press `Cmd+Enter` (Mac) or `Ctrl+Enter` (Windows/Linux) to run the query
3. **View Results**: See the generated SQL and results in a table format
4. **Manage Tables**: Click the × button on any table to remove it

## Development

### Backend Commands
```bash
cd app/server
uv run python server.py      # Start server with hot reload
uv run pytest               # Run tests
uv add <package>            # Add package to project
uv remove <package>         # Remove package from project
uv sync --all-extras        # Sync all extras
```

### Frontend Commands
```bash
cd app/client
npm run dev                 # Start dev server
npm run build              # Build for production
npm run preview            # Preview production build
```

## Project Structure

```
.
├── app/                    # Main application
│   ├── client/             # Vite + TypeScript frontend
│   └── server/             # FastAPI backend
│
├── adws/                   # AI Developer Workflows - Core agent system
├── scripts/                # Utility scripts (start.sh, stop_apps.sh)
├── specs/                  # Feature specifications
├── ai_docs/                # AI/LLM documentation
├── agents/                 # Agent execution logging
└── logs/                   # Structured session logs
```

## AI Developer Workflow (ADW) System

The ADW system automates the entire software development lifecycle by integrating GitHub issues with Claude Code CLI. It automatically detects issues, classifies them by type, generates implementation plans, executes the implementation, and creates pull requests—all without manual intervention.

### What ADW Does

ADW transforms GitHub issues into production-ready code through an intelligent, multi-stage workflow:

1. **Automatic Issue Detection** - Monitors GitHub for new issues or "adw" comment triggers
2. **Intelligent Classification** - Uses Claude Code to classify issues as /chore, /bug, or /feature
3. **Plan Generation** - Creates detailed implementation plans using the sdlc_planner agent
4. **Automated Implementation** - Executes plans using the sdlc_implementor agent
5. **Git Integration** - Creates semantic commits and pull requests with full context

### Value Proposition

- **Zero Manual Overhead** - Issues are processed automatically from detection to PR creation
- **Consistent Quality** - AI-driven planning ensures thorough implementation with proper testing
- **Full Traceability** - Every workflow run has a unique ADW ID for tracking and debugging
- **Flexible Triggers** - Choose manual, cron polling, or real-time webhook processing

### Trigger Modes

The ADW system supports three trigger modes to fit different workflows:

1. **Manual Mode** (`adw_plan_build.py`) - Process a specific issue on-demand
   ```bash
   uv run adws/adw_plan_build.py 123
   ```

2. **Cron Mode** (`trigger_cron.py`) - Continuous polling every 20 seconds
   ```bash
   uv run adws/trigger_cron.py
   ```
   - Detects new issues without comments
   - Detects issues with "adw" comment trigger

3. **Webhook Mode** (`trigger_webhook.py`) - Real-time GitHub event processing
   ```bash
   uv run adws/trigger_webhook.py
   ```
   - Responds immediately to issue creation
   - Processes "adw" comments in real-time
   - Meets GitHub's 10-second timeout requirement

### ADW Tracking System

Each workflow run is assigned a unique 8-character ADW ID (e.g., `a1b2c3d4`) that appears in:
- Issue comments: `a1b2c3d4_ops: ✅ Starting ADW workflow`
- Branch names: `feat-456-a1b2c3d4-add-feature`
- Commit messages: `Generated with ADW ID: a1b2c3d4`
- Output directories: `agents/a1b2c3d4/`
- Pull request descriptions

This ID enables complete traceability from issue to deployed code.

### ADW System Architecture

The ADW system consists of several core components that work together:

#### Core Modules

- **agent.py** - Claude Code CLI Integration
  - Executes Claude Code with prompts via subprocess
  - Parses JSONL output from agent runs
  - Manages execution environment and output directories
  - Validates Claude Code CLI installation
  - Stores prompts and raw outputs for debugging

- **github.py** - GitHub API Operations
  - Fetches issue details using gh CLI
  - Posts comments to issues
  - Manages labels and assignees
  - Lists open issues for monitoring
  - Extracts repository information

- **data_types.py** - Type-Safe Data Models
  - GitHub API response models (GitHubIssue, GitHubComment, GitHubUser)
  - Agent request/response models (AgentPromptRequest, AgentTemplateRequest)
  - Slash command types (IssueClassSlashCommand)
  - Claude Code result structures
  - Ensures type safety throughout the system

- **utils.py** - System Utilities
  - `make_adw_id()` - Generates unique 8-character workflow IDs
  - `setup_logger()` - Configures logging to console and file
  - Provides consistent logging infrastructure

#### Workflow Scripts

- **adw_plan_build.py** - Main Workflow Orchestrator
  - Entry point for processing a single GitHub issue
  - Orchestrates the complete plan-and-build workflow
  - Manages agent execution sequence:
    1. Classifier agent (determines issue type)
    2. Planner agent (generates implementation plan)
    3. Implementor agent (executes the plan)
  - Handles git operations (branching, committing)
  - Creates pull requests with full context

#### Triggers

- **trigger_cron.py** - Automated Monitoring Trigger
  - Polls GitHub every 20 seconds for qualifying issues
  - Detects new issues without comments
  - Detects issues with "adw" comment trigger
  - Launches adw_plan_build.py in background for each issue
  - Suitable for deployment as systemd service

- **trigger_webhook.py** - GitHub Webhook Server
  - FastAPI server for real-time GitHub event processing
  - Listens on port 8001 (configurable via PORT env var)
  - Endpoints:
    - `/gh-webhook` - Receives GitHub issue and comment events
    - `/health` - Health check endpoint
  - Responds immediately to meet GitHub's 10-second timeout
  - Launches workflows in background with unique ADW IDs
  - Requires public URL (ngrok, tunnel, or production server)

#### Utilities

- **health_check.py** - System Health Validation
  - Validates environment variables (ANTHROPIC_API_KEY, CLAUDE_CODE_PATH, etc.)
  - Checks git repository configuration
  - Tests Claude Code CLI functionality
  - Tests GitHub CLI authentication
  - Returns structured health check results
  - Run before deployment to verify setup

### Agent Execution Model

The ADW system uses a three-agent pipeline:

1. **Classifier Agent** (via `.claude/commands/classifier.md`)
   - Analyzes issue title and description
   - Returns `/chore`, `/bug`, or `/feature` classification
   - Fast execution using haiku model

2. **Planner Agent** (via `.claude/commands/plan.md`)
   - Receives classified issue and type
   - Generates detailed implementation plan
   - Creates spec file in `specs/` directory
   - Includes step-by-step tasks and validation commands
   - Uses sonnet model for planning quality

3. **Implementor Agent** (via `.claude/commands/implement.md`)
   - Reads the generated plan file
   - Implements all steps in the plan
   - Runs validation commands
   - Creates commits with proper formatting
   - Uses sonnet or opus model depending on complexity

### ADW Workflow Process

Each workflow run progresses through these stages:

#### 1. Trigger Detection
- **Cron**: Polls GitHub API every 20 seconds for new issues or "adw" comments
- **Webhook**: Receives real-time GitHub events (issues.opened, issue_comment.created)
- **Manual**: Direct invocation with issue number

#### 2. Issue Classification
- Fetches full issue details from GitHub
- Posts comment: `{adw_id}_ops: ✅ Starting ADW workflow`
- Executes classifier agent with issue title and description
- Receives classification: `/chore`, `/bug`, or `/feature`
- Classification stored in `agents/{adw_id}/classifier/raw_output.jsonl`

#### 3. Branch Creation
- Automatic feature branch creation with naming convention:
  ```
  {type}-{issue_number}-{adw_id}-{slug}
  ```
  Examples:
  - `feat-456-a1b2c3d4-add-user-auth`
  - `bug-789-e5f6g7h8-fix-login-error`
  - `chore-123-i9j0k1l2-update-docs`
- Branch created from current main branch
- Checked out for all subsequent operations

#### 4. Plan Generation
- Executes sdlc_planner agent with classified issue
- Agent creates detailed implementation plan
- Plan saved to `specs/{slug}-plan.md`
- Plan includes:
  - Technical approach
  - Step-by-step tasks
  - File modifications
  - Validation commands
  - Testing requirements

#### 5. Plan Commit
- Commits the plan file with message format:
  ```
  sdlc_planner: {type}: {description} for #{issue_number}

  Generated with ADW ID: {adw_id}
  🤖 Generated with [Claude Code](https://claude.ai/code)
  ```

#### 6. Implementation
- Executes sdlc_implementor agent with plan file path
- Agent reads plan and implements all steps
- Agent runs validation commands
- Implementation logged to `agents/{adw_id}/sdlc_implementor/raw_output.jsonl`

#### 7. Implementation Commit
- Commits all changes with semantic commit message:
  ```
  sdlc_implementor: {type}: {description} for #{issue_number}

  Generated with ADW ID: {adw_id}
  🤖 Generated with [Claude Code](https://claude.ai/code)
  ```

#### 8. Pull Request Creation
- Creates PR using `gh pr create`
- PR title: Issue title
- PR body includes:
  - Summary of changes
  - Link to original issue
  - ADW ID for traceability
  - Implementation approach
  - Testing performed
- PR automatically linked to original issue

### Environment Setup Requirements

#### Prerequisites

- **Python 3.12+** - Required for modern type hints and features
- **uv** - Python package manager (https://astral.sh/uv)
- **gh CLI** - GitHub command-line tool (https://cli.github.com)
- **Claude Code CLI** - Anthropic's Claude Code CLI (https://docs.anthropic.com/en/docs/claude-code)

#### Required Environment Variables

```bash
# Required for Claude Code (unless using subscription)
export ANTHROPIC_API_KEY="sk-ant-..."

# Optional: Path to Claude Code CLI (defaults to "claude")
export CLAUDE_CODE_PATH="/path/to/claude"

# Auto-detected from git remote (can override)
export GITHUB_REPO_URL="https://github.com/owner/repository"

# Optional: Only if using different GitHub account than 'gh auth login'
export GITHUB_PAT="ghp_..."

# Optional: For sandbox environments
export E2B_API_KEY="..."
```

#### Installation Steps

```bash
# Install GitHub CLI
brew install gh              # macOS
# or: sudo apt install gh    # Ubuntu/Debian
# or: winget install --id GitHub.cli  # Windows

# Authenticate with GitHub
gh auth login

# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh  # macOS/Linux
# or: powershell -c "irm https://astral.sh/uv/install.ps1 | iex"  # Windows

# Install Claude Code CLI
# Follow instructions at https://docs.anthropic.com/en/docs/claude-code

# Verify installation
uv run adws/health_check.py
```

#### Health Check Validation

Run `health_check.py` to verify your setup:

```bash
uv run adws/health_check.py
```

The health check validates:
- ✅ Environment variables (ANTHROPIC_API_KEY, CLAUDE_CODE_PATH)
- ✅ Git repository configuration
- ✅ Claude Code CLI installation and functionality
- ✅ GitHub CLI authentication
- ✅ Repository URL detection

### Usage and Trigger Options

#### adw_plan_build.py - Manual Single-Issue Processing

Process a specific issue on-demand for testing or manual workflows.

```bash
# Basic usage
uv run adws/adw_plan_build.py 123

# What happens:
# 1. Fetches issue #123 from GitHub
# 2. Creates feature branch (e.g., feat-123-a1b2c3d4-issue-title)
# 3. Classifies issue type using classifier agent
# 4. Generates implementation plan using sdlc_planner agent
# 5. Implements solution using sdlc_implementor agent
# 6. Creates commits and pull request
```

**Example Output:**
```
ADW ID: a1b2c3d4
issue_command: /feature
Working on branch: feat-123-a1b2c3d4-add-authentication
plan_file_path: specs/add-authentication-system-plan.md
Pull request created: https://github.com/owner/repo/pull/456
```

**Use Cases:**
- Testing ADW workflow on specific issues
- Processing high-priority issues immediately
- Development and debugging of ADW system

#### trigger_cron.py - Continuous Monitoring

Automated polling for new issues or "adw" comment triggers.

```bash
# Start monitoring (polls every 20 seconds)
uv run adws/trigger_cron.py

# Detection logic:
# - Processes new issues that have no comments
# - Processes issues where latest comment is exactly "adw"
# - Ignores already-processed issues

# Example log output:
# 2024-01-15 10:30:45 - Starting ADW cron trigger
# 2024-01-15 10:30:46 - Issue #123 has no comments - processing
# 2024-01-15 10:30:47 - Issue #456 - latest comment is 'adw' - processing
# 2024-01-15 10:31:07 - Polling... (checking for new issues)
```

**Deployment as systemd service:**
```bash
# Create service file: /etc/systemd/system/adw-cron.service
[Unit]
Description=ADW Cron Trigger
After=network.target

[Service]
Type=simple
User=your-user
WorkingDirectory=/path/to/project
Environment="ANTHROPIC_API_KEY=sk-ant-..."
ExecStart=/path/to/uv run adws/trigger_cron.py
Restart=always

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl enable adw-cron
sudo systemctl start adw-cron
```

**Use Cases:**
- Automated issue processing for teams
- Background monitoring without manual intervention
- Environments where webhooks aren't available

#### trigger_webhook.py - Real-Time Webhook Server

FastAPI server for instant GitHub event processing.

```bash
# Start webhook server (default port 8001)
uv run adws/trigger_webhook.py

# Custom port
PORT=3000 uv run adws/trigger_webhook.py

# Server endpoints:
# - POST /gh-webhook - Receives GitHub events
# - GET /health - Health check endpoint
```

**GitHub Webhook Configuration:**

1. Go to your repository's Settings → Webhooks
2. Add webhook with:
   - **Payload URL**: `https://your-server.com/gh-webhook`
   - **Content type**: `application/json`
   - **Events**: Select "Issues" and "Issue comments"
   - **Active**: ✅ Checked

3. For local development, use a tunnel:
   ```bash
   # Using ngrok
   ngrok http 8001
   # Use the ngrok URL in GitHub webhook settings
   ```

**How it works:**
- Receives `issues.opened` events → starts ADW workflow
- Receives `issue_comment.created` events with "adw" body → starts ADW workflow
- Responds within GitHub's 10-second timeout requirement
- Launches workflows in background to avoid blocking

**Use Cases:**
- Production deployments requiring instant response
- Teams that need immediate issue processing
- Integration with CI/CD pipelines

#### health_check.py - System Validation

Validates ADW system configuration before deployment.

```bash
uv run adws/health_check.py

# Checks performed:
# ✅ ANTHROPIC_API_KEY or CLAUDE_CODE_SUBSCRIPTION
# ✅ CLAUDE_CODE_PATH (defaults to "claude")
# ✅ Git repository detected
# ✅ Claude Code CLI installed and executable
# ✅ GitHub CLI authenticated
# ✅ Repository URL detected from git remote

# Example output:
# ✅ All health checks passed
# Repository: https://github.com/owner/repo
# Claude Code: /usr/local/bin/claude
# GitHub Auth: Logged in as username
```

**Use Cases:**
- Pre-deployment validation
- Troubleshooting environment issues
- Verifying setup after configuration changes

### ADW Tracking and Logging

#### ADW ID System

Every workflow run receives a unique 8-character identifier generated by `utils.make_adw_id()`:
- Format: `[a-z0-9]{8}` (e.g., `a1b2c3d4`)
- Appears in all workflow artifacts:
  - Issue comments: `a1b2c3d4_ops: ✅ Starting ADW workflow`
  - Branch names: `feat-123-a1b2c3d4-issue-title`
  - Commit messages: `Generated with ADW ID: a1b2c3d4`
  - Output directories: `agents/a1b2c3d4/`
  - Pull request descriptions
- Enables complete traceability from issue to code

#### Output Directory Structure

All agent execution artifacts are organized by ADW ID:

```
agents/
├── a1b2c3d4/                    # ADW workflow run
│   ├── classifier/
│   │   ├── raw_output.jsonl     # Full JSONL output from agent
│   │   └── prompts/
│   │       └── prompt.txt       # Exact prompt sent to agent
│   ├── sdlc_planner/
│   │   ├── raw_output.jsonl
│   │   └── prompts/
│   │       └── prompt.txt
│   └── sdlc_implementor/
│       ├── raw_output.jsonl
│       └── prompts/
│           └── prompt.txt
├── e5f6g7h8/                    # Another workflow run
│   └── ...
```

Each agent directory contains:
- **raw_output.jsonl** - Complete JSONL output from Claude Code CLI
- **prompts/prompt.txt** - Exact prompt template sent to the agent

#### Logging System

Configured by `utils.setup_logger()` with dual output:

**Console Output (INFO and above):**
```
2024-01-15 10:30:45 - INFO - Starting ADW workflow for issue #123
2024-01-15 10:30:46 - INFO - ADW ID: a1b2c3d4
2024-01-15 10:30:47 - INFO - Executing classifier agent
2024-01-15 10:31:02 - INFO - Classification: /feature
```

**File Logging (DEBUG and above):**
- Location: `logs/adw_{timestamp}.log`
- Includes full DEBUG details for troubleshooting
- Environment variable dumps
- Agent execution parameters
- Git operation details

**Viewing Logs:**
```bash
# View latest log file
tail -f logs/adw_*.log | tail -1

# Search logs by ADW ID
grep "a1b2c3d4" logs/*.log

# View agent JSONL output
cat agents/a1b2c3d4/sdlc_planner/raw_output.jsonl | jq .

# View exact prompt sent to agent
cat agents/a1b2c3d4/sdlc_planner/prompts/prompt.txt
```

### ADW Integration

The ADW system integrates with multiple external systems:

#### GitHub Issues Integration

**Operations performed via `github.py`:**

- **Issue Fetching** (`fetch_issue()`)
  - Uses `gh api` to retrieve full issue details
  - Returns typed GitHubIssue model
  - Includes title, body, labels, assignees, comments

- **Comment Posting** (`make_issue_comment()`)
  - Posts workflow status updates to issues
  - Format: `{adw_id}_ops: ✅ Starting ADW workflow`
  - Provides visibility to issue creators

- **Label Management** (`mark_issue_in_progress()`)
  - Adds labels: `adw-in-progress`, issue type labels
  - Assigns issue to configured GitHub user
  - Signals active processing

- **Issue Listing** (`fetch_open_issues()`)
  - Lists all open issues for monitoring
  - Used by trigger_cron.py for polling
  - Filters by comment criteria

#### Claude Code CLI Integration

**Operations performed via `agent.py`:**

- **Slash Command Execution** (`execute_template()`)
  - Invokes Claude Code with slash commands: `/classifier`, `/plan`, `/implement`
  - Commands defined in `.claude/commands/` directory
  - Passes issue context as arguments

- **Prompt Execution** (`prompt_claude_code()`)
  - Executes Claude Code with custom prompts
  - Manages subprocess execution
  - Captures stdout/stderr for logging

- **JSONL Output Parsing** (`parse_jsonl_output()`)
  - Parses line-delimited JSON output from agents
  - Extracts tool calls, results, and completion messages
  - Returns structured ClaudeCodeResult

- **Environment Setup**
  - Sets ANTHROPIC_API_KEY from environment
  - Configures output directories for each agent
  - Stores prompts and raw outputs for debugging

- **CLI Validation** (`check_claude_installed()`)
  - Verifies Claude Code CLI is installed
  - Checks executable permissions
  - Validates version compatibility

#### Git Integration

**Operations performed by `adw_plan_build.py`:**

- **Branch Management**
  - Creates feature branches with pattern: `{type}-{issue_number}-{adw_id}-{slug}`
  - Checks out branch for all operations
  - Pushes to remote with `-u` flag

- **Commit Creation**
  - Two commits per workflow:
    1. Plan commit (after sdlc_planner)
    2. Implementation commit (after sdlc_implementor)
  - Format:
    ```
    {agent_name}: {type}: {description} for #{issue_number}

    Generated with ADW ID: {adw_id}
    🤖 Generated with [Claude Code](https://claude.ai/code)
    ```

- **Pull Request Generation**
  - Uses `gh pr create` to open PR
  - Title: Issue title
  - Body: Implementation summary with ADW ID
  - Automatically links to original issue via `#123` syntax

#### Custom Slash Commands Integration

**Located in `.claude/commands/` directory:**

- **classifier.md** - Issue classification command
  - Input: Issue title and description
  - Output: `/chore`, `/bug`, or `/feature`
  - Maps to IssueClassSlashCommand enum in data_types.py

- **plan.md** (sdlc_planner) - Plan generation command
  - Input: Classified issue with type
  - Output: Detailed implementation plan file
  - Saved to `specs/{slug}-plan.md`

- **implement.md** (sdlc_implementor) - Implementation command
  - Input: Path to plan file
  - Output: Code changes implementing the plan
  - Executes validation commands from plan

**Command Type Mapping:**
```python
class IssueClassSlashCommand(str, Enum):
    CHORE = "/chore"
    BUG = "/bug"
    FEATURE = "/feature"
```

These types ensure type-safe classification throughout the workflow.

### ADW Troubleshooting

#### First Step: Run Health Check

Always start debugging with the health check script:

```bash
uv run adws/health_check.py
```

This validates:
- ✅ Environment variables
- ✅ Git repository setup
- ✅ Claude Code CLI installation
- ✅ GitHub CLI authentication

#### Common Issues and Solutions

**"Claude Code CLI is not installed"**
```bash
# Check if installed
which claude
echo $CLAUDE_CODE_PATH

# Reinstall from official source
# https://docs.anthropic.com/en/docs/claude-code

# Set custom path if not in PATH
export CLAUDE_CODE_PATH="/path/to/claude"
```

**"Missing ANTHROPIC_API_KEY"**
```bash
# Check if set
echo $ANTHROPIC_API_KEY

# Set API key
export ANTHROPIC_API_KEY="sk-ant-..."

# Or use Claude Code subscription (no API key needed)
claude auth login
```

**"GitHub CLI not authenticated"**
```bash
# Check authentication
gh auth status

# Login to GitHub
gh auth login

# Verify access to repository
gh repo view owner/repository
```

**"Agent execution failed"**
```bash
# Check agent output for errors
cat agents/{adw_id}/sdlc_planner/raw_output.jsonl | tail -1 | jq .

# View exact prompt sent
cat agents/{adw_id}/sdlc_planner/prompts/prompt.txt

# Check logs for details
tail -f logs/adw_*.log
```

**"Branch already exists"**
```bash
# Delete existing branch (careful!)
git branch -D feat-123-a1b2c3d4-old-branch
git push origin --delete feat-123-a1b2c3d4-old-branch

# Or work with existing branch
git checkout feat-123-a1b2c3d4-old-branch
```

**"PR creation failed"**
```bash
# Check if PR already exists
gh pr list --head feat-123-a1b2c3d4-branch-name

# Manually create PR
gh pr create --title "Title" --body "Description"

# Check GitHub CLI permissions
gh auth status
```

#### Viewing Agent Logs

**Check classifier output:**
```bash
cat agents/{adw_id}/classifier/raw_output.jsonl | jq .
```

**Check planner output:**
```bash
cat agents/{adw_id}/sdlc_planner/raw_output.jsonl | jq .
```

**Check implementor output:**
```bash
cat agents/{adw_id}/sdlc_implementor/raw_output.jsonl | jq .
```

**View prompts sent to agents:**
```bash
cat agents/{adw_id}/sdlc_planner/prompts/prompt.txt
```

#### Debugging Workflows

**Enable debug mode:**
```bash
export ADW_DEBUG=true
uv run adws/adw_plan_build.py 123
```

**Check workflow status:**
```bash
# View recent commits
git log --oneline | head -10

# Check current branch
git branch --show-current

# View PR status
gh pr list
```

**Search logs by ADW ID:**
```bash
grep "a1b2c3d4" logs/*.log
```

#### Configuration Options

**Model Selection**

Edit `agent.py` around line 129 to change the model:
```python
# Faster, lower cost (default)
model="sonnet"

# Better for complex tasks
model="opus"

# Fastest, cheapest (for simple tasks)
model="haiku"
```

**Polling Interval**

Edit `trigger_cron.py` to change polling frequency:
```python
# Default: 20 seconds
time.sleep(20)

# Change to 60 seconds
time.sleep(60)
```

**Webhook Port**

Change the port for webhook server:
```bash
PORT=3000 uv run adws/trigger_webhook.py
```

#### Additional Resources

- **ADW System Documentation**: `adws/README.md`
- **Data Models**: `adws/data_types.py`
- **Agent Implementation**: `adws/agent.py`
- **GitHub Operations**: `adws/github.py`
- **Workflow Orchestration**: `adws/adw_plan_build.py`

## API Endpoints

- `POST /api/upload` - Upload CSV/JSON file
- `POST /api/query` - Process natural language query
- `GET /api/schema` - Get database schema
- `POST /api/insights` - Generate column insights
- `GET /api/health` - Health check

## Security

### SQL Injection Protection

The application implements comprehensive SQL injection protection through multiple layers:

1. **Centralized Security Module** (`core/sql_security.py`):
   - Identifier validation for table and column names
   - Safe query execution with parameterized queries
   - Proper escaping for identifiers using SQLite's square bracket notation
   - Dangerous operation detection and blocking

2. **Input Validation**:
   - All table and column names are validated against a whitelist pattern
   - SQL keywords cannot be used as identifiers
   - File names are sanitized before creating tables
   - User queries are validated for dangerous operations

3. **Query Execution Safety**:
   - Parameterized queries used wherever possible
   - Identifiers (table/column names) are properly escaped
   - Multiple statement execution is blocked
   - SQL comments are not allowed in queries

4. **Protected Operations**:
   - File uploads with malicious names are sanitized
   - Natural language queries cannot inject SQL
   - Table deletion uses validated identifiers
   - Data insights generation validates all inputs

### Security Best Practices for Development

When adding new SQL functionality:
1. Always use the `sql_security` module functions
2. Never concatenate user input directly into SQL strings
3. Use `execute_query_safely()` for all database operations
4. Validate all identifiers with `validate_identifier()`
5. For DDL operations, use `allow_ddl=True` explicitly

### Testing Security

Run the comprehensive security tests:
```bash
cd app/server
uv run pytest tests/test_sql_injection.py -v
```


### Additional Security Features

- CORS configured for local development only
- File upload validation (CSV and JSON only)
- Comprehensive error logging without exposing sensitive data
- Database operations are isolated with proper connection handling

## Troubleshooting

**Backend won't start:**
- Check Python version: `python --version` (requires 3.12+)
- Verify API keys are set: `echo $OPENAI_API_KEY`

**Frontend errors:**
- Clear node_modules: `rm -rf node_modules && npm install`
- Check Node version: `node --version` (requires 18+)

**CORS issues:**
- Ensure backend is running on port 8000
- Check vite.config.ts proxy settings