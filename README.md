# Claude Code System Prompts

An independent research project documenting the internal prompt architecture and agentic behavior of **Claude Code**, Anthropic's AI-powered software engineering assistant.

This documentation was produced through behavioral analysis, output observation, and study of publicly available discussions. It serves as an educational resource for understanding how modern agentic AI coding assistants are designed.

## Overview

Claude Code uses a sophisticated multi-layered prompt architecture. The main system prompt is not a static string but is dynamically assembled at runtime from modular section-builder functions. A boundary marker splits it into a globally cacheable prefix and a session-specific suffix, enabling prompt caching across API calls.

Beyond the core identity prompt, the system includes specialized agent prompts, a multi-worker coordinator, a 2-stage security classifier for auto-approving tool calls, and a suite of utility prompts for memory selection, session search, and tool summarization.

## Prompt Catalog

### Core Identity

| # | Prompt | Description |
|---|--------|-------------|
| 01 | [Main System Prompt](prompts/01_main_system_prompt.md) | Dynamically assembled master prompt covering identity, behavior, tool guidance, tone, and efficiency |
| 02 | [Simple Mode](prompts/02_simple_mode.md) | Minimal 4-line prompt activated by `CLAUDE_CODE_SIMPLE` |
| 03 | [Default Agent Prompt](prompts/03_default_agent_prompt.md) | Base prompt inherited by all sub-agents |
| 04 | [Cyber Risk Instruction](prompts/04_cyber_risk_instruction.md) | Security boundaries for authorized vs. prohibited actions |

### Orchestration

| # | Prompt | Description |
|---|--------|-------------|
| 05 | [Coordinator System Prompt](prompts/05_coordinator_system_prompt.md) | Multi-worker orchestrator with 4-phase workflow and concurrency rules |
| 06 | [Teammate Prompt Addendum](prompts/06_teammate_prompt_addendum.md) | Communication protocol for swarm and team mode |

### Specialized Agents

| # | Prompt | Description |
|---|--------|-------------|
| 07 | [Verification Agent](prompts/07_verification_agent.md) | Adversarial testing specialist that tries to break implementations |
| 08 | [Explore Agent](prompts/08_explore_agent.md) | Read-only codebase exploration with strict no-modify constraints |
| 09 | [Agent Creation Architect](prompts/09_agent_creation_architect.md) | Designs new agent configurations from user requirements |
| 10 | [Status Line Setup Agent](prompts/10_statusline_setup_agent.md) | Configures terminal status line across shell environments |

### Security and Permissions

| # | Prompt | Description |
|---|--------|-------------|
| 11 | [Permission Explainer](prompts/11_permission_explainer.md) | Explains tool risk levels before user approval |
| 12 | [Auto Mode Classifier](prompts/12_yolo_auto_mode_classifier.md) | 2-stage security classifier for auto-approving tool calls |

### Tool Descriptions

| # | Prompt | Description |
|---|--------|-------------|
| 13 | [Tool-Specific Prompts](prompts/13_tool_prompts.md) | All 30+ tool descriptions including Bash, Edit, Agent, and fork semantics |

### Utility and Helpers

| # | Prompt | Description |
|---|--------|-------------|
| 14 | [Tool Use Summary](prompts/14_tool_use_summary.md) | Generates git-commit-style labels for completed tool batches |
| 15 | [Session Search](prompts/15_session_search.md) | Semantic search across past conversation sessions |
| 16 | [Memory Selection](prompts/16_memory_selection.md) | Selects relevant memory files for query context |
| 17 | [Auto Mode Critique](prompts/17_auto_mode_critique.md) | Reviews user-written auto-mode classifier rules |
| 20 | [Session Title](prompts/20_session_title.md) | Haiku-powered 3-7 word session title generator |
| 29 | [Agent Summary](prompts/29_agent_summary.md) | Periodic progress updates for sub-agents in coordinator mode |
| 30 | [Prompt Suggestion](prompts/30_prompt_suggestion.md) | Predicts user follow-up commands for clickable suggestions |

### Context Window Management

| # | Prompt | Description |
|---|--------|-------------|
| 21 | [Compact Service](prompts/21_compact_service.md) | Multi-variant conversation summarization with analysis/summary blocks |
| 22 | [Away Summary](prompts/22_away_summary.md) | 1-3 sentence session recap for returning users |

### Dynamic Sections

| # | Prompt | Description |
|---|--------|-------------|
| 18 | [Proactive Mode](prompts/18_proactive_mode.md) | Autonomous agent with tick-based pacing and terminal focus awareness |
| 23 | [Chrome Browser Automation](prompts/23_chrome_browser_automation.md) | Browser extension integration: GIF recording, tab management, dialog handling |
| 24 | [Memory Instruction](prompts/24_memory_instruction.md) | CLAUDE.md loading, @include directives, and frontmatter globs |

### Bundled Skills

| # | Prompt | Description |
|---|--------|-------------|
| 19 | [Simplify Skill](prompts/19_simplify_skill.md) | Three-agent parallel review for code reuse, quality, and efficiency |
| 25 | [Skillify Skill](prompts/25_skillify.md) | Interview-based skill creation, generates SKILL.md files |
| 26 | [Stuck Skill](prompts/26_stuck_skill.md) | Internal diagnostic for frozen or slow sessions |
| 27 | [Remember Skill](prompts/27_remember_skill.md) | Promotes auto-memory entries into CLAUDE.md files |
| 28 | [Update Config Skill](prompts/28_update_config_skill.md) | Manages settings.json, hooks, and permission arrays |

## Architecture

### Prompt Assembly

The main system prompt is built through a pipeline of section builders:

```
System Prompt Assembly
    |
    |   Static Prefix (globally cached)
    |-- Identity and Cyber Risk
    |-- Permission modes, hooks, reminders
    |-- Code style, security, error handling
    |-- Reversibility, blast radius
    |-- Tool preferences, parallel calls
    |-- Tone and style rules
    |-- Output efficiency patterns
    |
    |   CACHE BOUNDARY
    |
    |   Dynamic Suffix (session-specific)
    |-- Agent tools, skills, verification
    |-- Memory file content
    |-- Model overrides
    |-- Environment info (CWD, OS, git state)
    |-- Language preferences
    |-- Custom output styles
    |-- MCP server instructions
    |-- Context window management
```

### Auto Mode Classifier

The auto-approval system uses a separate classifier prompt assembled from:

1. **Base prompt** with classifier instructions
2. **Default rules** with allow, deny, and environment sections
3. **User overrides** that replace sections entirely
4. **2-stage classification**: Stage 1 runs fast, Stage 2 uses extended thinking if uncertain

### Context Window Pipeline

```
User Message
    |
    v
[Micro-Compaction]  -- Cache-aware tool result deletion
    |
    v
[Compact Service]   -- Full/partial summarization
    |
    v
[Prompt Suggestion] -- Predict next user command
    |
    v
[Away Summary]      -- Session recap if user was idle
```

### Memory System

```
Memory Loading Order (first loaded = lowest priority):
    |
    |-- Enterprise managed config
    |-- User global config
    |-- Project config (shared, checked in)
    |-- Project rules directory
    |-- Local config (private, git-ignored)
    |
    |   @include directives resolve transitively (max depth: 5)
    |   Frontmatter paths field enables conditional injection
```

### Environment Variables

| Variable | Effect |
|----------|--------|
| `CLAUDE_CODE_SIMPLE` | Activates the minimal 4-line system prompt |
| `USER_TYPE=ant` | Enables internal-only sections and model overrides |
| Feature flags | Gate proactive mode, verification agents, fork subagents, and more |

## Repository Structure

```
claude-code-system-prompts/
    README.md
    prompts/
        01_main_system_prompt.md        19_simplify_skill.md
        02_simple_mode.md               20_session_title.md
        03_default_agent_prompt.md      21_compact_service.md
        04_cyber_risk_instruction.md    22_away_summary.md
        05_coordinator_system_prompt.md 23_chrome_browser_automation.md
        06_teammate_prompt_addendum.md  24_memory_instruction.md
        07_verification_agent.md        25_skillify.md
        08_explore_agent.md             26_stuck_skill.md
        09_agent_creation_architect.md  27_remember_skill.md
        10_statusline_setup_agent.md    28_update_config_skill.md
        11_permission_explainer.md      29_agent_summary.md
        12_yolo_auto_mode_classifier.md 30_prompt_suggestion.md
        13_tool_prompts.md
        14_tool_use_summary.md
        15_session_search.md
        16_memory_selection.md
        17_auto_mode_critique.md
        18_proactive_mode.md
```

## Purpose

This project exists as an educational resource for AI researchers, developers building agentic systems, and anyone interested in understanding the design patterns behind production-grade AI coding assistants.

The documented patterns cover topics relevant to the broader AI engineering community:
- Multi-agent orchestration and coordination
- Security classification for autonomous tool use
- Context window management and conversation compaction
- Memory systems with hierarchical override semantics
- Prompt caching strategies for latency optimization

## Disclaimer

This is an independent research project. The content represents analysis and observations of Claude Code's behavior and architecture. This project is not affiliated with, endorsed by, or connected to Anthropic in any way. All trademarks belong to their respective owners.
