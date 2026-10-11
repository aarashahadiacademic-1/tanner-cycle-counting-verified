# Tanner graph cycle counting — verified benchmark snapshot

This repository preserves the exact code-file contents and directory/file ordering of the audited `tanner-cycle-counting-2026` snapshot. The transfer was checked by comparing Git blob SHA-1 for every copied file; the source algorithms were not edited during migration.

## Source and data

- `src/bench_core.cpp`: exact counting and independently coded Karimi–Banihashemi (KB) baseline.
- `src/chunk_hybrid.cpp`: single-processor hybrid using conservative bounded BFS and residual-DFS microbatches. It **does not call `step()` for every operation**.
- `src/exact_benchmark.cpp`: same-target exact versus KB comparisons.
- `graphs/*.edges`: 12 fixed non-QC benchmark graphs. Q410 and Q412 are deterministically created by `scripts/generate_core.py`.
- `scripts/generate_core.py` and `scripts/generate_diverse.py`: fixed graph generators.
- `scripts/run_all.py`: checks every graph SHA-256 and reruns the two experiment tables.
- `results/table1_exact_7runs.csv`: ten rows, Exact and KB median wall times over seven runs.
- `results/table2_hybrid_100runs.csv`: eleven rows, Exact/KB medians over seven runs; Hybrid mean runtime and MARE over 100 runs.

## Reproduction

Requires Python 3.8+ and a C++17 compiler. From the repository root:

    python3 scripts/generate_core.py
    python3 scripts/generate_diverse.py
    g++ -std=c++17 -O3 src/exact_benchmark.cpp -o exact_benchmark
    g++ -std=c++17 -O3 src/chunk_hybrid.cpp -o chunk_hybrid
    python3 scripts/run_all.py

The scripts verify exact input SHA-256 hashes. Measurement runtime depends on machine load. Recorded conditions used `g++ 14.2 -O3`, r=101, epsilon=0.08, Q=4096 weighted credits, 7 baseline repetitions, and 100 hybrid trials, with the fixed random seeds embedded in the script.

## Scope of the verification

All eleven recorded hybrid rows have Hybrid/standalone-Exact elapsed-time ratios below 2.0 for **this benchmark run** (maximum about 1.98 on A128r). This is an empirical observation, not a proof or a guarantee for different machines, graphs, seeds, or sizes. Other experiments can exceed 2–3 times standalone Exact.

For Q=4096 on the listed benchmark graphs, conservative upper bounds ensure that complete BFS or per-boundary DFS microtasks fit the remaining credits; the implementation refuses Q values too small for an atomic microtask. This does not establish an arbitrary-Q implementation of the full theoretical RAM scheduler. The accounting is a documented implementation credit model, not a literal count of CPU instructions. Wall-clock runtime is not covered by the asymptotic work theorem.

Cycle counts from the recorded Exact and KB runs agree. The two Q generators have been checked against the recorded SHA-256 hashes. A complete clean-room end-to-end reproduction on a second machine has **not** been certified by this README.

Graph construction and correctness checking are not timed. Graphs are synthetic Tanner graphs, not standardized LDPC codes.
