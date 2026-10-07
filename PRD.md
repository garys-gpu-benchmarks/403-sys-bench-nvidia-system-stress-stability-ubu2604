# PRD.md:  "The Why"; Product requirements, benchmark metadata table, high-level requirements, etc.

Product Requirements Document

"The Why"; Product requirements, benchmark metadata table, high-level requirements, etc. Defines the benchmark goal, validation objective, test name, benchmark number, category, and high-level success criteria.

## Benchmark Matrix Document Metadata (via benchmark_specification.json)

This PRD.md section is populated from benchmark_specification.json, which is the structured source of benchmark-specific product requirements.

## Workload Number
403

## Workload Name
CPU + System Stress (Baseline Health)

## Execution Summary (Run and Measure)
Run bin/gpu_stress for yaml duration at gpu_load_percent (FP16 cuBLAS GEMM, busy for that percent of each 100 ms window) together with stress-ng --cpu --cpu-load --timeout, polling nvidia-smi temperature, power, utilization, thermal slowdown, and uncorrectable ECC each poll_interval, to measure combined CPU and GPU stress. DCGM and sensors are not run

## Main Goal
Analyze GPU thermal and power behavior under combined CPU and GPU load

## Validation Objective
Validates that stress-ng and the cuBLAS GEMM load (bin/gpu_stress) both run for the yaml duration and that peak GPU temperature, sustained (mean) GPU power, and uncorrectable GPU ECC errors are parsed from nvidia-smi samples taken every poll_interval seconds. The run fails if bin/gpu_stress does not start or report GEMMs, or if mean utilization.gpu is below half of gpu_load_percent (minimum 10%). Peak temperature above temp_threshold, mean power above power_threshold, or a hardware thermal slowdown event with throttle_check true marks every sample error

## Workload Category
System Validation & Reliability

## Validation Requirement

The benchmark must include an automated SQLite-integrated validation layer that verifies persisted results from `results/benchmark.db`. Validation must confirm:

1. The benchmark run completed successfully with no tool errors.
2. Required samples and aggregate metrics were persisted for every swept shape.
3. Metrics are finite and physically sensible (positive, within plausible bounds).
4. Measured values satisfy configured thresholds when the workload defines pass/fail gates.
5. The benchmark fails validation when required data is missing, invalid, or outside bounds.

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Automation | Runs to completion without manual intervention after `bash run_benchmark.sh` |
| Idempotency | Re-running `run_benchmark.sh` appends a new run; never corrupts existing rows |
| Persistence | All metrics survive script exit; `results/benchmark.db` is the durable record |
