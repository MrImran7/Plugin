# Using the GitHub Plugin with ChatGPT and Codex

This repository is a practical study guide for connecting GitHub to ChatGPT/Codex and using the integration throughout the software-development lifecycle.

> **Terminology:** This guide calls it the **GitHub plugin**, but the product may display it as a GitHub **app**, **connector**, or **integration**. It gives ChatGPT/Codex controlled access to repositories authorized by your GitHub account. It is different from developing a custom ChatGPT plugin.

## 1. What the integration provides

After GitHub is connected and a repository is authorized, ChatGPT/Codex can help with:

- Discovering repositories, branches, commits, issues, and pull requests
- Reading and explaining source code, README files, configuration, and documentation
- Searching code for symbols, APIs, bugs, security-sensitive operations, and architectural patterns
- Comparing branches or commits and summarizing changes
- Reviewing pull requests and their changed files
- Investigating CI workflow runs, jobs, steps, logs, and artifacts
- Creating and updating files, branches, commits, issues, pull requests, labels, assignees, and comments when write access is available
- Requesting reviewers, replying to review comments, and resolving review threads
- Running Codex tasks from pull-request comments or automated GitHub workflows

The exact operations available depend on the ChatGPT experience, workspace settings, GitHub installation, repository permissions, and the permissions granted for the current task.

## 2. Connect GitHub

1. Open ChatGPT or Codex settings.
2. Find **Apps**, **Connectors**, or **GitHub**.
3. Connect your GitHub account.
4. Install or authorize the GitHub app for the required account or organization.
5. Choose **Only select repositories** when possible and select the repositories needed.
6. Confirm the repository appears in ChatGPT/Codex.
7. Start with a harmless read request:

```text
List the branches in MrImran7/Plugin and summarize the README from develop.
```

If a repository is missing, verify that the GitHub app is installed for the correct account, that the repository is selected, and that organization policy permits access.

## 3. Access and credential model

The GitHub connection uses the authorization managed by ChatGPT and GitHub. Do not paste a GitHub password, personal access token, SSH private key, or OpenAI API key into a prompt or commit one to the repository.

Conceptually:

```text
User request
    |
    v
ChatGPT/Codex
    |
    | authorized GitHub operation
    v
GitHub integration
    |
    | repository-scoped permissions
    v
Selected GitHub repository
```

Important boundaries:

- ChatGPT/Codex can access only repositories exposed by the connected GitHub installation and allowed by the user's GitHub permissions.
- Read permission does not imply write permission.
- Organization administrators may restrict installation or repository access.
- Repository authorization and task approval are separate controls.
- Secrets required by CI belong in GitHub Actions secrets or an approved secret manager—not in source files.
- Review requested changes, target repository, and target branch before approving a write.

For the Codex GitHub Action, store the OpenAI API key as a GitHub Actions secret such as `OPENAI_API_KEY`. Pass it through the workflow's secret context and never print it in logs.

## 4. A safe working pattern

Use this sequence for repository changes:

1. **Inspect** — identify the repository, branch, relevant files, and existing conventions.
2. **Explain** — ask for the current behavior and evidence from code.
3. **Plan** — define the requested result, constraints, and tests.
4. **Implement** — make the smallest coherent change on a feature branch.
5. **Verify** — run or inspect tests, lint, builds, and CI.
6. **Review** — inspect the diff for correctness, security, and unintended changes.
7. **Commit** — use a focused commit message.
8. **Pull request** — summarize why, what changed, how it was tested, and remaining risks.
9. **Merge** — merge only after required checks and human approvals pass.

For production repositories, prefer branch protection and pull requests over direct commits to `main`.

## 5. Useful prompt patterns

Always include the repository and branch when they are not obvious.

### Understand a repository

```text
Analyze owner/repository on branch develop. Explain:
1. the main components,
2. program entry points,
3. request and data flow,
4. external dependencies,
5. how to build and test it.
Cite the relevant files and clearly label any inference.
```

### Trace one feature

```text
In owner/repository, trace authentication from the login UI to the backend.
Show where credentials enter, how requests are validated, where tokens are
created/stored/refreshed, and how logout or revocation works. Cite code and docs.
```

### Search for a symbol

```text
Find every definition and important call site of FooManager in owner/repository.
Explain ownership, threading assumptions, error handling, and lifecycle.
```

### Review a commit or branch

```text
Compare develop with main in owner/repository. Report behavior changes,
possible regressions, missing tests, security risks, and documentation gaps.
Prioritize findings by severity and cite exact files.
```

### Review a pull request

```text
Review PR #42 in owner/repository. Focus on correctness, concurrency,
resource leaks, security, compatibility, and test coverage. Do not summarize
the diff unless it helps explain a finding.
```

### Diagnose CI

```text
Inspect the latest failed workflow for PR #42 in owner/repository.
Identify the first actionable failure, distinguish root cause from cascading
errors, and propose the smallest fix. Do not rerun or modify anything yet.
```

### Implement a change

```text
On branch develop in owner/repository, implement <requirement>.
Preserve unrelated code, follow repository instructions, add/update tests,
run the relevant checks, and commit with a concise message. Do not merge.
```

### Create an issue

```text
Create a GitHub issue in owner/repository for <problem>. Include observed
behavior, expected behavior, reproduction steps, environment, acceptance
criteria, and suggested labels. Show me the draft before creating it.
```

### Prepare a pull request

```text
Create a pull request from feature/foo to develop in owner/repository.
Include the problem, solution, important design decisions, verification,
risk, rollback notes, and related issue. Do not merge it.
```

## 6. Pull-request workflows with Codex

When Codex code review is enabled for a repository, a pull-request comment can request a review:

```text
@codex review
```

A focused request is usually better:

```text
@codex review for concurrency errors, unsafe memory ownership, and missing tests
```

Where supported, request a deeper security-oriented review with:

```text
@codex security review
```

After a finding is posted, a follow-up can request a fix:

```text
@codex fix the P1 issue
```

Codex can also be configured to review eligible pull requests automatically. Automated review supplements tests, branch protection, and human approval; it does not replace them.

## 7. Repository instructions with AGENTS.md

An `AGENTS.md` file can provide persistent instructions to Codex. Put repository-wide guidance at the repository root and narrower guidance closer to the files it controls.

Example:

```markdown
# AGENTS.md

## Build and test

- Configure with: cmake -S . -B build
- Build with: cmake --build build
- Test with: ctest --test-dir build --output-on-failure

## Engineering rules

- Use C++17.
- Preserve backward compatibility for public headers.
- Add a regression test for every bug fix.
- Never commit credentials, generated build output, or device logs.

## Code Review Rules

- Flag unchecked buffer lengths at trust boundaries.
- Flag blocking work added to Android main-thread paths.
- Require cleanup for every acquired file descriptor and mapped buffer.
```

Keep instructions concise, testable, repository-specific, and version controlled. Leave deterministic formatting checks to CI.

## 8. GitHub issues and project coordination

The integration can reduce administrative work:

- Turn an error report into a structured issue.
- Search for related or duplicate issues before creating a new one.
- Add labels and assignees after verifying the intended values.
- Summarize a long issue discussion and list unresolved decisions.
- Convert agreed acceptance criteria into a development checklist.
- Link code changes and pull requests back to the issue.
- Draft release notes from merged pull requests or a commit range.

Example:

```text
Search open issues in owner/repository for reports related to NPU inference
freezing after thermal events. Group likely duplicates and summarize the
evidence, affected versions, and missing diagnostic data.
```

## 9. CI/CD automation with the Codex GitHub Action

The official `openai/codex-action@v1` can run repeatable Codex tasks in GitHub Actions. Common uses include pull-request feedback, migration assistance, release preparation, and policy checks.

Minimal structure:

```yaml
name: Codex review

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: openai/codex-action@v1
        with:
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
          prompt-file: .github/codex/prompts/review.md
          sandbox: read-only
```

Security recommendations:

- Give the workflow the minimum GitHub permissions it needs.
- Use `persist-credentials: false` when checkout credentials are unnecessary.
- Prefer `read-only` or `workspace-write` sandboxing over unrestricted execution.
- Treat pull-request content as untrusted input.
- Avoid exposing secrets to workflows triggered from untrusted forks.
- Pin and review third-party actions according to your organization's supply-chain policy.
- Separate analysis from jobs that post comments or modify repository content.

## 10. High-value study exercises

Use this repository to practice incrementally:

1. Add an `AGENTS.md` with build, test, and review rules.
2. Ask ChatGPT to explain the repository and identify missing documentation.
3. Create a feature branch and add a small sample program.
4. Ask ChatGPT to create tests and commit them to that branch.
5. Open a pull request into `develop`.
6. Request `@codex review` and address the findings.
7. Add a simple GitHub Actions build/test workflow.
8. Intentionally break a test and ask ChatGPT to diagnose the CI failure.
9. Create an issue from the failure and link the repair pull request.
10. Compare `develop` with `main` and generate release notes.

## 11. Limits and verification

ChatGPT/Codex can make mistakes or lack necessary repository context. Always verify:

- Repository and branch names before writes
- The complete diff before committing or merging
- Test and CI results rather than relying only on a summary
- Security-sensitive changes with appropriate human review
- Generated commands before running them in valuable environments
- Claims against source code, repository documentation, and authoritative product documentation

Some operations may not be exposed in every ChatGPT interface even when GitHub is connected. For example, account-level or repository-settings operations may still require the GitHub website.

## 12. Official documentation

- [Connect and use GitHub with Codex](https://learn.chatgpt.com/docs/third-party/github)
- [Codex cloud](https://learn.chatgpt.com/docs/cloud)
- [Code review](https://learn.chatgpt.com/docs/code-review)
- [Codex GitHub Action](https://learn.chatgpt.com/docs/github-action)

---

The best results come from precise prompts: name the repository, branch or pull request, desired outcome, constraints, verification steps, and whether write operations are allowed.
