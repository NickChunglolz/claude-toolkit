---
name: verify-regression
description: Prove that a bug-fix test is a real regression test. Runs the new test against the OLD code (must FAIL) and the NEW code (must PASS). If the test passes on old code, it doesn't actually catch the bug. Use when the user says "verify the regression", "/verify-regression", "did my test actually catch the bug", "is this test useful", or right after writing a test for a bug fix.
---

# /verify-regression

A regression test that passes on the broken code is not a regression test. This skill proves the new test fails before the fix and passes after.

## Inputs

Ask only what's missing:

1. **Test selector** — file path plus test name or pattern. Examples: `tests/test_cart.py::test_discount_negative_qty`, `packages/api/cart.test.ts -t "negative qty"`.
2. **Old ref** — the commit before the fix. Default: if HEAD is the fix commit, use `HEAD~1`. If on a feature branch, use `git merge-base HEAD main` (or the repo's default branch).
3. **Test runner** — auto-detect from repo files (pytest, jest, vitest, go test, bun test, cargo test). Confirm only if ambiguous.

## Algorithm

1. **Pre-checks.**
   - Working tree must be clean enough that the fix is committed (or fully staged). If dirty in ways that touch the test or the file under test, ask the user to commit/stash first.
   - Confirm the test file actually exists at HEAD.

2. **Create an old-ref worktree.**
   - `git worktree add ../.verify-regression-<short-sha> <old-ref>` (use a hidden-ish path so it's obvious it's temporary).
   - If `graft` is installed (`command -v graft`), prefer `graft new` consistent with the rest of the toolkit.

3. **Drop the new test into the old worktree.**
   - `git show HEAD:<test-file> > <worktree>/<test-file>` for each test file involved.
   - If the test imports helpers, fixtures, or factories that don't exist at the old ref, the test will fail on import. That's a false FAIL. Detect it (look for ImportError / ModuleNotFoundError / cannot find module in the output) and report that the test depends on new infrastructure, not the bug fix.

4. **Run the test in the old worktree. Expect FAIL.**
   - PASS on old code means the test does not catch the bug. Stop. Report. Suggest the test is asserting on the wrong invariant, mocking the broken path, or that the bug is environmental (config or data, not code).
   - FAIL on old code is the good case. Proceed.

5. **Run the test in the main checkout. Expect PASS.**
   - FAIL means the fix isn't actually fixing it, or the test setup differs between worktrees. Stop and report.

6. **Clean up.** Always remove the worktree, even on failure. `git worktree remove <path>` or `graft rm`.

## Output

Success:

```
### Regression check: <test>
Old ref:  <sha> <subject>
Old code: FAIL (test catches the bug, good)
New code: PASS (fix works)
Verdict:  real regression test.
```

Failure (old code passed):

```
### Regression check: <test>
Old ref:  <sha>
Old code: PASS
Verdict:  not a real regression test. Likely causes: asserts the wrong invariant, mocks the broken code path, or the bug is environmental (config/data) not in code.
```

Failure (import error on old code):

```
### Regression check: <test>
Old ref:  <sha>
Old code: ERROR — test imports symbols that don't exist at old ref.
Verdict:  cannot verify. The test depends on new infrastructure. Either rewrite the test to use only old-API surface, or accept that the regression is "this couldn't even be expressed before".
```

## Don'ts

- Don't run on a dirty working tree. False signal.
- Don't run the full test suite. Only the targeted test.
- Don't leave worktrees behind. Clean up on every exit path.
- Don't claim a test is a regression test if the old-code-FAIL step was skipped or errored.
- Don't write or modify tests. Read-only on test content; the only writes are temporary worktrees that get removed.
