# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load. gpu_load_percent: share of each 100 ms window the GPU runs FP16 GEMMs (default 50). Sweep dimensions: num_threads, cpu_load_percent, gpu_load_percent, temp_threshold, power_threshold, throttle_check, ecc_check, poll_interval.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| num_threads | `--num-threads` | smoke=16, baseline=16, extended=16 | 16 | From Parameter list; see Execution Description With Parameters. |
| cpu_load_percent | `--cpu-load-percent` | smoke=50, baseline=50, extended=50 | 50 | From Parameter list; see Execution Description With Parameters. |
| gpu_load_percent | `--gpu-load-percent` | smoke=50, baseline=50, extended=50 | 50 | From Parameter list; see Execution Description With Parameters. |
| temp_threshold | `--temp-threshold` | smoke=75, baseline=75, extended=75 | 75 | From Parameter list; see Execution Description With Parameters. |
| power_threshold | `--power-threshold` | smoke=700, baseline=700, extended=700 | 700 | From Parameter list; see Execution Description With Parameters. |
| throttle_check | `--throttle-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| ecc_check | `--ecc-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| poll_interval | `--poll-interval` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| duration | `--duration` | smoke=5, baseline=200, extended=580 | 200 | From Parameter list; see Execution Description With Parameters. |
| output_format | `--output-format` | smoke=csv, baseline=csv, extended=csv | csv | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Run stress-ng --cpu --cpu-load, bin/gpu_stress (cuBLAS GEMM), and nvidia-smi --query-gpu
```

## Raw Output Format

CSV with one time-series row per elapsed second

sample_index,status,second,gpu_util_pct,gpu_temp_c,gpu_power_w,ecc_uncorrected,peak_gpu_junction_temp_c,sustained_gpu_power_w,thermal_throttle_event_count,gpu_system_ecc_error_count,error_message
0,ok,0,30,55,120,0,55,120,0,0,

## Metrics

- **#1: Peak GPU temperature** — stored as `peak_gpu_junction_temp_c`.
- **#2: Sustained GPU power** — stored as `sustained_gpu_power_w`.
- **#3: GPU ECC error count** — stored as `gpu_system_ecc_error_count`.
- **#4: Thermal throttle events** — stored as `thermal_throttle_event_count`.

## Framework

Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load.

## Installation and Execution Summary

Run bin/gpu_stress for yaml duration at gpu_load_percent (FP16 cuBLAS GEMM, busy for that percent of each 100 ms window) together with stress-ng --cpu --cpu-load --timeout, polling nvidia-smi temperature, power, utilization, thermal slowdown, and uncorrectable ECC each poll_interval, to measure combined CPU and GPU stress. DCGM and sensors are not run

## Platform Portability

- **AMD (primary):** ```bash
Run stress-ng --cpu --cpu-load, bin/gpu_stress (cuBLAS GEMM), and nvidia-smi --query-gpu
```
- **NVIDIA:** Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

CSV with one time-series row per elapsed second

sample_index,status,second,gpu_util_pct,gpu_temp_c,gpu_power_w,ecc_uncorrected,peak_gpu_junction_temp_c,sustained_gpu_power_w,thermal_throttle_event_count,gpu_system_ecc_error_count,error_message
0,ok,0,30,55,120,0,55,120,0,0,

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load.
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load.

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
