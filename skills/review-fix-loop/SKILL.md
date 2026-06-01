---
name: review-fix-loop
description: Iterative review-fix-commit cycle — reviews code, fixes findings, commits, and repeats until clean or iteration limit reached
---

Perform iterative review-fix-commit cycles. Each iteration finds issues,
fixes them, and commits the fixes. Stop early if a review finds zero findings.

The number of iterations defaults to 10. Pass a number as the argument to override:

  /review-fix-loop 5

Parse the iteration count from: $ARGUMENTS
If $ARGUMENTS is empty or not a positive integer, default to 10.

## For each iteration (1 through N):

### Step 1 — Review

Spawn a sub-agent to run the code review. The sub-agent should:

1. Invoke the `/full-review` skill.
2. After the review completes and findings are compiled, write the entire
   findings summary to `review.md` in the working directory. Include all
   findings organized by severity with file:line references.

Wait for the sub-agent to complete before proceeding.

### Step 2 — Fix

Read `review.md`. If it contains zero actionable findings at any severity level,
stop the loop early — the code is clean.

Otherwise, spawn a sub-agent to fix the findings. The sub-agent should:

1. Read `review.md`.
2. Evaluate each finding for merit — skip false positives and inapplicable findings.
3. Fix all legitimate findings by editing source files directly.
4. Write `fix.md` with a summary formatted as a git commit message:
   - First line: concise subject line (50 chars or less)
   - Blank line
   - Body: bullet points describing each fix with file references
   - If no findings had merit and no changes were made, write only:
     "No actionable findings — all items were false positives."

Wait for the sub-agent to complete before proceeding.

### Step 3 — Commit

Read `fix.md`.

- If `fix.md` indicates no actionable findings, skip the commit.
- Otherwise, check `git status`. If there are changes:
  - Stage all modified files (do NOT stage `review.md` or `fix.md`).
  - Commit using the content of `fix.md` as the commit message.

### Step 4 — Cleanup

Remove `review.md` and `fix.md`.

Report which iteration just completed and what was fixed (one line).

---

## Early exit

If Step 2 determines there are zero actionable findings, stop iterating.
Report that the codebase is clean and no further iterations are needed.

## Final summary

After all iterations complete (or early exit), report:

- Total iterations completed
- Number of commits created
- Summary of what was fixed across all cycles
