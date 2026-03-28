# Changelog

All notable changes to the Agentic Design Patterns Skills library are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.12] — 2026-03-28

### Changed
- Removed legacy `github-release/` folder (superseded by `github-release-skills/`)
- Skills repo now managed exclusively via `github-release-skills/` (git clone)

---

## [1.11] — 2026-03-27

### Fixed
- Renamed `mcp` skill frontmatter `name` field from `mcp-setup` → kept as `mcp-setup`; folder renamed `mcp-setup/` → `mcp/` to avoid collision with Gemini CLI built-in `/mcp` slash command
- `PATTERN_DIR_MAP` in `mcp_server.py` maps `mcp-setup` → `mcp/` folder path

---

## [1.10] — 2026-03-27

### Added
- Gemini CLI Extension format created (`gemini-extension.json`, `GEMINI.md`, `mcp_server.py`, `commands/`, `agents/`)
- `github-release-extension/` folder — git clone of `hajekim/agentic-design-patterns-extension`
- `github-release-skills/` folder — git clone of `hajekim/agentic-design-patterns-skills`
- Two separate release workflows: Skills (multi-platform) and Extension (Gemini CLI)

---

## [1.9] — 2026-03-27

### Added
- Japanese triggers (284) added to all 28 skills
- Chinese (Simplified) triggers (281) added to all 28 skills
- Total trigger count: **1,376** across EN / KR / JA / ZH
- `## Multilingual Trigger System` section added to README with per-language examples

---

## [1.8] — 2026-03-27

### Fixed
- Migrated `google-generativeai` → `google-genai` across 23 skill files (deprecated SDK)
- Migrated `vertexai.TextEmbeddingModel` → `client.models.embed_content()` in 2 files
- Replaced deprecated LangChain imports across 2 files:
  - `langchain_community.vectorstores.Chroma` → `langchain-chroma`
  - `langchain.text_splitter` → `langchain-text-splitters`
  - `ConversationChain` → `RunnableWithMessageHistory`

---

## [1.7] — 2026-03-27

### Changed
- Gemini CLI and Antigravity installation guides expanded into separate sections
- Verified skill discovery paths: `.gemini/skills/`, `.agents/skills/` (cross-tool standard)
- Added install options A–D for each platform (user-level, workspace, Git, selective)
- Added `gemini skills` management commands (`list`, `link`, `install`, `uninstall`)
- Documented `.agents/skills/` precedence over `.gemini/skills/` within the same scope

---

## [1.6] — 2026-03-27

### Changed
- README fully rewritten in English (was mixed EN/KR)
- Korean language retained only in trigger phrase examples and the Multilingual Trigger section

---

## [1.5] — 2026-03-27

### Added
- Platform Compatibility table: Gemini CLI / Antigravity / Claude Code feature comparison
- Claude Code installation guide: options A–D (global, project, selective, direct reference)
- Branched Quick Start into per-platform Setup sections
- `## Platform Usage` section split into Gemini CLI, Antigravity, and Claude Code subsections
- Claude Code-specific frontmatter fields documented (`context`, `allowed-tools`, `argument-hint`)

---

## [1.4] — 2026-03-27

### Added
- Upgraded default model to `gemini-2.5-flash` across all 28 skill code examples
- `## Related Skills` section added to all 28 skills with concrete combination scenarios
- Appendix F — Reasoning Engines skill (`appendix-reasoning-engines/SKILL.md`)
  - Covers Thinking Budget parameter, standard vs. reasoning model selection, inference-time compute scaling, hybrid routing

### Changed
- Model Reference table added to README: Flash / Pro / Thinking Budget guidance

---

## [1.3] — 2026-03-27

### Added
- Korean trigger phrases added to all 28 skills (337 phrases total)
- Korean triggers cover both technical terms and conversational phrases ("~해줘", "~어떻게 해?")

---

## [1.2] — 2026-03-27

### Added
- Context Engineering section added to Prompt Chaining skill
- ML Model-Based Routing section added to Routing skill (scikit-learn classifier example)
- Agent Complexity Levels framework (Level 0–3) added to Prompt Chaining skill and README

---

## [1.1] — 2026-03-27

### Fixed
- ADK API corrected across all affected skills:
  - `Agent` → `LlmAgent` (correct class name)
  - `InMemoryRunner.run(text)` → `Runner` + `InMemorySessionService` pattern
  - `from google.adk.agents import LlmAgent`
  - `from google.adk.runners import Runner`
  - `from google.adk.sessions import InMemorySessionService`
- Fixed in: `mcp/SKILL.md`, `prompt-chaining/SKILL.md`, and all skills referencing ADK patterns

---

## [1.0] — 2026-03-27

### Added
- Initial release — 28 skills based on *Agentic Design Patterns* by Antonio Gulli
- 21 chapter skills: Core Patterns (7), State Management (4), Reliability (3), Advanced Patterns (7)
- 7 appendix skills: Prompt Engineering, GUI Agents, Agentic Frameworks, AgentSpace, AI CLI, Coding Agents, Reasoning Engines
- Each skill follows DEFINE → PLAN → ACTION workflow
- Multilingual triggers: English (474 phrases) + Korean (337 phrases)
- Google ADK (LlmAgent + Runner + InMemorySessionService) as primary implementation
- LangChain / LangGraph as secondary implementations
- CrewAI for multi-agent collaboration patterns
- Compatible with Gemini CLI, Antigravity, and Claude Code
