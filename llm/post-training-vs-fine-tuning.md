# Post-training vs fine-tuning

## Pre-training

The model learns to predict the next word on a huge amount of data, such as text from the internet. Language and knowledge come from this stage, but at this point the model is not yet an assistant that follows instructions well.

## Post-training

Every training step that turns a pre-trained model into a usable one.

- SFT (supervised fine-tuning): train on examples of good input–response pairs.
- Preference learning such as RLHF/RLAIF: reward better responses.
- Safety training.

A model like Claude follows instructions and writes good explanations because of the post-training its developer has done.

## Fine-tuning

Train an already-trained model again on data for a specific task, adjusting its weights. Technically it is a form of post-training, but in practice the terms are used like this:

- "post-training" is the general stage the model developer does.
- "fine-tuning" is a user adapting the model to their own task.

There are two main approaches:

- Full fine-tuning, which trains all weights.
- PEFT methods like LoRA, which train only a small part.

For a classification task, this would be SFT on "document → label (+ explanation)" pairs.

## How it differs from prompting

Prompting leaves the model as it is and changes its behavior by putting instructions (rules, criteria) into each call. Changes to the prompt apply immediately and are easy to version. Fine-tuning has the model learn the criteria directly from examples.

## Why it can help

1. **Some criteria are hard to write down.** Whether a domain expert would see something as a concern can be a subtle judgment that differs by organization. With prompting, you look at missed cases and add rules by hand. That means a person finds a pattern and puts it into words, and as cases pile up the prompt gets longer and overfits. Fine-tuning learns that boundary directly from the labels.
2. **Existing labels get used for training.** Prompting uses labels only for evaluation; fine-tuning trains on them.
3. **Fewer flips between runs.** Some borderline cases change between runs even at temperature 0. Fine-tuning doesn't remove the numerical nondeterminism behind this (see [batch invariance](batch-invariance.md)). But a model trained on the task usually becomes more decisive on familiar cases, so fewer outputs flip.
4. **Cost and speed.** Fine-tuning a smaller model can sometimes reach similar performance more cheaply and quickly than calling a large model.
5. **Scores you can split.** A fine-tuned classifier can output a score such as the probability of Yes. A model's self-reported confidence is often "High" for almost everything, so it can't separate cases. A calibrated score lets you split results into sensitivity tiers.

## Limits

- **It copies the labels more faithfully.** With prompting, you can balance the written criteria against the labels. Fine-tuning builds the labelers' biases straight into the model. Label quality sets the ceiling.
- **Too few positives.** A few dozen positive examples is small to learn from, and the imbalance is severe.
- **Less flexible.** When the criteria change, you have to retrain. You can't just edit text and bump a version as with a prompt.
- **Limited support on managed services.** In Amazon Bedrock, Claude 3 Haiku was the only Claude model that could be fine-tuned, and it reached end of life on September 10, 2026. Otherwise you have to fine-tune and deploy an open model yourself.

## In one line

When adding rules by hand reaches its limit, let the model learn the criteria directly from labels. The precondition is trustworthy labels. Until then, a step in between is to put similar labeled cases into the prompt as few-shot examples, which uses the labels without changing any weights.

# 사후 학습 vs 미세 조정

## Pre-training (사전 학습)

인터넷 텍스트 같은 대량의 데이터로 "다음 단어 맞히기"를 학습하는 단계다. 언어와 지식은 여기서 생기지만, 이 상태로는 지시를 잘 따르는 어시스턴트가 아니다.

## Post-training (사후 학습)

사전 학습이 끝난 모델을 쓸 만한 모델로 다듬는 모든 학습 단계를 말한다.

- SFT(supervised fine-tuning): 좋은 입력–응답 예시를 보여주며 학습시킨다.
- RLHF/RLAIF 같은 선호 학습: 더 나은 응답에 보상을 준다.
- 안전성 학습

Claude 같은 모델이 지시를 따르고 설명을 잘 쓰는 건 개발사가 해 둔 post-training 덕분이다.

## Fine-tuning (미세 조정)

이미 학습된 모델을 특정 작업 데이터로 한 번 더 학습시켜 가중치를 조정하는 것이다. 기술적으로는 post-training의 한 형태다. 다만 실무에서는 이렇게 구분해서 쓴다.

- "post-training"은 모델 개발사가 하는 범용 단계
- "fine-tuning"은 사용자가 자기 작업에 맞추는 단계

방식은 크게 두 가지다.

- 전체 가중치를 학습하는 full fine-tuning
- 일부만 학습하는 LoRA 같은 PEFT

분류 작업이라면 "문서 → 라벨 (+설명)" 쌍으로 하는 SFT가 된다.

## prompting과의 차이

prompting은 모델은 그대로 두고, 호출할 때 지시문(규칙, 기준)을 넣어서 행동을 바꾼다. 프롬프트를 바꾸면 바로 반영되고, 버전 관리도 쉽다. fine-tuning은 모델이 예시에서 기준을 직접 학습하게 한다.

## 왜 도움이 될 수 있나

1. **기준이 글로 다 쓰기 어려운 종류일 때가 있다.** 전문가가 우려로 보는지는 조직마다 미묘하게 달라질 수 있는 판단이다. prompting에서는 놓친 사례를 보고 규칙을 수동으로 추가한다. 사람이 패턴을 찾아 말로 옮기는 과정이고, 사례가 쌓일수록 프롬프트가 길어지고 과적합된다. fine-tuning은 라벨에서 그 경계를 직접 학습한다.
2. **이미 있는 라벨을 학습에 쓴다.** prompting은 라벨을 평가에만 쓰고, fine-tuning은 학습에 쓴다.
3. **실행마다 답이 덜 뒤집힌다.** temperature 0에서도 실행마다 바뀌는 경계 사례가 있다. 이 흔들림의 원인인 계산상의 비결정성은 fine-tuning으로 없어지지 않는다([batch invariance](batch-invariance.md) 참고). 다만 작업에 맞춰 학습된 모델은 익숙한 사례에서 보통 더 확실하게 판단해서, 답이 뒤집히는 경우가 줄어든다.
4. **비용과 속도.** 작은 모델을 fine-tuning하면 큰 모델을 부르는 것보다 더 저렴하고 빠르게 비슷한 성능을 낼 수도 있다.
5. **나눌 수 있는 점수가 생긴다.** fine-tuning한 분류 모델은 Yes일 확률 같은 점수를 낼 수 있다. 모델이 스스로 말하는 confidence는 거의 다 "High"로 나와서 구분이 안 되는 경우가 많다. 보정된 점수가 있으면 결과를 민감도 단계로 나눌 수 있다.

## 한계

- **라벨을 더 충실히 복제한다.** prompting은 글로 쓴 기준과 라벨 사이에서 조율할 수 있다. fine-tuning은 라벨을 단 사람의 편향을 그대로 모델에 반영한다. 라벨 품질이 곧 상한이다.
- **양성 예시가 너무 적다.** 수십 건은 학습하기에는 적고 불균형이 심하다.
- **유연성이 떨어진다.** 기준이 바뀌면 다시 학습해야 한다. 프롬프트처럼 텍스트만 고쳐서 버전을 올릴 수 없다.
- **관리형 서비스에서는 지원이 제한된다.** Amazon Bedrock에서 fine-tuning이 가능했던 Claude 모델은 Claude 3 Haiku뿐이었고, 2026년 9월 10일에 지원이 끝났다. 그 밖에는 오픈 모델을 직접 fine-tuning해서 배포해야 한다.

## 한 줄로 말하면

"규칙을 손으로 늘리는 방식이 한계에 이르면, 라벨에서 기준을 직접 배우게 하자." 그 전제는 믿을 만한 라벨이다. 그 사이 단계로는 라벨된 비슷한 사례를 프롬프트 예시로 넣는 방법(few-shot)이 있다. 가중치를 바꾸지 않고 라벨을 활용할 수 있다.

## Sources
- Ouyang et al., Training language models to follow instructions with human feedback (InstructGPT, 2022): https://arxiv.org/abs/2203.02155
- Hu et al., LoRA: Low-Rank Adaptation of Large Language Models (2021): https://arxiv.org/abs/2106.09685
- AWS Machine Learning Blog, Fine-tune Anthropic's Claude 3 Haiku in Amazon Bedrock (includes the end-of-support notice): https://aws.amazon.com/blogs/machine-learning/fine-tune-anthropics-claude-3-haiku-in-amazon-bedrock-to-boost-model-accuracy-and-quality
- Related TIL: [batch invariance](batch-invariance.md)
