# A model can't see the probability of the token it just picked (logprobs)

**Why the model can't read its own probabilities**
A model writes one token at a time.
1. It reads the text so far and computes probabilities for every candidate next token, e.g. "yes" 0.51, "no" 0.49.
2. The server (sampler) picks one token. At temperature 0 it picks the highest, "yes".
3. Only the chosen token "yes" moves to the next step. The number 0.51 is not part of the next input.
4. The model computes the next token from "...yes" alone.

So asking the model "how sure were you?" does not read the real probability. To get it, request logprobs from the server.

**logprob**
- The log of the probability that one token was picked. One value per token.
- "yes" picked at 0.51 → logprob = ln(0.51) ≈ −0.67
- Probability 1 → 0; lower probability → more negative. Always ≤ 0.

| probability | logprob (ln) |
|---|---|
| 1 | 0 |
| 0.9 | −0.11 |
| 0.5 | −0.69 |
| 0.1 | −2.3 |
| 0.01 | −4.6 |
| → 0 | → −∞ |

Range: greater than −∞, at most 0. Closer to 0 = more confident.

**Whole answer = product → sum in log space**
- The probability of the whole answer ("yes, because ...") is the product of each step's **conditional** probability: P(yes) × P(, | yes) × P(because | yes,) × …
- Taking logs turns the product into a sum: adding the logprobs gives the log probability of the whole answer.
- Why log: multiplying hundreds of probabilities gets too small for floats. float32 loses precision below ~10^−38 and becomes 0 below ~10^−45. Sums stay stable.

**In practice**
- A structured JSON answer (e.g. per-question yes/no) is thousands of tokens. The total sum means little; look only at the tokens that decide the answer, e.g. the logprob of "yes" in `"Q1": "yes"`.
- Confident ≠ correct. A logprob near 0 means the model was sure, not that it was right. Compare against labels (calibration) before relying on it.
- Whether the server's logprobs are before or after temperature depends on the server and its settings. Check the docs.

**Related:** [Self-consistency tests](self-consistency-test.md): use logprobs to rerun only answers below a cutoff.

## 모델은 자기가 고른 토큰의 확률을 모른다 (logprobs)

**왜 모델은 자기 확률을 읽을 수 없나**
모델은 한 토큰씩 글을 만든다.
1. 지금까지의 글을 읽고, 다음 토큰 후보 전체의 확률을 계산한다. 예: "yes" 0.51, "no" 0.49
2. 서버(sampler)가 토큰 하나를 고른다. temperature가 0이면 가장 높은 "yes"를 고른다.
3. 다음 단계로 넘어가는 건 고른 토큰 "yes" 하나뿐이다. 0.51이라는 숫자는 다음 입력에 들어가지 않는다.
4. 모델은 "...yes"까지의 글만 보고 다음 토큰을 계산한다.

그래서 모델에게 "방금 얼마나 확신했어?"라고 물어도 실제 확률을 읽어서 답하는 게 아니다. 확률이 필요하면 서버에서 logprobs를 받아야 한다.

**logprob**
- 토큰 하나가 뽑힐 확률에 log를 씌운 값. 토큰마다 하나씩 나온다.
- "yes"를 0.51로 골랐다면 logprob = ln(0.51) ≈ −0.67
- 확률이 1이면 0, 낮을수록 더 큰 음수 → 항상 0 이하

범위는 −∞ 초과, 0 이하. 0에 가까울수록 모델이 확신한 것이다. (값은 위 표와 같음)

**답 전체의 확률 = 곱 → log를 쓰면 합**
- 답 전체("yes, because ...")가 나올 확률은 각 단계의 **조건부** 확률을 모두 곱한 값이다. P(yes) × P(, | yes) × P(because | yes,) × …
- log를 씌우면 곱이 합이 된다. logprob을 모두 더하면 답 전체의 로그 확률이다.
- log를 쓰는 이유: 확률을 수백 개 곱하면 너무 작아져서 컴퓨터가 다루지 못한다. float32는 10^−38 아래부터 정밀도를 잃고, 약 10^−45보다 작으면 0이 된다. 더하기는 안정적이다.

**실제로 쓸 때 (중요)**
- 구조화된 JSON 응답(예: 질문별 yes/no 판정)은 토큰이 수천 개다. 전체 합은 의미가 없고, 답을 정하는 토큰만 봐야 한다. 예: `"Q1": "yes"`에서 "yes"가 나온 자리의 logprob.
- 확신 ≠ 정답. 0에 가까운 logprob은 모델이 확신했다는 뜻이지 맞았다는 뜻은 아니다. 쓰기 전에 정답 라벨과 비교해서 확인(calibration)해야 한다.
- 서버가 주는 logprobs가 temperature 적용 전 값인지 후 값인지는 서버와 설정에 따라 다르다. 문서를 확인할 것.

**관련 글:** [Self-consistency test](self-consistency-test.md): logprob이 기준값보다 낮은 답만 다시 돌리는 방법.

## Sources
- OpenAI Cookbook, Using logprobs: https://cookbook.openai.com/examples/using_logprobs
- Guo et al., On Calibration of Modern Neural Networks (2017): https://arxiv.org/abs/1706.04599
- Kadavath et al., Language Models (Mostly) Know What They Know (2022): https://arxiv.org/abs/2207.05221
- Single-precision floating-point format: https://en.wikipedia.org/wiki/Single-precision_floating-point_format
