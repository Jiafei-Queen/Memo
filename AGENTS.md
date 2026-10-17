# AGENTS.md

## Documentation

- Keep the README focused on an overview and basic usage.
- Put feature implementation details in the `docs/` directory, not in the README.
- Organize documents into appropriate subdirectories under `docs/`; avoid placing unrelated documents directly in its root.

## Project Structure

- Keep the project structure and module boundaries clear. Code readability and maintainability are top priorities.
- Before adding a feature, assess the existing project structure and coupling.
- If the assessment indicates that refactoring is necessary, ask the user for approval before proceeding.

## Environment Issues

- If you encounter environment issues, such as permission or network problems, stop work and report the issue to the user before proceeding.

## Testing

- After modifying code, run only the necessary tests and avoid time-consuming tests when they are not needed.
- Must run the complete test suite before committing.

## Code Changes

- Do not remove or substantially shorten existing project comments without a clear reason.
- Keep only necessary comments, and write them concisely.

## Pre-Commit Review

Before committing, check for:

- Unnecessary or overly long comments.
- Dead code.
- Documentation that needs to be updated.
- Documentation placed in the appropriate `docs/` subdirectory.
- Tests appropriate for the changes.

## Commit Messages

- First follow the commit style already used by the project.
- If the project has no commit history or no clear commit style, use the Conventional Commits specification.
- Keep commit messages concise.
- State only what changed and why.
- Do not include `Co-Authored-By`.
- Do not include unnecessary information, such as the number of tests that passed.
- Keep each line under 72 characters.

