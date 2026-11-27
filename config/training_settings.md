# Training settings (Unsloth Studio)

Published model: run `unsloth_Llama-3.1-8B-Instruct__project-anti_hallucinate-v13_1795752790` (leaderboard model number 1795752790).

| Setting | Value |
| :-- | :-- |
| Base model | unsloth/Llama-3.1-8B-Instruct |
| Method | LoRA, 16-bit |
| Dataset | `data/train_chat_v13.jsonl` (1,932 examples) |
| Eval dataset | a held-out file, checked every 50 steps |
| Epochs | 3 |
| Learning rate | 0.0001 |
| Rank / alpha | 32 / 32 |
| Dropout | 0 |
| Target modules | all 7 (q, k, v, o, gate, up, down) |
| Optimizer | AdamW 8-bit |
| Batch size | 2, gradient accumulation 4 |
| Warmup steps | 30 |
| Max sequence length | 1024 |
| Seed | 3407 |
| Gradient checkpointing | Unsloth |
| Packing | off |
| Train on | Assistant completions only |
| Early stopping | off |
| Save steps | 1000 (a low value filled the disk with checkpoints) |
