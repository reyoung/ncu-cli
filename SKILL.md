---
name: ncu-cli
description: Use when installing or using the local ncu-cli tool from ref/ncu-cli to analyze NVIDIA Nsight Compute CSV exports, compare CUDA kernel profiles, or run ncu-cli built-in analyzers such as roofline, memory, occupancy, instruction, warp_stall, launch_config, and arch.
---

# ncu-cli

Use this skill when the task involves diagnosing CUDA kernel performance from
Nsight Compute CSV output with the local `ncu-cli` binary or source checkout.

## Install

Prefer the local checkout in the current repo:

```bash
cargo install --path ref/ncu-cli --root "$HOME/.local" --locked --force
```

Then verify:

```bash
ncu-cli --version
ncu-cli --help
```

If the user asks to install from the upstream project instead, use
`ref/ncu-cli/install.sh` or the README one-line installer, but avoid that for
repo-local changes because it reclones the upstream repository.

## Inputs

`ncu-cli` expects an Nsight Compute CSV export. If the user already has a CSV,
use that file directly. If a profile must be generated, run the user's exact
workload under `ncu` and save CSV output, for example:

```bash
ncu --csv --page raw ./your_cuda_app > profile.csv
```

For local GPU workloads, follow the host or repo GPU lease instructions when
launching the profiled command.

## Core Workflow

1. Inspect metadata first:

   ```bash
   ncu-cli info profile.csv
   ncu-cli summary profile.csv
   ```

2. Run full diagnostics on the relevant kernel:

   ```bash
   ncu-cli analyze profile.csv
   ncu-cli analyze profile.csv --kernel decode_attention
   ```

3. Save machine-readable or report output when the result will be reused:

   ```bash
   ncu-cli analyze profile.csv --kernel decode_attention --format markdown -o ncu.md
   ncu-cli export profile.csv --format json -o kernels.json
   ```

4. Compare before and after profiles for optimization work:

   ```bash
   ncu-cli diff before.csv after.csv
   ncu-cli diff before.csv after.csv --format markdown -o ncu-diff.md
   ```

## Built-In Analyzers

Use individual analyzers when the user asks about a specific bottleneck:

```bash
ncu-cli skill list
ncu-cli skill run roofline profile.csv
ncu-cli skill run memory profile.csv --kernel gemm
ncu-cli skill run occupancy profile.csv --format json
ncu-cli skill run warp_stall profile.csv --kernel decode
```

Available analyzer names are:

- `roofline`: compute, memory, balanced, or latency-bound classification.
- `memory`: coalescing, L1/L2 hit rates, bank conflicts, and DRAM pressure.
- `occupancy`: active warps, theoretical-vs-achieved occupancy, and spills.
- `instruction`: tensor core use, instruction mix, and divergence signals.
- `warp_stall`: dominant PC-sampling stall reasons and targeted actions.
- `launch_config`: occupancy limiters from registers, shared memory, and warps.
- `arch`: Ampere, Hopper, and Blackwell-specific advice.

## Reporting

When summarizing results, include the profile path, selected kernel, GPU/SM
architecture if available, main bottleneck, top critical or warning findings,
and the exact command used. For optimization claims, prefer `ncu-cli diff`
between before and after CSVs over comparing prose reports manually.
