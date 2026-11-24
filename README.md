# MFC CI data

History written by the CI of [MFlowCode/MFC](https://github.com/MFlowCode/MFC) and read by the MFC website. Do not edit by hand.

## `benchmarks.jsonl`

One JSON object per line: one benchmark lane (machine, compiler, offload model) measuring one commit of `master`. It feeds the [performance history page](https://mflowcode.github.io/benchmarks.html).

| Field | Meaning |
|---|---|
| `sha`, `subject`, `pr`, `date` | The `master` commit measured, the pull request that produced it, and the commit date |
| `cluster`, `device`, `interface` | The lane, e.g. `frontier_amd`, `gpu`, `omp` |
| `cases.<name>.grind` | Grind time: nanoseconds per grid point, equation and Runge-Kutta stage |
| `cases.<name>.def` | Hash of the case's input file, arguments and benchmark flags; grind times are comparable only within one `def` |

Lines are appended by the `record` job of `.github/workflows/bench.yml` after each scheduled benchmark run.

Lines with a `measured` field were rebuilt from past pull-request benchmark runs, each of which also measured `master`; `measured` is when that run was made, so one commit can have several lines per lane. On the Frontier lanes of those lines, `sha` is the `master` commit current when the lane's job started (the result file did not record it). Results that a runner uploaded under the wrong lane were left out.
