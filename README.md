# QH-Bench: Chinese Adolescent LLM Safety Benchmark

This repository is the anonymous review artifact for **Raising the Bar for Chinese Adolescent LLM Safety: A Culturally-Grounded, Fine-Grained Benchmark**.

QH-Bench evaluates whether language models respond safely and helpfully to Chinese adolescent safety scenarios. It contains complementary single-turn and multi-turn tracks. The single-turn track measures performance across fine-grained risk domains. The multi-turn track measures whether a model uses dialogue history to maintain or update its safety judgment.

## Benchmark at a glance

| Component | Scale |
| --- | ---: |
| Single-turn test items | 715 |
| Risk domains | 10 |
| Subdomains | 50 |
| Fine-grained risk scenarios | 143 |
| Multi-turn trajectories | 100 |
| User turns in the multi-turn track | 400 |
| Cross-turn mechanisms | 10 |
| Evaluated open-weight models | 13 |

All benchmark records are synthetic and were reviewed before inclusion. They are not transcripts of real adolescent conversations.

## Repository contents

```text
datasets/
  青少年内容安全单轮场景.json       # 715 single-turn items
  青少年内容安全多轮场景.json       # 100 four-turn trajectories
  青少年内容安全分类表.xlsx          # three-level risk taxonomy
scripts/
  eval_single_turn.py              # single-turn model inference
  eval_multi_turn.py               # multi-turn model inference
  score_single_turn.py             # single-turn automatic judging
  score_multi_turn.py              # multi-turn automatic judging
  analyze_results.py               # tables, statistics, and figures
  run_risk_refresh.py              # optional experiment orchestration
  setup_vllm_env.sh                # reference vLLM environment setup
  requirements.txt
results/
  model_outputs/                   # complete outputs for 13 models
  scores/                          # automatic scores and human-audit records
  analysis/                        # generated diagnostic figures
```

The release excludes manuscript source, author information, development logs, local caches, and temporary files.

## Data format

The single-turn JSON stores an item ID, three taxonomy labels, a scenario background, and one user message. Evaluation concatenates the background and user message into one model input.

The multi-turn JSON stores a trajectory ID, a mechanism ID and name, a scenario background, and four predefined user messages. The first model input concatenates the background and first user message. Later turns retain the model's actual preceding responses and append the next predefined user message. Dialogue history is reset between trajectories.

The taxonomy and mechanism labels are used for dataset organization and analysis. They are not passed to the evaluated model or the automatic judge.

## Installation

The inference scripts expect Python, an OpenAI-compatible model endpoint, and a vLLM-compatible runtime for local models.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r scripts/requirements.txt
```

The provided `setup_vllm_env.sh` records the environment used for the reported experiments. Hardware-specific settings may need adjustment.

## Run model inference

List the supported model presets:

```bash
python scripts/eval_single_turn.py --list-models
```

Run one single-turn evaluation:

```bash
python scripts/eval_single_turn.py \
  --model-presets qwen2.5-7b-instruct \
  --api-base http://127.0.0.1:8000/v1
```

Run one multi-turn evaluation:

```bash
python scripts/eval_multi_turn.py \
  --model-presets qwen2.5-7b-instruct \
  --api-base http://127.0.0.1:8000/v1
```

The reported experiments used temperature `0`, top-p `0.9`, a maximum of `512` new tokens, and repetition penalty `1.05`.

## Score model outputs

Configure an OpenAI-compatible judge endpoint without storing credentials in the repository:

```bash
export JUDGE_API_KEY="..."
export JUDGE_API_BASE="https://your-judge-endpoint.example/v1"
export JUDGE_MODEL="claude-opus-4-8"

python scripts/score_single_turn.py
python scripts/score_multi_turn.py
```

The single-turn judge receives the scenario background, user message, and model response. The multi-turn judge receives the complete four-turn interaction and assigns one score to the full trajectory. Both tracks use the same five-level safety-helpfulness scale from `-2` to `+2`, with track-specific judging instructions stored in the dataset files and scoring scripts.

The original experiments used a third-party OpenAI-compatible endpoint. The request model identifier alone may not uniquely identify the server-side model used by that provider.

## Analyze results

```bash
python scripts/analyze_results.py
```

The analysis reads the released score files and regenerates aggregate tables and diagnostic figures. Raw model outputs are included so reviewers can inspect individual cases and re-run judging.

## Basic integrity checks

```bash
python scripts/eval_single_turn.py --list-models
python scripts/eval_multi_turn.py --list-models
python scripts/score_single_turn.py --help
python scripts/score_multi_turn.py --help
python scripts/analyze_results.py --help
```

The expected complete release contains 715 unique single-turn items, 100 unique multi-turn trajectories, 9,295 single-turn model responses, 1,300 multi-turn model-trajectory records, and 5,200 assistant responses across the multi-turn trajectories.

## Intended use and safety notice

QH-Bench is intended for model evaluation, safety research, and diagnostic analysis. It is not designed to estimate the real-world prevalence or severity of adolescent risks. The benchmark contains sensitive and potentially harmful scenarios and model responses. These materials should not be presented to adolescents as advice or used to facilitate harmful behavior.

Benchmark results should be interpreted for the released item set and considered together with the target application, system safeguards, human oversight, and the needs of its users.

## Anonymous review

This review artifact intentionally omits author names, affiliations, acknowledgments, and links that would reveal author identity. Citation and license information will be added after the double-blind review period.
