# Payload

- Originally means "cargo being carried." In a rocket, the payload is the satellite it actually carries, not the fuel or the rocket body.
- In network requests and messages, it is the actual content (the body), excluding metadata such as headers.
- In LLM APIs, "payload" usually means the whole JSON request body, including settings like model, messages, and temperature.
- Depending on context, it can include those settings or refer only to the model input (prompt + material).

Example (LLM API request body):

```json
{"model": "...", "temperature": 0, "messages": [{"role": "user", "content": "<prompt + material>"}]}
```

The whole object is the payload in the API sense; the `content` string is the payload in the model-input sense.

# 페이로드

- 원래 뜻은 "운반되는 화물"이다. 로켓에서는 연료나 로켓 몸체가 아니라 실제로 실어 나르는 위성을 payload라고 부른다.
- 네트워크 요청이나 메시지에서는 헤더 같은 메타데이터를 뺀 실제 내용(body)을 가리킨다.
- LLM API에서는 보통 요청 JSON body 전체를 payload라고 한다. model, messages, temperature 같은 설정도 여기에 포함된다.
- 문맥에 따라 설정까지 포함하기도 하고, 모델 입력(프롬프트 + 자료)만 가리키기도 한다.

예시 (LLM API 요청 body):

```json
{"model": "...", "temperature": 0, "messages": [{"role": "user", "content": "<prompt + material>"}]}
```

객체 전체가 API 의미의 payload이고, `content` 문자열이 모델 입력 의미의 payload다.
