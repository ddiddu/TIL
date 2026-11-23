# A model can't read the probability of the tokens it generated (logprobs)

A model cannot read the probability of the tokens it used when generating an answer.

This is because of how a model writes text. It goes one token at a time:

1. The model reads the text so far and computes the probability of every candidate next token, for example "yes" 0.51, "no" 0.49.
2. The server (sampler) looks at those probabilities and picks one token. At temperature 0 it picks the highest, "yes".
3. Only the chosen token "yes" moves on to the next step. The number 0.51 is thrown away there.
4. The model computes the next token from the text up to "...yes" only.

**logprob** = the log of the probability that one token was picked.

- One value per token. For example, if "yes" was picked with probability 0.51, its logprob is ln(0.51) ≈ −0.67.
- Probability 1 gives 0, and the lower the probability, the more negative the value. So it is always 0 or below.

**The product is the probability of the whole answer.**

- The probability of the whole answer ("yes, because ...") is the product of each token's conditional probability.
- Taking the log turns the product into a sum, so adding all the logprobs gives the log probability of the whole answer.
- Why log: multiplying hundreds of probabilities gets so small (0.0000…) that a computer can't represent it accurately. Floating-point numbers that are too small simply become 0: float32 loses precision below about 10^−38 and becomes 0 below about 10^−45. Addition stays stable.

**When we use it (important):** a structured JSON answer (e.g. per-question yes/no) is thousands of tokens. The total sum means nothing; look only at the tokens that decide the answer, e.g. the probability at the position where "yes" appeared in `"Q1": "yes"`.

**The largest logprob is 0.**

| probability | logprob (ln) |
|---|---|
| 1 | 0 |
| 0.9 | −0.11 |
| 0.5 | −0.69 |
| 0.1 | −2.3 |
| 0.01 | −4.6 |
| close to 0 | close to −∞ |

So the range of a logprob is above −∞ and at most 0. The closer to 0, the more confident the model was.

**Related:** [Self-consistency tests](self-consistency-test.md): use logprobs to rerun only answers below a cutoff.

## 모델은 자기가 생성한 토큰의 확률을 읽을 수 없다 (logprobs)

모델은 답을 생성할 때 사용한 자기 토큰의 확률을 읽을 수 없다.

모델이 글을 만드는 방식 때문이다. 한 토큰씩 이렇게 진행된다.

1. 모델이 지금까지의 글을 읽고, 다음에 올 토큰 후보 전체의 확률을 계산한다. 예를 들면 "yes" 0.51, "no" 0.49.
2. 서버(sampler)가 그 확률을 보고 토큰 하나를 선택한다. temp가 0이면 가장 높은 "yes"를 선택한다.
3. 다음 단계로 넘어가는 건 고른 토큰 "yes" 하나뿐이다. 0.51이라는 숫자는 거기서 버려진다.
4. 모델은 "...yes"까지의 글만 보고 다음 토큰을 계산한다.

**logprob** = token 하나가 뽑힐 확률에 log를 씌운 값

- token마다 하나씩 나온다. 예를 들어 "yes"를 확률 0.51로 골랐다면 logprob은 ln(0.51) ≈ −0.67이다.
- 확률이 1이면 0이고, 확률이 낮을수록 더 큰 음수가 된다. 그래서 값은 항상 0 이하다.

**곱은 답 전체의 확률이다.**

- 답 전체("yes, because ...")가 나올 확률은 각 토큰의 조건부 확률을 모두 곱한 값이다.
- log를 쓰면 곱이 합이 된다. 그래서 logprob을 모두 더하면 전체 답의 로그 확률이 된다.
- log를 쓰는 이유는, 확률을 수백 개 곱하면 0.0000…처럼 너무 작아져 컴퓨터가 정확히 다루지 못하기 때문이다. 컴퓨터의 실수는 너무 작으면 그냥 0이 된다. float32는 약 10^−38 아래부터 정밀도를 잃고, 약 10^−45보다 작으면 0으로 처리된다. 더하기는 안정적이다.

**우리가 쓸 때 (중요):** 구조화된 JSON 응답(예: 질문별 yes/no 판정)은 토큰이 수천 개다. 전체 합은 의미가 없고, 답을 정하는 토큰만 봐야 한다. 예를 들어 `"Q1": "yes"`에서 "yes"가 나온 자리의 확률이다.

**logprob의 가장 큰 값은 0이다.** (값은 위 표와 같다)

그래서 logprob의 범위는 −∞ 초과, 0 이하다. 0에 가까울수록 모델이 확신한 것이다.

**관련 글:** [Self-consistency test](self-consistency-test.md): logprob이 기준값보다 낮은 답만 다시 돌리는 방법.

## Sources
- OpenAI Cookbook, Using logprobs: https://cookbook.openai.com/examples/using_logprobs
- Single-precision floating-point format: https://en.wikipedia.org/wiki/Single-precision_floating-point_format
