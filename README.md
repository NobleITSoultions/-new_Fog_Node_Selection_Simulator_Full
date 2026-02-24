# TALOFNS Reproducibility Package (IEEE-style)

This repository provides a **self-contained** Java simulator that reproduces the experimental protocol described in the TALOFNS paper:

- **Poisson task arrivals** (rate λ = 200 tasks/minute)
- **Random RTT latency** sampled uniformly in [1, 5] ms
- **Dynamic trust** using exponential update: `Trust(t)=μ*Trust(t-1)+(1-μ)*S(t)` where S(t)=1 for success, 0 for failure
- **Load-aware dynamics** with a CPU utilization model and queueing
- **Raw per-task logs** for each run (n=5), for each algorithm
- **Mean ± std**, paired t-tests, and Bonferroni correction computed from raw logs via Python

> Important: This simulator is designed for **reproducibility**, not for “forcing” the exact same numeric values as any specific table.  
> The qualitative trends should be consistent if parameters match the paper.

## Requirements
- Java 11+
- Maven 3.8+

For statistics:
- Python 3.9+
- `pandas`, `numpy`, `scipy` (optional but recommended)

## Quick start (run all algorithms, n=5)
```bash
mvn -q -DskipTests package
java -jar target/talofns-repro-sim-1.0.0.jar --out logs --runs 5 --seed 20260126
```

This produces:
- `logs/raw/<algorithm>/run_<k>.csv`  (per-task raw logs)
- `logs/summary_per_run.csv`          (per-run metrics)

## Compute IEEE-style statistics
```bash
python3 analysis/analyze_results.py --logdir logs --alpha 0.05
```

Outputs:
- `logs/table_mean_std.csv`
- `logs/talofns_vs_all_ttests.csv` (paired t-tests + Bonferroni)
- `logs/bonferroni_report.txt`

## Parameters (defaults from paper)
- Fog nodes: 6 (homogeneous)
- Duration: 1 hour (simulated)
- Arrival rate: 200 tasks/minute
- Trust init: uniform in [0.7, 1.0]
- Trust threshold θ: 0.7
- Load threshold Lmax: 0.8
- Composite weights: α = 0.6 (latency), β = 0.4 (trust)

You can override defaults via CLI flags. See:
```bash
java -jar target/talofns-repro-sim-1.0.0.jar --help
```

## Project structure
- `src/main/java/sim` ... simulator core
- `src/main/java/sim/policy` ... node selection algorithms
- `analysis/analyze_results.py` ... statistics from raw logs

## Notes on modeling choices
The paper specifies stochastic arrivals, latency sampling, trust update, threshold filtering, and composite scoring.
Where implementation details are underspecified (e.g., exact queueing model), the simulator uses simple, documented assumptions
that preserve the intended behavior and support reproducibility.

