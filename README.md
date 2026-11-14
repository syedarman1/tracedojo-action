# TraceDojo agent test

Test how an AI agent recovers when its tools fail. This GitHub Action runs a
TraceDojo workflow against your trusted adapter, records the result, and fails the
job when outcome checks fail or the evidence is incomplete.

This Action has no baseline. A pull request that fails the same checks as its
base branch fails here too. To fail only on new regressions, use the workflow
from `npx tracedojo init --ci` instead. That gate compares each PR with the
exact base commit's saved report, passes PRs that only repeat the base's
failures, and fails PRs as incomplete until the base branch has saved a report.

## Usage

Use Node.js 22.23.1 or later. CI tests Node 22.23.1 and Node 24 on Linux.
Install `@tracedojo/sdk` as an exact dev dependency in your application and commit
your lockfile. Create a workflow and adapter with `npx tracedojo init`.

```yaml
name: Agent tests
on: [push, pull_request]
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
      - run: npm ci --ignore-scripts
      - uses: syedarman1/tracedojo-action@v1.0.0
        with:
          config: dojo/workflow.json
          adapter: dojo/adapter.mjs
          trials: '3'
          report: .tracedojo/pr-report.json
```

Pin actions to reviewed full commit SHAs in production. If your application needs
build steps, run those before the action. The action uses the SDK installed in
your checkout; it does not install dependencies, upload reports, or contact the
hosted TraceDojo dashboard.

## Inputs and output

| Input | Default | Meaning |
| --- | --- | --- |
| `config` | `dojo/workflow.json` | Workflow JSON inside the checkout |
| `adapter` | `dojo/adapter.mjs` | Trusted agent adapter inside the checkout |
| `trials` | `3` | Independent trials per fault, 1–20 |
| `report` | `.tracedojo/action-report.json` | New report path; existing files are never overwritten |

The `report` output contains the absolute requested report path. It may be absent,
or point to a file that was not created, when setup or execution fails.

Exit codes: **0** passes, **1** failed assertions, **2** invalid setup or execution
error, **3** incomplete evidence. A failed test can still leave a report on the
runner; saving it as an Actions artifact is your choice. Review traces before
sharing them.

## Scope and safety

This action runs one suite. For comparisons against the exact base commit and
sticky pull-request comments, use the SDK's `npx tracedojo init --ci` workflows.

Adapters execute local code with the runner's permissions. Never expose model
credentials to untrusted pull requests. Model-backed adapters may incur provider
charges; provider calls come from your adapter. Only run trusted adapters and
choose an appropriate provider request budget.

Passing tests establish behavior under the simulated tools and checks you define.
They do not certify production reliability or security.

## Development and license

Run `npm ci --ignore-scripts` and `npm test`. Tests use the published SDK and
scripted fixtures with no model calls. The initial action code is extracted from
[TraceDojo SDK commit 5d7aa4f](https://github.com/syedarman1/tracedojo-sdk/commit/5d7aa4f836cdfb4284f902a9a1107f906c6d8ec5).
Code and tests are Apache-2.0; see [LICENSE](LICENSE).

[SDK source](https://github.com/syedarman1/tracedojo-sdk) ·
[Documentation](https://tracedojo.com/docs) ·
[Security](https://tracedojo.com/docs/security) ·
[Limitations](https://tracedojo.com/docs/limitations)
