# swift-lint-config

Centralized SwiftLint configuration shared across Izzy's Swift repos, so
review comments about the same handful of recurring style issues stop
getting repeated PR after PR, repo after repo.

## What this is

This repo's `.swiftlint.yml` is the single canonical SwiftLint config. It is
**consumed remotely**, not copied, by:

- [`izzynavedo-org/Rotation`](https://github.com/izzynavedo-org/Rotation)
- [`izzynavedo-org/account-sync`](https://github.com/izzynavedo-org/account-sync)
- [`izzynavedo-org/pr-review-dispatcher`](https://github.com/izzynavedo-org/pr-review-dispatcher)

Each of those repos has a one-line `.swiftlint.yml` at its root:

```yaml
parent_config: https://raw.githubusercontent.com/izzynavedo-org/swift-lint-config/main/.swiftlint.yml
```

SwiftLint fetches that URL fresh on every run (falling back to a local cache
only if the fetch fails/times out), merges it in as the parent configuration,
and applies all of its rules.

## ⚠️ No version pinning — edits to `main` are live immediately

There is currently **no commit-SHA pinning** anywhere. The URL above points at
the `main` branch, so **any edit merged to `main` in this repo takes effect on
the very next CI run in Rotation, account-sync, AND pr-review-dispatcher,
simultaneously** — there is no staged rollout, no per-repo opt-in, and no
review gate on the consumer side. A bad edit here (a typo'd regex, an overly
aggressive severity bump, a syntax error) can break CI in all three repos at
once.

Because of that:

- Treat changes to `.swiftlint.yml` in this repo as you would any shared
  production config: review carefully, verify locally with `swiftlint lint`
  against real fixtures before merging, and merge only clean, well-tested
  changes.
- If you need a change that only some consumers should pick up (or want to
  stage a risky change safely), you can pin a consumer repo's `parent_config`
  to a specific commit SHA instead of `main`, e.g.:

  ```yaml
  parent_config: https://raw.githubusercontent.com/izzynavedo-org/swift-lint-config/<commit-sha>/.swiftlint.yml
  ```

  This is **available but not set up yet** — as of now all three consumers
  point at `main` for convenience. Pinning trades that convenience (config
  changes propagate automatically) for safety (a repo only updates when its
  `parent_config` line is deliberately bumped). Switch a repo to a pinned SHA
  if/when you want that tradeoff.

## Rule provenance

The rules in `.swiftlint.yml` encode real, recurring PR review comments from
Izzy's actual review history (not generic style-guide guessing). See the
comments inline in `.swiftlint.yml` for which rule maps to which recurring
comment. A few review comments don't have a reliable lint-rule equivalent
(too high false-positive risk, or no clean built-in/regex fit) — those are
documented as prose guidance in each consumer repo's `CONTRIBUTING.md`
instead.

## Custom rule severities

All `custom_rules` in this config are set to `error`. They are new rules with
no existing-violation backlog in any consuming repo as of this config's
introduction, so there's no reason to soften them to `warning`. Built-in
rules that are prone to noisy pre-existing violations in real codebases
(`line_length`, `function_body_length`, `type_body_length`, `file_length`,
`cyclomatic_complexity`) are kept at `warning` (with generous thresholds)
rather than `error`, so CI doesn't start red on day one for large existing
files.
