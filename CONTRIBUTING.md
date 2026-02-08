# Contributing Guide

## 1. Workflow
- Work only via Pull Requests (PRs).
- Branch from `main`, keep branches short-lived.
- Keep history clean: prefer squash merge for PRs.

## 2. Commit Rules (Conventional Commits)
Format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: feat, fix, docs, refactor, perf, test, chore, ci, build, revert

Rules:
- Subject <= 72 chars, no trailing period.
- Imperative mood: "add", "fix", "remove", "update".
- Body explains what and why (not how).
- Footer references task IDs (e.g., PROJ-123).
- If no task: `No-Issue: <reason>`.

Examples:
```
feat(api): add user search endpoint

Add search by email and phone for admin panel.

Refs: PROJ-123
```
```
fix(auth): handle expired refresh tokens

Avoid infinite refresh loop on 401.

BREAKING CHANGE: refresh token TTL changed to 7d
```

## 3. PR Requirements
- Clear description and context.
- Link to task/issue.
- Tests executed (or explain why not).
- Documentation updated in the same PR.

## 4. Documentation Rules
See `dev/docs/osnova/documentation_recommendations.md`.

## 5. Code Owners
See `CODEOWNERS` for review ownership.
