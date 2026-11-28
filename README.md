# MFC CI data

History written by the CI of [MFlowCode/MFC](https://github.com/MFlowCode/MFC) and read by the MFC website. Do not edit by hand.

## `benchmarks.jsonl`

One JSON object per line: one benchmark lane (machine, compiler, offload model) measuring one commit of `master`. It feeds the [performance history page](https://mflowcode.github.io/dashboard/benchmarks.html).

| Field | Meaning |
|---|---|
| `sha`, `subject`, `pr`, `date` | The `master` commit measured, the pull request that produced it, and the commit date |
| `cluster`, `device`, `interface` | The lane, e.g. `frontier_amd`, `gpu`, `omp` |
| `hardware` | GPU model where a cluster has several (Phoenix: `h200`, `h100`, `a100`, `l40s`, `v100`); grind times differ several-fold between them |
| `covers` | Every commit since the previously benchmarked one, as `[sha, subject, pr]`; any of them can be behind a change |
| `env` | Fingerprint of what sets the lane's compiler and environment at that commit: the machine's lines of `toolchain/modules`, the commands `toolchain/bootstrap/modules.sh` runs for it, and its batch template (`environment()` in MFC's `.github/scripts/bench-record.py`); grind times are comparable only within one `env`. Lines before that rule were recomputed with `.github/scripts/bench-backfill.py` (then named `bench-env-backfill.py`) |
| `cases.<name>.grind` | Grind time: nanoseconds per grid point, equation and Runge-Kutta stage |
| `cases.<name>.def` | Hash of the case's input file, arguments and benchmark flags; grind times are comparable only within one `def` |
| `cases.<name>.cmd` | Hash of how `bench.py` ran the case: the `mfc.sh run` arguments it passed (including what it adds itself, such as `--targets` and `--gbpp`), the build configuration and the environment variables it set (`harness()` in `bench-record.py`); grind times are comparable only within one `cmd`. Lines recorded before it existed were rebuilt with `.github/scripts/bench-backfill.py` from the files at their commit |
| `cases.<name>.dflt` | Present when MFC defaults the case relies on changed since the lane's previous benchmark: `"name: old -> new"` for each input of simulation's `s_assign_default_values_to_user_inputs` that the case does not set and that no switch it leaves off gates (`default_changes()` in `bench-record.py`). A code change: flagged on the dashboard, not a break in the line |

Lines are appended by the `record` job of `.github/workflows/bench.yml` after each scheduled benchmark run.

Lines with a `measured` field were rebuilt from past pull-request benchmark runs, each of which also measured `master`; `measured` is when that run was made, so one commit can have several lines per lane. Their `sha` is the commit the job log says was checked out as `master`; where the log no longer says, it is the `master` commit current when the job started and the line carries `approx: true`. Results that a runner uploaded under the wrong lane were left out.

## `ci/<YYYY-MM>.jsonl`

One JSON object per line: one Test Suite or Benchmark run that had at least one job on a cluster, filed under the month the run was created. It feeds the [CI health](https://mflowcode.github.io/dashboard/ci.html) and [test health](https://mflowcode.github.io/dashboard/tests.html) pages.

| Field | Meaning |
|---|---|
| `run`, `workflow`, `event`, `date` | The run, `test` or `bench`, what triggered it, when it was created |
| `master`, `sha`, `title`, `pr` | Whether it ran on `master`, the commit, the run title, and its pull request |
| `merges` | For a `master` run, the merges it is the first to test, as `[sha, title, pr]`: a scheduled run covers every merge since the one before |
| `jobs[]` | One entry per cluster job: `l` lane, `k` kind (`test`, `case-opt`, `bench`), `s` shard, `m` `smoke` or `full`, `a` attempt, `c` conclusion, `w` seconds waiting for a runner, `d` seconds from start to finish, `id` GitHub job id, `u` its SLURM jobs as `[name, nodes, elapsed s, queued s, state]`; for a test job whose log was read: `n` tests passed, `tt` seconds the test phase took, `slow` the six slowest tests as `[uuid, name, seconds]`, `f` tests that failed and `r` tests that passed only on a retry, each `[uuid, name, error class, first line of the error]`; for any failed job whose log was read, `why` (`tests`, `build`, `cluster`, `queue`, `timeout`, `node`, `checkout`, `upload`, `runner`, `other`) and `e`, the line of the log that says how |

Lines are written by `.github/workflows/ci-record.yml` after each run; a re-run replaces its run's line. Lines from before that recorder have an empty `u`.

## `usage.jsonl`

SLURM accounting from before the recorder existed, one line per day, cluster and kind of job: `node_hours`, `jobs` and `queued_median` (seconds). The CI health page uses it for days that have no `u` entries.

## `builds.jsonl`

SLURM accounting from before the recorder existed: median seconds a Frontier build job ran, one line per day, lane and build variant. The CI health page uses it for days that have no `u` entries.

## `scaling.jsonl`

One line per weekly scaling run on Frontier (`scaling.yml` in MFC): the master commit and, per run, `kind` (`weak` or `strong`), `nodes`, `ranks` (GPUs), `cells`, `grind` (ns per grid point, equation and RK stage, over all ranks) and `step` (seconds per time step). Efficiency against the smallest run is `grind(smallest)·ranks(smallest)/(grind(N)·ranks(N))`.
