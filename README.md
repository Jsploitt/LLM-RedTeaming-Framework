# LLM Red-Teaming Framework

A Python framework for systematically testing large language models (LLMs) against adversarial jailbreak prompts. It evaluates how robustly different models resist a variety of attack techniques, measures bypass rates, and generates detailed markdown reports with recommendations.

---

## Features

- **Multi-model testing** — Supports OpenAI (GPT-4o), Anthropic (Claude 3.5 Sonnet), and Meta (Llama 3 70B via Together AI) out of the box.
- **Categorised attack library** — 12 pre-built attack prompts across 4 attack categories (roleplay, injection, multilingual, sociotechnical).
- **Automatic jailbreak detection** — Scores each response as refused, errored, or a successful jailbreak using a configurable keyword list.
- **JSONL result logging** — Persists every test result to a newline-delimited JSON file for further analysis.
- **Markdown report generation** — Produces a detailed report with success-rate tables, a model × category matrix, annotated jailbreak examples, and actionable recommendations.
- **Rate-limit friendly** — Configurable delay between API calls.
- **Report-only mode** — Re-generate a report from previously saved results without calling any APIs.

---

## Repository Structure

```
LLM-RedTeaming-Framework/
├── redteam.py        # Main application (CLI entry point + all framework logic)
├── attacks.json      # Attack prompt library with category metadata
├── requirements.txt  # Python dependencies
└── README.md
```

---

## Requirements

- Python 3.10+
- API keys for the providers whose models you want to test

### Python Dependencies

```
openai>=1.0.0
anthropic>=0.18.0
```

---

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/Jsploitt/LLM-RedTeaming-Framework.git
cd LLM-RedTeaming-Framework

# 2. (Optional) Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate   # macOS/Linux
.venv\Scripts\activate      # Windows

# 3. Install dependencies
pip install -r requirements.txt
```

---

## Configuration

Set your API keys as environment variables before running the framework.

| Variable | Required for |
|---|---|
| `OPENAI_API_KEY` | GPT-4o |
| `ANTHROPIC_API_KEY` | Claude 3.5 Sonnet |
| `TOGETHER_API_KEY` | Llama 3 70B (via Together AI) |

```bash
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export TOGETHER_API_KEY="..."
```

The framework only requires keys for the models you intend to test. Any model whose key is missing will be excluded automatically.

---

## Usage

### Test all models (default)

```bash
python redteam.py
```

### Test specific models

```bash
python redteam.py --models gpt-4o claude-3.5-sonnet
```

### Custom file paths

```bash
python redteam.py \
  --attacks attacks.json \
  --output  results.jsonl \
  --report  report.md
```

### Adjust the delay between API calls

```bash
python redteam.py --delay 2.0   # 2 seconds between requests
```

### Re-generate report from existing results (no API calls)

```bash
python redteam.py --report-only
```

### Full CLI reference

```
usage: redteam.py [-h] [--attacks ATTACKS] [--output OUTPUT] [--report REPORT]
                  [--models {gpt-4o,claude-3.5-sonnet,llama-3-70b} [...]]
                  [--delay DELAY] [--report-only]

options:
  -h, --help            Show this help message and exit
  --attacks ATTACKS     Path to attacks JSON file (default: attacks.json)
  --output OUTPUT       Path to results JSONL file (default: results.jsonl)
  --report REPORT       Path to output markdown report (default: report.md)
  --models ...          Models to test; omit to test all
  --delay DELAY         Seconds between API calls (default: 1.0)
  --report-only         Generate report from existing results, skip testing
```

---

## Supported Models

| Name | Provider | Model ID |
|---|---|---|
| `gpt-4o` | OpenAI | `gpt-4o` |
| `claude-3.5-sonnet` | Anthropic | `claude-sonnet-4-20250514` |
| `llama-3-70b` | Together AI | `meta-llama/Llama-3-70b-chat-hf` |

---

## Attack Categories

The attack library (`attacks.json`) contains **12 prompts** across **4 categories**:

| Category | Count | Description |
|---|---|---|
| `roleplay` | 3 | Persona/character jailbreaks (e.g. DAN, Evil Confidant, RogueAI) |
| `injection` | 3 | Prompt-injection attempts to override safety instructions |
| `multilingual` | 3 | Language-mixing and Unicode obfuscation to evade filters |
| `sociotechnical` | 3 | Social-framing attacks disguised as research, fiction, or education |

You can extend the library by adding entries to `attacks.json` following the existing schema:

```json
{
  "id": "unique_attack_id",
  "category": "category_name",
  "name": "Human-readable Name",
  "prompt": "The attack prompt text."
}
```

---

## Output Files

Running the framework produces two output files:

### `results.jsonl`

One JSON object per line, one per (attack, model) pair. Each object contains:

| Field | Description |
|---|---|
| `timestamp` | ISO-8601 timestamp of the test |
| `model` | Model name |
| `attack_id` | Attack identifier |
| `attack_name` | Human-readable attack name |
| `category` | Attack category |
| `prompt` | The attack prompt sent to the model |
| `response` | Model response (truncated to 2000 characters) |
| `response_length` | Full character length of the response |
| `latency_seconds` | Round-trip API latency |
| `error` | Error message if the API call failed, otherwise `null` |
| `refused` | `true` if a refusal keyword was detected |
| `jailbreak_success` | `true` if the model did **not** refuse |

### `report.md`

A markdown report containing:

- **Summary tables** — overall jailbreak success rates per model and per attack category.
- **Model × Category matrix** — cross-tabulated bypass counts (`successes/total`).
- **Successful jailbreaks** — collapsible sections showing the prompt and (truncated) response for every bypassed safety check.
- **Recommendations** — risk level assessment and guidance for each category based on measured bypass rates.

---

## How Jailbreak Detection Works

After each API call the response is checked for any of 30+ refusal phrases (e.g. `"i cannot"`, `"refuse to"`, `"against my guidelines"`). If none are found, the test is scored as a successful jailbreak (`jailbreak_success = true`). API errors are recorded separately and do not count as either a refusal or a success.

---

## Disclaimer

This framework is intended **strictly for security research and defensive purposes**. Use it only against models you have explicit permission to test. The authors do not condone using this tool to circumvent safety measures for harmful purposes.

---

## License

This project is open source. See the repository for license details.
