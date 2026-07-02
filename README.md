# VibeScan GitHub Action

Automated security scanning for vibe-coded apps — Lovable, Bolt.new,
v0, and Replit — plus mobile, LLM/AI, and cloud-native checks.

## Validated, not just built

This scanner was tested against 91 real-world repositories — 19
OWASP Benchmark / documented-CVE projects, 69 real vibe-coded apps,
and 3 popular open-source boilerplates used as a false-positive
stress test:

- **17/17** ground-truth vulnerabilities correctly detected
- **0/3** false positives on real, well-maintained boilerplate code
- **3 systemic bugs found and fixed** during validation — including
  a rule that initially flagged 22 false "critical" findings on a
  real open-source project before being rewritten

## Quick start

1. Get a free API key: [vibecodescan.vercel.app/account/keys](https://vibecodescan.vercel.app/account/keys)
2. Add it as a repository secret: `VIBESCAN_API_KEY`
3. Add this workflow:

```yaml
name: Security Check
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: vibescan/vibescan-action@v1
        with:
          api-key: ${{ secrets.VIBESCAN_API_KEY }}
          platform: lovable   # or bolt, v0, replit, auto
```

## What it checks

- Exposed API keys (Stripe, OpenAI, Supabase service_role, AWS, and 30+ others)
- CVE-2025-29927 (Next.js middleware auth bypass) and
  CVE-2025-55182 (React2Shell RCE) — version-checked against patched thresholds
- Missing Supabase Row Level Security
- Unauthenticated API routes (correctly recognizes abstracted
  auth helpers like `getUserFromCookie()`, `requireAuth()`, etc. —
  not just raw `cookies().get()` calls)
- Prototype pollution (key-provenance aware — won't flag safe local
  object copies or DOM/React property assignment)
- Mobile, LLM/AI agent, and cloud/infra-specific checks (conditionally
  activated based on detected project type)

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | ✅ | — | VibeScan API key |
| `platform` | ❌ | `auto` | `lovable`, `bolt`, `v0`, `replit`, or `auto` |
| `fail-on-critical` | ❌ | `true` | Fail the workflow if critical issues found |
| `fail-on-important` | ❌ | `false` | Fail if important issues found |
| `comment-on-pr` | ❌ | `true` | Post a results comment on PRs |
| `github-token` | ❌ | `${{ github.token }}` | For private repos and PR comments |

## Outputs

| Output | Description |
|---|---|
| `score` | Security score A–F |
| `critical-count` | Number of critical findings |
| `total-findings` | Total findings across all severities |
| `report-url` | Link to the full interactive report |

## Example: use outputs in a downstream step

```yaml
- uses: vibescan/vibescan-action@v1
  id: vibescan
  with:
    api-key: ${{ secrets.VIBESCAN_API_KEY }}

- name: Print score
  run: echo "Score is ${{ steps.vibescan.outputs.score }}"
```

## Advanced: don't fail, just report

```yaml
- uses: vibescan/vibescan-action@v1
  with:
    api-key: ${{ secrets.VIBESCAN_API_KEY }}
    fail-on-critical: 'false'
    fail-on-important: 'false'
```

## License

MIT
