# Repository Guidelines

This guide keeps contributions aligned with the Gemini 2.5 Deep Think prompt library. Keep edits scoped to the relevant prompt and confirm the Worker/Validator flow remains intact.

## Project Structure & Module Organization
- Root contains numbered prompt playbooks such as `01_Architecture_Evaluation.md` through `04_Bug_Check.md`; keep numbering two digits and increment sequentially when adding new prompts.
- `README.md` provides the Japanese catalog; update the index whenever prompts change.
- `LICENSE` must remain untouched; add assets only when they clearly strengthen prompt clarity, and keep them in the repository root unless a dedicated directory becomes necessary.

## Build, Test, and Development Commands
- There is no compile step; open Markdown locally to preview the rendered flow before submitting.
- Run `npx markdownlint@^0.33 "**/*.md"` to catch structural issues (install Node locally if needed).
- Use `rg "## "` to confirm section numbering and detect duplicate headings.

## Prompt Style & Naming Conventions
- Write prompts in Japanese with concise English glossaries where already used; preserve the Worker/Validator terminology exactly.
- Keep headings numbered (`## 0.`, `## 1.`) and retain the directive, QA-focused tone; add short rationale blurbs under each rule.
- Use fenced YAML blocks for output examples, ensuring top-level keys follow `findings`, `validation`, `summary`.

## Testing Guidelines
- Dry-run each prompt in Gemini 2.5 Deep Think with a realistic code sample and ensure the YAML schema validates (no extra keys, quoted scalars).
- Capture at least one Worker/Validator iteration in your notes and summarize the outcome in the PR description.
- When adjusting severity logic, cross-check against existing prompts so terminology and recommendation tiers stay consistent.

## Commit & Pull Request Guidelines
- Follow the existing imperative style (`Add`, `Update`, `Fix`); group related edits in a single commit when possible.
- PRs should link any tracking issue, list the prompts touched, and note validation steps (`npx markdownlint`, Gemini dry-run).
- Include screenshots or copied YAML only if they illustrate a changed outcome; otherwise keep the PR lightweight.
