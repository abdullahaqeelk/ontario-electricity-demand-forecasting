# Status: completed

`notebook.ipynb` ran end to end with saved cell outputs and plots. No metrics were estimated or invented.

Dataset: IESO public annual hourly demand reports, **Ontario Demand (MW)**, 2022 and 2023. Source: http://reports.ieso.ca/public/Demand/

Date range (interval starts): **2022-01-01 00:00:00 through 2023-12-31 23:00:00**, 17,520 hours.

## Final test metrics

| Model | MAE (MW) | RMSE (MW) | MAPE (%) |
|---|---:|---:|---:|
| Baseline (t-24) | 761.04 | 1024.27 | 4.82 |
| LSTM | 218.25 | 286.38 | 1.38 |
| GRU | 196.68 | 256.25 | 1.24 |

GRU had the lowest test RMSE in this run. The recurrent models use the latest 24 hours, allowing them to follow recent changes that yesterday's same hour misses. The relative ranking is specific to this test period; weather and holidays are not model inputs.

## Run details

```text
17,520 hours: 2022-01-01 00:00:00 to 2023-12-31 23:00:00
Duplicate hours averaged: 0; missing hours interpolated: 0
Windows: train=12,590, validation=1,402, test=3,504
Validation starts: 2023-06-10 14:00:00
Test starts: 2023-08-08 00:00:00
Window shape: (12590, 24, 1)
Epochs run: {'LSTM': 20, 'GRU': 20}
```

## Problems and workarounds

- Sandboxed network access initially failed DNS resolution. Authorized downloads and dependency installation ran with network access.
- HTTPS IESO URLs returned HTML instead of CSV data. The original HTTP URLs worked; the downloaded headers and date ranges were checked. A PJM fallback URL was also tried and returned 404 before HTTP IESO succeeded; no fallback data was used.
- The default Python environment lacked TensorFlow and Jupyter libraries. A full new TensorFlow download was slow, so it was cancelled. Created a local `.venv` using the existing Python 3.11 ML environment with system site packages, then installed the two missing notebook libraries locally. Package versions used are pinned in `requirements.txt`.
- The existing TensorFlow build emits CUDA factory registration messages even in CPU mode; CPU training completed successfully. Matplotlib uses a writable temporary cache.
- Jupyter could not start its local TCP kernel inside the sandbox; notebook execution succeeded with the authorized local kernel outside that restriction.
- Explicit TensorFlow thread limits and a one-thread data pipeline keep CPU resource use small. No GPU is required.
- The experiment is a rolling one-hour forecast using past demand only. It does not include weather, holidays or a separate tuning study.
