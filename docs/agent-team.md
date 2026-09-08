# Project Pulse agent team

This team uses GitHub Copilot CLI to orchestrate the work for Mona's Project Pulse dashboard. All four custom agents use GPT-5.6 Luna.

| Agent | Model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | GPT-5.6 Luna | Coordinates the specialist agents, assigns file scopes, manages dependencies, and validates the integrated result. | `.github/agents/orchestrator.agent.md` |
| Planner | GPT-5.6 Luna | Researches the repository and creates the implementation plan, including phases, ownership, dependencies, parallel work, and validation. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.6 Luna | Implements the assigned Project Pulse application files with clear, testable, and deterministic behavior. | `.github/agents/coder.agent.md` |
| Designer | GPT-5.6 Luna | Defines the dashboard's usability, accessibility, information hierarchy, interaction flow, and visual design. | `.github/agents/designer.agent.md` |

The Orchestrator first asks the Planner to create the implementation plan, then coordinates Designer and Coder work using non-overlapping file assignments where possible. The Orchestrator reviews the completed Project Pulse dashboard, confirms validation results, and prepares the final handoff.
