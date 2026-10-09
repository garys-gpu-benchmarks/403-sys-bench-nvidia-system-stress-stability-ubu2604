# CPU + System Stress (Baseline Health) Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/403-sys-bench-nvidia-system-stress-stability-ubu2604/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/403-sys-bench-nvidia-system-stress-stability-ubu2604/actions/workflows/ci.yml)

Target: Ubuntu 26.04 · NVIDIA · see Hardware Requirements. This is a host benchmark, not a laptop `pip install` project.

## Quick Start

```bash
git clone https://github.com/garys-gpu-benchmarks/403-sys-bench-nvidia-system-stress-stability-ubu2604.git
cd 403-sys-bench-nvidia-system-stress-stability-ubu2604
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --profile smoke --validate
```
Results are written to `results/benchmark.db` and `results/summary.json`.

This workload is executed on the validation host after the repository is copied there. `setup.sh` and `run_benchmark.sh` do not open an outbound SSH session.

Prerequisites: Ubuntu 26.04; NVIDIA; Python 3.14.4; root or sudo for `setup.sh`. Framework: Bash, SQLite, Python, PyYAML, CUDA Runtime, stress-ng, nvidia-smi, bin/gpu_stress. This is a host benchmark, not a laptop `pip install` project.

```mermaid
flowchart LR
  setup.sh --> run_benchmark.sh --> parse_results.py --> results/benchmark.db
```

## 1. Overview

Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load. gpu_load_percent: share of each 100 ms window the GPU runs FP16 GEMMs (default 50). Sweep dimensions: num_threads, cpu_load_percent, gpu_load_percent, temp_threshold, power_threshold, throttle_check, ecc_check, poll_interval.

## 2. What It Validates

- Validates that stress-ng and the cuBLAS GEMM load (bin/gpu_stress) both run for the yaml duration and that peak GPU temperature, sustained (mean) GPU power, and uncorrectable GPU ECC errors are parsed from nvidia-smi samples taken every poll_interval seconds. The run fails if bin/gpu_stress does not start or report GEMMs, or if mean utilization.gpu is below half of gpu_load_percent (minimum 10%). Peak temperature above temp_threshold, mean power above power_threshold, or a hardware thermal slowdown event with throttle_check true marks every sample error
- #1: Peak GPU temperature (peak_gpu_junction_temp_c); is present and physically sensible.
- #2: Sustained GPU power (sustained_gpu_power_w); is present and physically sensible.
- #3: GPU ECC error count (gpu_system_ecc_error_count); is present and physically sensible.
- #4: Thermal throttle events (thermal_throttle_event_count) is present and physically sensible.

## 3. Metrics Captured

- **#1: Peak GPU temperature** — stored as `peak_gpu_junction_temp_c`.
- **#2: Sustained GPU power** — stored as `sustained_gpu_power_w`.
- **#3: GPU ECC error count** — stored as `gpu_system_ecc_error_count`.
- **#4: Thermal throttle events** — stored as `thermal_throttle_event_count`.

## 4. Hardware Requirements

### Supported environment

- OS: Ubuntu 26.04
- GPU vendor: NVIDIA
- Framework family: Bash, SQLite, Python, PyYAML, CUDA Runtime, stress-ng, nvidia-smi, bin/gpu_stress
- Python: Python 3.14.4

### Reference validation environment

The tables below describe the machine used to generate the reference results. They are not a requirement that every user buy that exact cloud instance.

### System

Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load.

### GPU

Ubuntu 26.04 / NVIDIA / Bash, SQLite, Python, PyYAML, CUDA Runtime, stress-ng, nvidia-smi, bin/gpu_stress

## 5. Software Requirements

| Component | Version |
|---|---|
| OS | Ubuntu 26.04 |
| Kernel | kernel 7.0.0 |
| Python | Python 3.14.4 |
| ROCm | CUDA 13.3 |
| rocBLAS | cuBLAS (bundled with CUDA 13.3) |

Runs stress-ng CPU workers and a cuBLAS GEMM GPU load (bin/gpu_stress) together while polling nvidia-smi every poll_interval seconds. num_threads: stress-ng --cpu workers (default 16). cpu_load_percent: stress-ng --cpu-load.

## 6. Installation

```bash
Run stress-ng --cpu --cpu-load, bin/gpu_stress (cuBLAS GEMM), and nvidia-smi --query-gpu
```

## 7. Running the Benchmark

```bash
Run stress-ng --cpu --cpu-load, bin/gpu_stress (cuBLAS GEMM), and nvidia-smi --query-gpu
```

**Validating results separately:**

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional
python3 -m venv .venv
source ".venv/bin/activate"
".venv/bin/python" scripts/validate_results.py
```

## 8. Output

### `results/benchmark.db` (SQLite)

CSV with one time-series row per elapsed second

sample_index,status,second,gpu_util_pct,gpu_temp_c,gpu_power_w,ecc_uncorrected,peak_gpu_junction_temp_c,sustained_gpu_power_w,thermal_throttle_event_count,gpu_system_ecc_error_count,error_message
0,ok,0,30,55,120,0,55,120,0,0,

```bash
Run stress-ng --cpu --cpu-load, bin/gpu_stress (cuBLAS GEMM), and nvidia-smi --query-gpu
```

### `results/summary.json`

Consolidated metrics from the most recent run — suitable for CI artifact upload or dashboard ingestion.

### `results/raw/<timestamp>.txt`

CSV with one time-series row per elapsed second

sample_index,status,second,gpu_util_pct,gpu_temp_c,gpu_power_w,ecc_uncorrected,peak_gpu_junction_temp_c,sustained_gpu_power_w,thermal_throttle_event_count,gpu_system_ecc_error_count,error_message
0,ok,0,30,55,120,0,55,120,0,0,

## 9. Baselines / Thresholds

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

## 10. Troubleshooting

**`setup.sh` missing collector**
Create cannot finish without `scripts/collect_workload.py`.

**`self_check` overlay rewritten**
Do not overwrite files listed in `results/overlay_lock.json`.

**Remote SSH drop during setup**
Reconnect and resume `bash setup.sh --assume-yes`. Do not wipe `.venv` or `.cache`.

## 11. NVIDIA H100 Coding Differences

Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Repository layout

```text
.
├── setup.sh
├── run_benchmark.sh
├── benchmark_specification.json
├── .github/workflows/      # thin CI callers (see Continuous Integration)
├── config/
├── scripts/
├── src/
├── tests/
├── docs/
├── results/
└── LICENSE
```

## Continuous Integration

| Workflow | Runs on | When | What it does |
|---|---|---|---|
| [CI](.github/workflows/ci.yml) | GitHub-hosted runner | every pull request, and every push to `main` | shellcheck, ruff, `bash -n`, `compileall`, `run_benchmark.sh --help`, specification schema, the results validator on a seeded fixture, required files, and actionlint. No GPU and no benchmark run. |
| [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) | self-hosted runner labeled `gpu`, `nvidia`, `ubu2604` | only when started by hand: **Actions → GPU Smoke Benchmark → Run workflow** (choose `smoke`, `baseline` or `extended`) | Verifies the pre-provisioned GPU stack, records `results/environment.json` (driver, runtime, kernel, GPU), runs the profile with `--validate`, shows headline metrics on the run page, and uploads the results. |

Both files are short callers. The steps themselves live once, for every workload in the suite, in [`garys-gpu-benchmarks/shared-workflows`](https://github.com/garys-gpu-benchmarks/shared-workflows), pinned at `@v1`. The GPU workflow is never triggered by pull requests, so code from a fork cannot run on the GPU host.

### Running it as part of the NVIDIA Ubuntu 26.04 bundle

This repository is one of the 32 workloads in [`bundle-nvidia-ubuntu-2604`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604), which holds them as git submodules. To put the whole bundle on a GPU host and run this workload from it:

```bash
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604 /opt/benchmarks
cd /opt/benchmarks/403-sys-bench-nvidia-system-stress-stability-ubu2604
bash run_benchmark.sh --profile smoke --validate
```
