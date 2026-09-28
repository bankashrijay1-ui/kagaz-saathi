# Kagaz Saathi (कागज़ साथी)

Private, offline explainer for Indian official documents (tax and bank notices, insurance, rent agreements, lab reports). Explains them in Hindi, Hinglish or English, on a Snapdragon-powered HP PC. Built for the Snapdragon AI Lab Build & Present Challenge.

## Why on-device
These papers contain Aadhaar, PAN, phone numbers and account numbers. Under India's DPDP Act, 2023, sending them to a cloud API is a real risk. Here:
1. Personal numbers are masked locally (regex) before the model sees anything, and the masked view is shown to the user.
2. The LLM runs on the Snapdragon NPU via a local server. The app only talks to `127.0.0.1`.
3. The server uses only the Python standard library, so there are no ARM64 wheel problems.

## Architecture
```
Browser UI  ->  server.py (mask PII, stream, time)  ->  local OpenAI-compatible LLM server (GenieX, NPU)
```

## Setup on Windows on Snapdragon
1. Install Python 3.11+ (ARM64 or x64 both work; this project needs no packages).
2. Install GenieX and pull a model, per the GenieX docs (github.com/qualcomm/GenieX):
   `geniex pull ai-hub-models/Qwen3-4B-Instruct-2507`, then start its OpenAI-compatible server. Note the URL it prints.
3. In PowerShell, in this folder:
```
$env:LLM_URL="http://127.0.0.1:<PORT>/v1"   # from GenieX output
$env:LLM_MODEL="<model name GenieX reports>"
$env:BACKEND_LABEL="NPU-GenieX-Qwen3-4B"
python server.py
```
4. Open http://127.0.0.1:5000, click Load sample, then Explain.

## Benchmark (fill with YOUR measured numbers only)
Run the same sample 5 times per backend. Change `BACKEND_LABEL` and the GenieX runtime (NPU bundle vs llama.cpp CPU/GPU) between runs. Rows are logged to `bench.jsonl` and shown in the app.

| Backend | Model | First word (ms) | Tokens/s | Peak RAM (Task Manager) | Battery drop per 10 runs |
|---|---|---|---|---|---|
| NPU | | | | | |
| CPU | | | | | |

Tokens are counted as streamed chunks, which approximates tokens. Offline test: turn Wi-Fi off and repeat one run.

## Limits (be honest in the write-up)
- Text input only; OCR of photos is future work.
- Explanations can be wrong. The app says so and never guesses masked values.
- Regex masking covers Aadhaar, PAN, phone, email and long account numbers, not names or addresses.

## License
MIT. Models keep their own licenses.
