# MFC CI data

History written by the CI of [MFlowCode/MFC](https://github.com/MFlowCode/MFC) and read by the MFC website. Do not edit by hand.

## `benchmarks.jsonl`

One JSON object per line: one benchmark lane (machine, compiler, offload model) measuring one commit of `master`. It feeds the [performance history page](https://mflowcode.github.io/benchmarks.html).

| Field | Meaning |
|---|---|
| `sha`, `subject`, `date` | The `master` commit measured and when |
| `cluster`, `device`, `interface` | The lane, e.g. `frontier_amd`, `gpu`, `omp` |
| `cases.<name>.grind` | Grind time: nanoseconds per grid point, equation and Runge-Kutta stage |
| `cases.<name>.def` | Hash of the case's input file, arguments and benchmark flags; grind times are comparable only within one `def` |

Lines are appended by the `record` job of `.github/workflows/bench.yml` after each scheduled benchmark run.

Lines dated before the recorder went live were rebuilt from past pull-request benchmark runs, each of which also measured `master`. For those, `date` is when the run was created, and on the Frontier lanes `sha` is the `master` commit current when the lane's job started (the result file did not record it).
