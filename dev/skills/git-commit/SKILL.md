---
name: git-commit
description: Creates well-formatted git commits following conventional commit standards. Use after completing a task or producing an artifact. Generates commit messages in Chinese.
---

## Usage
```
/commit
```

## Behavior
1. Analyze staged changes with `git diff --staged`
2. Generate a conventional commit message in chinese
3. Create the commit with proper formatting

## Commit Format
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

## Types
- feat: New feature
- fix: Bug fix
- docs: Documentation changes
- style: Code style changes
- refactor: Code refactoring
- test: Adding or modifying tests
- chore: Maintenance tasks

## Example Output
```
feat(auth): add password reset functionality

- Add forgot password form
- Implement email verification flow
- Add password reset endpoint
```