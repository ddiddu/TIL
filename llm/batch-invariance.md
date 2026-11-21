# temp 0 and a fixed seed don't guarantee identical LLM outputs (batch invariance)

Temperature 0 and a fixed seed help, but they don't guarantee identical outputs when batching and scheduling change.

**What it means:** how the GPU splits the computation depends on how many requests it is processing at the same time.

An analogy: you hand a team the job of adding 1,000 numbers.

- If the team is busy, one person adds all 1,000 numbers from start to finish.
- If people are idle, ten people each add 100, and then the partial sums are added together.

Mathematically, the answer should be the same, but a computer rounds at every step, so the last digits can differ. The same request can be split differently depending on whether it arrived alone or batched with seven others, and the last digits of the result change. That is what "not batch-invariant" means. With workers 1 (sending one request at a time), each request runs alone, so it splits the same way every time and produces the same result (as long as no one else is sending requests to the same server).

**Does everyone run with workers 1?** No, most don't.

- Most services just accept the variation. Where speed and throughput matter, like chatbots, concurrent requests are the default and a slightly different answer isn't a problem.
- Where reproducibility is essential, they turn on batch-invariant mode: evaluation, reinforcement learning, audit records. Outputs stay identical while keeping most of the speed of concurrent requests.
  - In vLLM, set `VLLM_BATCH_INVARIANT=1`.
  - It requires an NVIDIA GPU with compute capability 9.0 or higher; the GPUs listed are the H-series (H100, H200) and B-series (B100, B200).
  - It can be slower than the default mode, which the docs describe as an intentional trade-off for reproducibility.
- Many teams design around the variation: run several times and take a majority vote, or measure the noise level first and only treat larger differences as meaningful.
- workers 1 is the simplest option at small scale. It is enough for small experiments that clearly test a prompt's effect, but it gets slow as the scale grows.

# temp 0과 고정 seed로도 LLM 출력이 같다는 보장은 없다 (batch invariance)

temperature 0과 고정된 seed가 도움이 되지만, 배치 구성과 스케줄링이 달라지면 동일한 출력을 보장하지 않는다.

**뜻:** GPU가 계산을 나누는 방식이 "지금 동시에 계산하는 요청이 몇 개냐"에 따라 달라진다는 것이다.

비유하면 이렇다. 숫자 1,000개를 더하는 일을 직원들에게 나눠 맡긴다고 해 보자.

- 일이 많아 직원이 모두 바쁘면, 한 사람이 1,000개를 처음부터 끝까지 혼자 더한다.
- 일이 적어서 직원이 놀고 있으면, 10명이 100개씩 나눠 더한 다음 부분합을 다시 더한다.

수학적으로는 답이 같아야 하지만, 컴퓨터는 매 단계마다 반올림을 하기 때문에 끝자리가 약간 달라질 수 있다. 같은 요청이라도 혼자 들어왔는지, 7개처럼 묶였는지에 따라 GPU가 나누는 방식이 달라지고, 그래서 결과의 끝자리가 달라진다. 이게 "batch-invariant하지 않다"는 말이다. workers 1(요청을 한 번에 하나씩 보내기)이면 항상 혼자 들어가니 매번 같은 방식으로 나뉘고, 그래서 결과도 같다(같은 서버를 다른 요청이 함께 쓰지 않는다면).

**다른 사람들은 다 workers 1로 돌리나?** 아니다, 대부분은 그러지 않는다.

- 대부분의 서비스는 흔들림을 그냥 받아들인다. 챗봇처럼 속도와 처리량이 중요한 곳은 동시 요청이 기본이고, 답이 조금 달라도 문제가 되지 않는다.
- 재현성이 꼭 필요한 경우에는 batch invariant 모드를 켠다. 평가, 강화학습, 감사 기록 같은 곳이다. 동시 요청의 속도를 거의 유지하면서 결과가 동일하게 나온다.
  - vLLM에서는 `VLLM_BATCH_INVARIANT=1`로 켠다.
  - compute capability 9.0 이상인 NVIDIA GPU가 필요하며, 명시된 GPU는 H-series(H100, H200)와 B-series(B100, B200)다.
  - 켜면 기본 모드보다 느려질 수 있고, 문서는 이를 재현성을 위한 의도된 맞교환이라고 설명한다.
- 흔들림을 전제로 설계하는 곳도 많다. 여러 번 돌려서 다수결을 내거나, 노이즈 수준을 먼저 재고 그보다 큰 차이만 유의미하다고 본다.
- workers 1은 소규모로 사용하기에 가장 간단한 방법이다. 프롬프트 효과를 명확히 확인하기에는 소규모 실험으로 충분하다. 대신 규모가 커지면 느려진다.

## Sources
- vLLM docs v0.19.1, Batch Invariance: https://docs.vllm.ai/en/v0.19.1/features/batch_invariance/
