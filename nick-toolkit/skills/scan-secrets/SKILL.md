---
name: scan-secrets
description: Read-only audit for leaked secrets in a file, directory, diff, or staged change. Wraps `gitleaks` if installed, otherwise does a regex sweep for common token patterns (AWS, GCP, GitHub, Slack, OpenAI, JWT, PEM). Reports findings with file/line and pattern name, always redacted. Never edits or removes. Use when the user says "/scan-secrets", "check for leaked secrets", "scan this for tokens", "did I leak anything", or before publishing/deploying a sensitive artifact.
---

# /scan-secrets

Read-only audit for leaked secrets. Reports findings, never edits.

## Inputs

Ask only what's missing:

1. **Target** — file path / directory / `--staged` (currently-staged changes) / `--diff <ref>` (diff from ref to HEAD). Default: current working tree at `.`.
2. **Mode** — `quick` (regex only) / `full` (gitleaks if available, else regex). Default: `full`.

## Algorithm

1. **Detect scanner.** `command -v gitleaks` to check. Prefer it when available.

2. **Run the scan.**
   - Directory or file with gitleaks: `gitleaks detect --source <path> --no-banner --redact --report-format json --report-path /tmp/gitleaks-<short-random>.json`
   - Staged with gitleaks: `gitleaks protect --staged --no-banner --redact --report-format json --report-path /tmp/...`
   - Diff with gitleaks: `git diff <ref> | gitleaks detect --no-git --pipe --no-banner --redact --report-format json --report-path /tmp/...`
   - Regex fallback (when gitleaks missing): `grep -rEn` for each pattern below across the target, skip `.git/`, `node_modules/`, build output. Aggregate hits.

3. **Patterns for the regex fallback** (extend as needed, keep narrow to limit false positives):
   - AWS access key: `AKIA[0-9A-Z]{16}`
   - AWS secret in env-style: `aws_secret_access_key\s*=\s*[A-Za-z0-9/+=]{40}`
   - GitHub token: `gh[pousr]_[A-Za-z0-9]{36,255}`
   - Slack token: `xox[abprs]-[A-Za-z0-9-]{10,}`
   - OpenAI: `sk-[A-Za-z0-9]{20,}`
   - JWT: `eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+`
   - GCP service account marker: `"private_key":\s*"-----BEGIN PRIVATE KEY-----`
   - PEM private key: `-----BEGIN (RSA |EC |DSA |OPENSSH |PGP |)PRIVATE KEY-----`
   - Generic high-entropy (last resort): `[A-Za-z0-9/+]{40,}={0,2}` — high false-positive rate, label as "low-confidence" in the report.

4. **Always redact in output.** If gitleaks returns a value, replace it with `[REDACTED:<pattern>]`. Never write the raw value to chat, even with `--no-redact`.

## Output

```
### Secret scan: <target>
Scanner:   <gitleaks vX.Y.Z | regex fallback>
Mode:      <quick | full>
Findings:  <N>

<file>:<line>  <pattern>  [REDACTED]
...

Verdict: <clean | N findings | scan failed>
```

If gitleaks isn't installed and the regex fallback ran:

```
Verdict: 0 findings in regex fallback. Install gitleaks for deeper coverage:
  brew install gitleaks       # macOS
  # or: https://github.com/gitleaks/gitleaks/releases
```

## Don'ts

- Don't print the secret value, ever, even with `--no-redact`.
- Don't edit, rewrite, or remove files. Pure read-only.
- Don't claim "clean" without naming what scanner ran. Distinguish "gitleaks clean" from "regex fallback clean".
- Don't scan `.git/` internals or build output.
- Don't suggest rotating a specific credential in the output. Recommend rotation as a class of action; the user owns the actual rotation.
- Don't write findings to a file. Output is chat only. (If the user explicitly asks for a report file, ask which path and confirm before writing.)
