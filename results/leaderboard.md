# Results 

A message counts as correct only if category, urgency and needs_human are all right. The test set has 24 standard and 76 hard messages.

| Model | All three correct | Category | Urgency | needs_human recall | JSON only | Hard |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Llama 3.1 8B, untuned (class baseline) | 31% | 84% | 40% | 95% | 9% | 30% |
| **This model** (run 1795752790) | **84%** | 95% | 95% | 96% | 99% | 80% |

Source: the class baseline scorecard and the class leaderboard. One message is one point, so a difference of a few points can be luck. The pass mark was 75%.
