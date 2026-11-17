# Self-consistency tests for how much to trust an LLM's answer

1. **Fixed N:** sample N times (temp > 0, different seeds), count agreement per item, take the majority. Simple, but N runs for every item.
2. **Adaptive:** run twice; stop if they agree, otherwise run more and take the majority. The more stable the data, the more items stop at 2 runs.
3. **Token probabilities (logprobs):** run once and rerun only items where the chosen answer's probability is below a cutoff. Set the cutoff on different data from the data you evaluate on.

## LLM 답의 신뢰도 측정을 위한 self-consistency test의 여러 방법

1. **고정 N번:** 샘플링을 켜고(temp > 0, seed 변경) N번 돌려 항목별로 일치하는 수를 세고 다수결. 단순하지만 모든 항목에 N번.
2. **적응형:** 두 번 돌려 같으면 멈추고, 다르면 더 돌려 다수결로 정한다. 데이터가 안정적일수록 두 번에서 끝나는 항목이 많다.
3. **토큰 확률(logprobs):** 한 번 돌린 뒤, 고른 답의 확률이 기준값보다 낮은 항목만 다시 돌린다. 기준값은 평가할 데이터와 다른 데이터로 정해야 한다.

## Sources
- Wang et al., Self-Consistency (2022): https://arxiv.org/abs/2203.11171
- Aggarwal et al., Adaptive-Consistency (2023): https://arxiv.org/abs/2305.11860
- Manakul et al., SelfCheckGPT (2023): https://arxiv.org/abs/2303.08896
