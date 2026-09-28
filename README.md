# github-metrics-report

`ghperf` reports one person's contributions to one GitHub organization over a
time window, as markdown tables on stdout. It exists to make performance
self-reporting a command rather than an afternoon of clicking through GitHub.

Everything is derived from the GitHub GraphQL API. Nothing is cloned, and
nothing is counted by hand -- except "projects owned / maintained", which is
emitted as `TBD -- requires manual entry` because no API field answers it
honestly.

## Dependencies

| Tool                 | Minimum | Required | Used for                                       |
| -------------------- | ------- | -------- | ---------------------------------------------- |
| bash                 | 4.0     | yes      | --                                             |
| curl                 | 7.20    | yes      | HTTP transport                                 |
| jq                   | 1.5     | yes      | request construction and aggregation           |
| GNU coreutils `date` | 8.0     | yes      | date specs; BSD/macOS `date` will not work     |
| git                  | 2.0     | no       | source line at the top of the report           |
| gh                   | 2.0     | no       | credentials, if no token is in the environment |

Developed against bash 5.2, curl 7.88, jq 1.6, coreutils 9.1.

## Authentication

First match wins:

1. `gh`, if installed and authenticated
2. `$GITHUB_TOKEN`
3. `$GH_TOKEN`

The token needs `read:org`, plus `repo` to see private repositories. There is no
`--token` flag, because arguments are visible to `ps`.

## Usage

```sh
ghperf --org OpenCHAMI                                    # all time
ghperf --org OpenCHAMI --since 2026-01-01                 # year to date
ghperf --org OpenCHAMI --since -90d --until now           # last 90 days
ghperf --org OpenCHAMI --user someone --since "1 year ago"
ghperf --org OpenCHAMI --since 2026-01-01 > review.md
```

`--org` is required. `--user` defaults to the authenticated account.

Dates use journalctl syntax: `2026-01-01`, `"2026-01-01 10:30:00"`, `yesterday`,
`today`, `now`, `-90d`, `"2 weeks ago"`, `@1700000000`.

## Exit codes

| Code | Meaning                                        |
| ---- | ---------------------------------------------- |
| 0    | Report written to stdout                       |
| 1    | Usage error, or unknown organization or user   |
| 2    | No usable credentials                          |
| 3    | API failure, or a pagination assertion tripped |

Diagnostics go to stderr, so stdout stays pipeable. A window containing no
activity is not an error; it reports zeros.
