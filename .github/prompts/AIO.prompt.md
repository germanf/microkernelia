---
agent: agent
---

# Unified AI Agent Prompting Strategy (ES-KNN → USC)

- **Language**: Always respond in Spanish.
- **Goal**: Guide tasks through clear phases for coherent, traceable execution.

## Phase 1 — Task Definition (TaskDefinitionRequest)
Request precise task definition before acting:
- Context & objective: What exactly do you want to achieve?
- Environment & language: Tools, targets, platforms, constraints (e.g., `no_std`).
- Expected inputs/outputs: Types, formats, file paths.
- Success criteria: Validation, tests, concrete artifacts.

Template:
"Please describe the task in detail: context, language/environment, purpose, inputs/outputs, and constraints. Also specify success criteria (what you'll validate and how)."

## Phase 2 — Deep Understanding (UnderstandingPhase · ES-KNN)
Build understanding via exemplar selection by similarity (ES-KNN):
- Explore workspace and select relevant code (APIs, patterns, flows).
- Summarize key design and dependencies (modules, boundaries, data flows).
- Identify project conventions affecting implementation.

Template:
"First: explain in 2 sentences the applicable architecture/pattern. Then: select and summarize 2-3 code pieces closest to the problem and how they condition the solution."

## Phase 3 — Structured Execution (ExecutionPhase · USC)
Organize context and execute with consistency (USC):
- Structure the solution: steps, decisions, dependencies.
- Implement, validate, and produce minimal deliverables (code, binaries, test instructions).
- Self-refine if inconsistencies are detected.

Template:
"Reason through three brief approaches (USC) and select the most coherent. Detail steps, then execute the solution and validate. Deliver artifacts and test commands."

## Complementary Prompting Techniques
- **Step-Back Prompting**: Summarize general concepts before details.
- **Self-Ask**: Formulate and answer clarifying questions to refine requirements.
- **Tree-of-Thought**: Compare options (pros/cons) before deciding.
- **Few-Shot Contrastive CoT**: Use correct/incorrect examples for ambiguous cases.
- **Self-Refine**: Iteratively review and improve output when useful.

## Constraints & Success Criteria
- Respect project environment (e.g., `no_std` in kernel/drivers; `std` only in tooling).
- Maintain coherence with repository styles and conventions.
- Reference relevant workspace files/paths when explaining decisions.
- Clear deliverables: minimal functional code, build/run instructions, tests or validations.
- Success = executable/verifiable solution aligned with requirements and provided context.

## Closure
- If unclear, request precision in Phase 1.
- If dependencies or configurations are missing, flag them and propose alternatives.
- Keep all final responses in English.

## Model Recommendations by Phase

### Quick Reference — Model Selection
- **GPT-4o**: Deep code reading/architecture, large refactors.
- **GPT-5**: System design, complex decisions, Tree-of-Thought.
- **GPT-5.1 / 5.2**: Solid daily driver (default recommended).
- **GPT-5-Codex / 5.1-Codex**: Pure programming, code gen/correction.
- **GPT-5-Codex-Max**: Massive refactors, cross-cutting migrations.
- **GPT-5-Codex-Mini**: Fast, cheap autocomplete.
- **GPT-5-mini**: Simple scripts and utilities.
- **Raptor mini (Preview)**: Brainstorming, sketches, exploration.
- **Grok Code Fast 1**: Ultra-fast iterations.
- **Claude Opus 4.5**: Architecture and long reasoning.
- **Claude Sonnet 4 / 4.5**: Clean code + good explanations.
- **Claude Haiku 4.5**: Fast responses and comments.
- **Gemini 2.5 Pro**: Cross-stack analysis, docs, cloud.
- **Gemini 3 Pro / Flash**: Structured generation and speed.

### Model Recommendations by Use Case
- **GPT-4o**: UnderstandingPhase (ES-KNN) and architectural review.
- **GPT-5 (core)**: ExecutionPhase with USC, ToT, Self-Refine.
- **GPT-5.1 / 5.2**: Production daily driver (APIs, services, logic).
- **GPT-5-Codex**: Codegen, tests, guided refactors; use when the "what" is clear.
- **Codex-Max**: Full migrations/frameworks; cross-cutting changes.
- **Codex-Mini / 5-mini**: Snippets, helpers, scripts.
- **Raptor/Grok**: Ideation and speed; avoid critical logic.
- **Claude Opus/Sonnet/Haiku**: Architecture, code/explanation balance, speed.
- **Gemini**: Infra/cloud/docs; Flash prioritizes speed over depth.

### Mapping to Prompting Strategy
- **TaskDefinitionRequest**: Raptor mini / Claude Haiku / Gemini Flash.
- **UnderstandingPhase (ES-KNN, Self-Ask, Step-Back)**: GPT-4o / Claude Opus / Gemini 2.5 Pro.
- **ExecutionPhase (USC, Tree-of-Thought, Self-Refine)**: GPT-5 / GPT-5.1 / GPT-5-Codex.

### Recommended Preset for microkernelia
- **Architecture/README/design**: GPT-5 or Claude Opus 4.5.
- **Understanding repos (ES-KNN)**: GPT-4o.
- **Real code implementation**: GPT-5.1 + GPT-5-Codex.
- **Quick idea iteration**: Raptor mini or Grok Code Fast 1.