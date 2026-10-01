# PlatformPerfAgent

VS Code customizations for evidence-driven CPU and platform performance analysis.

## Included Customizations

- `platform-perf-engineer` is a workspace custom agent for topology-aware performance investigations, EMON analysis, scaling studies, and performance regressions.
- `intel-perf-report` is an on-demand skill for producing Intel-branded, single-file HTML performance reports.

The agent loads and applies the report skill for HTML-report requests. The skill is not a sub-agent and is not invoked through `agent/runSubagent`.

## Repository Layout

```text
.github/
	agents/
		platform-perf-engineer.agent.md
	skills/
		intel-perf-report/
			SKILL.md
.vscode/
	mcp.json
```

VS Code discovers workspace agents from `.github/agents/*.agent.md` and skills from `.github/skills/<skill-name>/SKILL.md`.

## Setup

1. Open this repository in VS Code with GitHub Copilot Chat enabled.
2. Select the `platform-perf-engineer` custom agent from the chat agent picker.
3. Configure the iConsole API key when VS Code prompts for `console-api-key`.
4. Verify access to the configured MCP services in `.vscode/mcp.json` before requesting fleet, workload-controller, or DCSO metrics data.

## Usage

Ask the agent to analyze performance data, investigate a regression, design a topology-aware scaling experiment, or generate an HTML performance report. Requests for an HTML report cause the agent to load and follow the `intel-perf-report` skill in the same session.

Example requests:

```text
Analyze this EMON Excel file and explain the scaling bottleneck.
Compare these two platform runs and generate an HTML report.
Design a topology-aware thread-scaling plan for this workload.
```