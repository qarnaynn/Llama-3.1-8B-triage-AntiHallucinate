# Llama-3.1-8B-triage-AntiHallucinate: Customer-Message Triage

A fine-tuned Llama-3.1-8B-Instruct that reads a customer message for an online shop in Malaysia and returns a JSON triage record. Fine-tuned with LoRA in Unsloth Studio.

**Model:** [qarnaynn/Llama-3.1-8B-triage-AntiHallucinate on Hugging Face](https://huggingface.co/qarnaynn/Llama-3.1-8B-triage-AntiHallucinate)

## Contents
- [Overview](#overview)
- [Results](#results)
- [Quick start](#quick-start)
- [Dataset](#dataset)
- [Training](#training)
- [Evaluation](#evaluation)
- [Repository structure](#repository-structure)
- [Limitations](#limitations)
- [License](#license)

## Overview
Shops get many customer messages, and a person has to decide what each one is about, how urgent it is, and whether someone must step in. This model does that first sorting step. The fine-tune teaches the base model to answer with only a valid JSON record and to follow our labelling rules, which the untuned model does not do well (it scored 31%, and only 9% of its answers were clean JSON).

**Input**
```
My parcel never arrived and I was charged twice.
```
**Output**
```json
{"category": "delivery", "urgency": "medium", "needs_human": false}
```

| Field | Values |
| :-- | :-- |
| `category` | delivery, refund_return, product_fault, payment_billing, account_access, general_enquiry |
| `urgency` | low, medium, high |
| `needs_human` | true, false |

## Results
Measured on the lecturer's 100 hidden test messages (24 standard, 76 hard), which we never trained on. A message counts only if all three fields are right.

| Model | All three correct | Category | Urgency | needs_human recall | JSON only |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Llama 3.1 8B (untuned, class baseline) | 31% | 84% | 40% | 95% | 9% |
| **This model** | **84%** | 95% | 95% | 96% | 99% |

Details in [`results/leaderboard.md`](results/leaderboard.md). One message is one point, so differences of a few points can be luck. The model scored 80% on the hard messages. The class ranked us second.

## Quick start
Use the system prompt in [`data/system_prompt.txt`](data/system_prompt.txt) and temperature 0. The model is a LoRA adapter, so it needs the base model `unsloth/Llama-3.1-8B-Instruct`.

```sh
pip install -U huggingface_hub
hf download qarnaynn/Llama-3.1-8B-triage-AntiHallucinate --local-dir model
```

### Python with Unsloth
```py
from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained("model", max_seq_length=1024, load_in_4bit=True)
FastLanguageModel.for_inference(model)

messages = [
    {"role": "system", "content": open("data/system_prompt.txt", encoding="utf-8").read()},
    {"role": "user", "content": "My parcel never arrived and I was charged twice."},
]
inputs = tokenizer.apply_chat_template(messages, add_generation_prompt=True, return_tensors="pt").to(model.device)
output = model.generate(input_ids=inputs, max_new_tokens=64, do_sample=False)
print(tokenizer.decode(output[0][inputs.shape[1]:], skip_special_tokens=True))
```

### Unsloth Studio
Add the model in Studio, load it in the **Chat** tab, paste the system prompt, set temperature to 0 and send a message.

**Requirements:** a NVIDIA GPU for the Unsloth option.

## Dataset
| Split | File | Examples | Source | Languages |
| :-- | :-- | :-- | :-- | :-- |
| Train (published model) | `data/train_chat_v13.jsonl` | 1,932 | Synthetic, AI-assisted, plus class data | English, Bahasa Malaysia, Manglish |
| Train (later version) | `data/train_chat_v14.jsonl` | 1,981 | Same, plus 49 long messages | Same |
| Evaluation holdout | `data/eval_holdout_v2*.jsonl` | 50 | Written by us, not in training | About half Bahasa Malaysia / Manglish |
| Test | class hidden set (not published) | 100 | Lecturer | Mixed |

- **Labelling rules:** [`data/LABELING_GUIDE.md`](data/LABELING_GUIDE.md). Main rule: `needs_human` is true if urgency is high, or the customer contacted twice or more, or a refund is more than 14 days overdue.
- **Cleaning:** duplicates removed, and any row too close to the test or holdout messages removed.
- More in [`data/README.md`](data/README.md).

## Training
**Method:** LoRA (16-bit) on Llama-3.1-8B-Instruct, in Unsloth Studio.

| Option considered | For | Against | Chosen? |
| :-- | :-- | :-- | :-- |
| Full fine-tuning | Most flexible | Needs far more GPU memory and time; easy to overfit on about 2,000 rows | No |
| LoRA | Small adapter, fast, works with a few thousand examples | Slightly less flexible | **Yes** |
| QLoRA (4-bit) | Even less memory | Lower precision; we had enough memory for 16-bit | No |

**Why:** our task is narrow (a fixed JSON answer), we had about 2,000 examples, and we had limited GPU time. LoRA gave a small adapter that is easy to share. The adapter is trained only on the assistant's answer ("assistant completions only").

**Base model:** Llama-3.1-8B-Instruct, because it already follows instructions and is small enough to train and serve cheaply.

Full settings are in [`config/training_settings.md`](config/training_settings.md): 3 epochs, learning rate 0.0001, rank/alpha 32/32, batch 2 with gradient accumulation 4, context length 1024, seed 3407.

Loss graphs from Studio can be added in [`assets/`](assets/).

## Evaluation
We used the course's `auto_eval.py` (in `scripts/`) to score models on our own 50-message holdout before choosing which one to load for the leaderboard:

```sh
python scripts/auto_eval.py --test data/eval_holdout_v2_raw.jsonl --openai-url <server>/v1 --model <model name> --api-key <key> --workers 4 --out results
```

Never put a real key or server address in this repository. We scored higher on our own tests (about 93%) than on the hidden class set (84%), because the hidden messages were longer and more varied.

## Repository structure
```
.
├── README.md
├── LICENSE
├── requirements.txt
├── assets/       graphs and images
├── config/       training settings
├── data/         datasets, system prompt, labelling guide
├── scripts/      evaluation and dataset-check scripts (from the course)
├── serving/      system prompt and serving notes
└── results/      leaderboard results
```

## Limitations
- Weaker on long, unusual or mixed-language messages. The hard set was our weakest area (80%).
- Trained and tested on English, Bahasa Malaysia and Manglish only.
- It can still get urgency or needs_human wrong. Not for decisions without human review.

## License
- Code: MIT (see `LICENSE`). The scripts in `scripts/` come from the course material.
- Model: built on Llama 3.1 and subject to the [Llama 3.1 Community License](https://github.com/meta-llama/llama-models/blob/main/models/llama3_1/LICENSE). **Built with Llama.**
- Data: our rows are CC BY 4.0. Class data is from this course.

## Acknowledgements
Unsloth · Meta (Llama 3.1) · our lecturer and the course material

## Authors
Zulqarnain (Hugging Face: qarnaynn) and Haziq Zulkifli (GitHub: Haziqzulkifli5115), team Anti-Hallucinate
