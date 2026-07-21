---
type: daily
created: 2026-07-21
tags: []
reviewed:
---
# 2026-07-21 — First Contribution Experience

## Work Done
- Created my first pull request to understand the open-source contribution workflow.
- Forked a repository, made changes, committed them, pushed to my fork, and opened a PR.
- Investigated why the GitHub Actions workflow failed after the PR was created.
- Learned the difference between `pull_request` and `pull_request_target` workflows.

---

## Problems Encountered
- The repository's auto-merge GitHub Action failed during the checkout step.
- GitHub blocked the workflow because it attempted to check out forked code from a `pull_request_target` workflow.
- Initially assumed the issue was with my PR, but discovered it was a CI workflow/security configuration issue in the repository.

---

## Decisions Made

- No changes were made to my PR because the failure was not caused by my contribution.
- Continue practicing open-source contributions on beginner-friendly repositories.

---

## Ideas

- Learn GitHub Actions fundamentals and common CI workflows.
- Study GitHub security features such as `pull_request` vs `pull_request_target`.
- Build a small repository with my own GitHub Actions workflow to understand CI from the maintainer's perspective.

---

| `pull_request`                                 | `pull_request_target`                                    |
| ---------------------------------------------- | -------------------------------------------------------- |
| Runs **on the contributor's code**             | Runs **on the repository's code**                        |
| No access to repository secrets (for fork PRs) | Has access to repository secrets                         |
| Used to **test the PR**                        | Used to **manage the PR** (labels, comments, automation) |

### Example

Suppose you fork a repository and submit this PR:

```
main
└── app.py

Your PR
└── app.py   (you modified it)
```

### `pull_request`

GitHub says:

> "I'll run the workflow using **your modified `app.py`**."

```
Workflow
    │
    ▼
Checkout your PR code
    │
Run tests
```

✅ Good for testing whether your changes work.

---

### `pull_request_target`

GitHub says:

> "I'll run the workflow from the **repository's main branch**, not your PR."

```
Workflow
    │
    ▼
Use repository's workflow
    │
Comment on PR
Add labels
Assign reviewers
```

It **does not trust your code** because it has access to repository secrets.

---

### Your case

Your PR triggered:

```
pull_request_target
```

Then the workflow tried to do:

```
Checkout YOUR fork's code
```

GitHub stopped it because that would allow **untrusted code** to run with **trusted permissions**.

So it threw this error:

```
Refusing to check out fork pull request code...
```

---

### One-line memory trick

- **`pull_request`** → **Run my code** (safe, no secrets)
    
- **`pull_request_target`** → **Manage my PR** (has secrets, don't run my code)
    

That's the easiest way to remember the difference.
---
## Links

- https://github.com/firstcontributions/first-contributions
- https://github.com/firstcontributions/first-contributions/pulls