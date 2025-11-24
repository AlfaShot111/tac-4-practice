# Chore: Document the ADW System

## Chore Description
Update the main README.md file to include comprehensive documentation about the AI Developer Workflow (ADW) system located in the `adws/` directory. The documentation should explain how the ADW system works, its architecture, components, triggers, workflow stages, and how it integrates with GitHub issues and Claude Code CLI to automate software development tasks.

The ADW system is an automated development workflow that:
- Monitors GitHub issues through cron jobs or webhooks
- Classifies issues into types (chore/bug/feature)
- Generates implementation plans using Claude Code
- Implements solutions automatically
- Creates commits and pull requests

## Relevant Files
Use these files to resolve the chore:

- **README.md** - Main project documentation that needs to be updated with ADW system information
  - Currently has a minimal "ADWs" section that only lists the scripts
  - Needs comprehensive documentation about the system architecture, components, and workflow

- **adws/README.md** - Contains detailed ADW documentation
  - Quick start guide with environment setup
  - Script usage guide for each ADW script
  - Workflow explanation
  - Troubleshooting and configuration information

- **adws/data_types.py** - Pydantic data models used throughout the ADW system
  - Defines GitHub API response models (GitHubIssue, GitHubComment, GitHubUser, etc.)
  - Defines agent request/response models (AgentPromptRequest, AgentTemplateRequest, etc.)
  - Defines slash command types and Claude Code result structures

- **adws/utils.py** - Utility functions for ADW system
  - `make_adw_id()` - Generates unique 8-character workflow IDs
  - `setup_logger()` - Configures logging to both console and file
  - Logging infrastructure for tracking workflow execution

- **adws/github.py** - GitHub API operations module
  - `fetch_issue()` - Retrieves issue details using gh CLI
  - `make_issue_comment()` - Posts comments to issues
  - `mark_issue_in_progress()` - Adds labels and assignees
  - `fetch_open_issues()` - Lists all open issues
  - `get_repo_url()` and `extract_repo_path()` - Repository information extraction

- **adws/agent.py** - Claude Code CLI integration module
  - `prompt_claude_code()` - Executes Claude Code with prompts
  - `execute_template()` - Runs slash command templates
  - `parse_jsonl_output()` - Parses Claude Code JSONL output
  - `check_claude_installed()` - Validates CLI installation
  - Environment setup and output management

- **adws/adw_plan_build.py** - Main workflow orchestration script
  - Entry point for processing a single GitHub issue
  - Orchestrates the complete plan-and-build workflow
  - Manages agent execution sequence (classifier → planner → implementor)
  - Handles git operations (branching, committing)
  - Creates pull requests with full context

- **adws/trigger_cron.py** - Automated monitoring trigger
  - Polls GitHub every 20 seconds for qualifying issues
  - Detects new issues without comments
  - Detects issues with "adw" comment trigger
  - Launches adw_plan_build.py in background for each issue

- **adws/trigger_webhook.py** - GitHub webhook server
  - FastAPI server for real-time GitHub event processing
  - Listens for issue creation and comment events
  - Responds immediately to meet GitHub's 10-second timeout
  - Launches workflows in background with unique ADW IDs

- **adws/health_check.py** - System health validation script
  - Validates environment variables (ANTHROPIC_API_KEY, CLAUDE_CODE_PATH, etc.)
  - Checks git repository configuration
  - Tests Claude Code CLI functionality
  - Tests GitHub CLI authentication
  - Returns structured health check results

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Read and Understand Current Documentation
- Read the main README.md to understand the current structure and identify where ADW documentation should be expanded
- Read adws/README.md completely to understand all the information that needs to be incorporated
- Understand the relationship between all ADW components and how they work together

### Step 2: Create Comprehensive ADW System Overview Section
- Update the existing "ADWs" section in README.md with a comprehensive overview
- Include a high-level explanation of what the ADW system does
- Explain the value proposition: automated issue processing from detection to PR creation
- Add a clear description of the three trigger modes (manual, cron, webhook)
- Include information about the unique ADW ID tracking system

### Step 3: Document ADW System Architecture
- Add a new "ADW System Architecture" subsection
- Explain the core components and their roles:
  - **Core Modules**: agent.py, github.py, data_types.py, utils.py
  - **Workflow Scripts**: adw_plan_build.py (main orchestrator)
  - **Triggers**: trigger_cron.py, trigger_webhook.py
  - **Utilities**: health_check.py
- Describe how components interact with each other
- Explain the agent execution model (classifier, planner, implementor)

### Step 4: Document the ADW Workflow Stages
- Add a new "ADW Workflow Process" subsection with detailed stage information:
  1. **Trigger Detection** - How issues are detected (cron polling, webhooks, manual)
  2. **Issue Classification** - Using Claude Code to classify as /chore, /bug, or /feature
  3. **Branch Creation** - Automatic feature branch naming convention: `{type}-{issue_number}-{adw_id}-{slug}`
  4. **Plan Generation** - sdlc_planner agent creates implementation plan in specs/ directory
  5. **Plan Commit** - Commits the plan file with proper message format
  6. **Implementation** - sdlc_implementor agent executes the plan
  7. **Implementation Commit** - Commits all changes with semantic commit messages
  8. **Pull Request Creation** - Automated PR with full context and links
- Include the commit message format and PR structure

### Step 5: Document Environment Setup Requirements
- Add a new "ADW Environment Setup" subsection
- List all required environment variables with descriptions:
  - `ANTHROPIC_API_KEY` - Required for Claude Code (unless using subscription)
  - `CLAUDE_CODE_PATH` - Path to Claude Code CLI (defaults to "claude")
  - `GITHUB_REPO_URL` - Automatically detected from git remote
  - `GITHUB_PAT` - Optional, only if using different GitHub account than 'gh auth login'
  - `E2B_API_KEY` - Optional, for sandbox environments
- List prerequisites: Python 3.12+, uv, gh CLI, Claude Code CLI
- Reference the health_check.py script for validation

### Step 6: Document Usage and Trigger Options
- Update the existing ADW script list with comprehensive usage information
- Expand each script with:
  - **adw_plan_build.py** - Manual single-issue processing with usage examples
  - **trigger_cron.py** - Continuous monitoring with polling interval and detection logic
  - **trigger_webhook.py** - Real-time webhook server with endpoint details and GitHub webhook configuration
  - **health_check.py** - System validation with all checks performed
- Include example commands and expected outputs

### Step 7: Add ADW Tracking and Logging Information
- Document the ADW ID tracking system (8-character unique identifiers)
- Explain the output directory structure: `agents/{adw_id}/{agent_name}/`
- Describe the logging system:
  - Console output for INFO and above
  - File logging with full DEBUG details
  - Location of execution logs and raw JSONL output
  - Prompt storage in prompts/ subdirectories

### Step 8: Document Integration Points
- Add a new "ADW Integration" subsection
- Explain how ADW integrates with:
  - **GitHub Issues** - Issue fetching, comment posting, label management
  - **Claude Code CLI** - Slash command execution, prompt templates, JSONL output parsing
  - **Git** - Branch management, commit creation, PR generation
  - **Custom Slash Commands** - Located in .claude/commands/ directory
- Mention the relationship between IssueClassSlashCommand types and custom commands

### Step 9: Add Troubleshooting and Configuration
- Add a "ADW Troubleshooting" subsection with common issues and solutions
- Reference the health_check.py script as the first debugging step
- Include common errors from adws/README.md
- Add information about viewing agent logs and debugging workflows
- Document the model selection configuration (sonnet vs opus in agent.py)

### Step 10: Run Validation Commands
- Verify README.md formatting is correct
- Ensure all sections are properly structured with markdown
- Validate that the documentation is comprehensive and accurate

## Validation Commands
Execute every command to validate the chore is complete with zero regressions.

- `cat README.md` - Verify the README.md has been updated with comprehensive ADW documentation
- `cd app/server && uv run pytest` - Run server tests to validate no regressions

## Notes
- The ADW system is a critical part of the project that automates the entire software development lifecycle
- The documentation should be accessible to developers who want to understand or extend the system
- Maintain consistency with the existing README.md style and structure
- The ADW system uses the Claude Agent SDK and custom slash commands in .claude/commands/
- Each ADW workflow run is uniquely identified and fully logged for debugging and tracking
- The system supports both API key-based and subscription-based Claude Code authentication
