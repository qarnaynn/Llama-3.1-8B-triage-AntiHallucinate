# Data

| File | What it is |
| :-- | :-- |
| `system_prompt.txt` | The system prompt used for every row, in training and in testing |
| `LABELING_GUIDE.md` | The labelling rules (class material) |
| `class_train_raw.jsonl` | The 157 labelled training messages given in class (class data from this course) |
| `train_chat_v13.jsonl` | **Training set of the published model.** 1,932 chat-format rows |
| `train_chat_v14.jsonl` | A later version with 1,981 rows (adds 49 long, realistic messages). Not used for the published model |
| `eval_holdout_v2.jsonl` | 50-message holdout in chat format, for the Unsloth eval dataset |
| `eval_holdout_v2_raw.jsonl` | The same 50 messages with plain columns, for `scripts/auto_eval.py` |

The class test messages were never used for training and are not published here.

**Format of the chat files** (one JSON object per line): `messages` holds a system message, the customer message, and the assistant's JSON answer.

**Label distribution (published training set, 1,932 rows)**

| urgency low | medium | high | needs_human true | false |
| :-- | :-- | :-- | :-- | :-- |
| 426 | 993 | 513 | 845 | 1,087 |

Category counts: delivery 404, refund_return 389, product_fault 354, payment_billing 311, general_enquiry 241, account_access 233.

**How it was made:** synthetic examples written with AI assistance on top of the class training data, in English, Bahasa Malaysia and Manglish, grown over several versions. Duplicates were removed, and any row too close to the class test messages or to our holdout files was removed. Labels follow `LABELING_GUIDE.md`.

**Data licence:** our own rows are CC BY 4.0. The class files (`class_train_raw.jsonl`, `LABELING_GUIDE.md`) are class material from this course.
