# General Principles

Follow these principles across all languages and frameworks to ensure clean, maintainable, and efficient code.

## Design Patterns

- Don't Repeat Yourself (DRY)
- Keep It Simple, Stupid (KISS)
- You Aren't Gonna Need It (YAGNI)
- Single Responsibility Principle (SRP)
- Separation of Concerns (SoC)

## Code Style

- No comments in code - use descriptive names instead
- Extract magic strings/numbers into constants
- Use descriptive function names over inline logic
- Always use global defined theme colors

## Workflow

- Make changes only, no explanations
- Provide text explanations only if explicitly requested

## README Maintenance

- Whenever a workflow file under `.github/workflows/` is added, modified, or removed, update `README.md` accordingly.
- Each workflow must have a section under `## Reusable Workflows` documenting its name, purpose, inputs, outputs, secrets, and a minimal usage example.
- Keep sections grouped by concern in this order: **Versioning** → **Build & Test** → **Deployment**. Within each group, order alphabetically by technology.

