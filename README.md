# MFC CI data

History written by the CI of [MFlowCode/MFC](https://github.com/MFlowCode/MFC) and read by the MFC website. Do not edit by hand.

## `benchmarks.jsonl`

One JSON object per line: one benchmark lane (machine, compiler, offload model) measuring one commit of `master`. It feeds the [performance history page](https://mflowcode.github.io/benchmarks.html).

| Field | Meaning |
|---|---|
| `sha`, `subject`, `pr`, `date` | The `master` commit measured, the pull request that produced it, and the commit date |
| `cluster`, `device`, `interface` | The lane, e.g. `frontier_amd`, `gpu`, `omp` |
| `hardware` | GPU model where a cluster has several (Phoenix: `h200`, `h100`, `a100`, `l40s`, `v100`); grind times differ several-fold between them |
| `covers` | Every commit since the previously benchmarked one, as `[sha, subject, pr]`; any of them can be behind a change |
| `cases.<name>.grind` | Grind time: nanoseconds per grid point, equation and Runge-Kutta stage |
| `cases.<name>.def` | Hash of the case's input file, arguments and benchmark flags; grind times are comparable only within one `def` |

Lines are appended by the `record` job of `.github/workflows/bench.yml` after each scheduled benchmark run.

Lines with a `measured` field were rebuilt from past pull-request benchmark runs, each of which also measured `master`; `measured` is when that run was made, so one commit can have several lines per lane. On the Frontier lanes of those lines, `sha` is the `master` commit current when the lane's job started (the result file did not record it). Results that a runner uploaded under the wrong lane were left out.

## `ci/<YYYY-MM>.jsonl`

One JSON object per line: one Test Suite or Benchmark run that had at least one job on a cluster, filed under the month the run was created. It feeds the [CI health page](https://mflowcode.github.io/ci.html).

| Field | Meaning |
|---|---|
| `run`, `workflow`, `event`, `date` | The run, `test` or `bench`, what triggered it, when it was created |
| `master`, `sha`, `title`, `pr` | Whether it ran on `master`, the commit, the run title, and for a `master` commit its pull request |
| `merges` | For a `master` run, the merges it is the first to test, as `[sha, title, pr]`: a scheduled run covers every merge since the one before |
| `jobs[]` | One entry per cluster job: `l` lane, `k` kind (`test`, `case-opt`, `bench`), `s` shard, `m` `smoke` or `full`, `a` attempt, `c` conclusion, `w` seconds waiting for a runner, `d` seconds from start to finish, `id` GitHub job id, `u` its SLURM jobs as `[name, nodes, elapsed s, queued s, state]` |

Lines are written by `.github/workflows/ci-record.yml` after each run; a re-run replaces its run's line. Lines from before that recorder have an empty `u`.

## `usage.jsonl`

SLURM accounting from before the recorder existed, one line per day, cluster and kind of job: `node_hours`, `jobs` and `queued_median` (seconds). The CI health page uses it for days that have no `u` entries.
