# HubVibe Site Compliance Audit

Runs a real, deterministic site audit against a live URL and fails the build
if it doesn't pass: WCAG 2.1 A/AA accessibility (axe-core), SEO, security
headers, and performance.

Every check runs against the actual page — axe-core for accessibility, a real
HTTP response for security headers, a real browser page load for performance.
Nothing here is an LLM guessing at quality, and a check that couldn't run is
never reported as a pass.

## Usage

```yaml
- name: HubVibe compliance audit
  uses: Its-fortunatefolly/hubvibe-audit-action@v1
  with:
    url: https://your-site.example.com
    api-key: ${{ secrets.HUBVIBE_API_KEY }}
```

Adopting it without letting a third-party outage block your deploys:

```yaml
- uses: Its-fortunatefolly/hubvibe-audit-action@v1
  with:
    url: https://staging.example.com
    api-key: ${{ secrets.HUBVIBE_API_KEY }}
    fail-on-error: false      # network/402/5xx warns instead of failing
    fail-on-violation: true   # real findings still gate the build
```

Findings are written to the job summary, so reviewers see the table in the
Checks tab rather than digging through raw logs.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `url` | yes | — | The live URL to audit. |
| `api-key` | no | — | HubVibe API key (`X-API-Key`). Without it the run reports the service's 402 challenge and how to pay. |
| `endpoint` | no | `bundle` | `wcag`, `seo`, `security`, `performance`, or `bundle`. |
| `fail-on-violation` | no | `true` | Set `false` to report findings without failing the build. |
| `fail-on-error` | no | `true` | Set `false` so infrastructure failures warn instead of failing. |
| `timeout-seconds` | no | `90` | Per-attempt HTTP timeout. |
| `retries` | no | `2` | Retries on network error or 5xx. 4xx is never retried. |
| `base-url` | no | the hosted service | Override for self-hosted deployments. |

## Outputs

| Output | Description |
|---|---|
| `passed` | `"true"` or `"false"` — the audit's overall result. |
| `response` | Raw JSON response body. |
| `http-status` | HTTP status of the final attempt (`000` if it never completed). |

## Pricing

$0.03 per single audit, $0.10 for the bundle. For CI, paying per call over
x402/MPP directly against the REST endpoints is usually the better fit — no
subscription key to store as a repository secret. See
[`/.well-known/agent.json`](https://hubvibe-831480473793.us-south1.run.app/.well-known/agent.json),
which is the only place guaranteed to match what checkout actually charges.

## Source

Developed in the [HubVibe monorepo](https://github.com/Its-fortunatefolly/HubVibe).
This repository is generated from it, because Marketplace requires an action
repository to contain no workflow files.
