# OpenCode Observability Hands On

[English README](README_en.md)

LiteLLM + Phoenix + OpenCode를 이용하여 OpenCode의 내부 동작 원리를 알아보는 실습 프로젝트입니다.
로컬 개발 환경에서 OpenCode의 요청을 LiteLLM Proxy로 보내고, 이를 다시 실제 LLM 공급자(Google Gemini, Grok 등)로 라우팅하며, 모든 요청/응답 로그를 Phoenix를 통해 관찰합니다.

## 🚀 구성 요소 (Components)

- **LiteLLM Proxy**: OpenAI 호환 프록시로, 클라이언트의 요청을 받아 적절한 LLM 모델로 전달합니다.
- **Phoenix**: LLM 실행 추적 및 관측(Observability)을 위한 UI 도구입니다.
- **Postgres**: 프록시의 메타데이터를 저장합니다.
- **OpenCode**: 사용자가 상호작용하는 AI 코딩 에이전트 CLI 도구입니다.

## 📁 주요 파일 (Key Files)

- `docker-compose.yml`: Proxy, Phoenix, Postgres 컨테이너를 실행합니다.
- `litellm_config.yaml`: 모델 별명(Alias) 및 실제 모델 ID 매핑을 정의합니다.
- `.env`: API Key 및 Proxy 설정 파일입니다.
*   `opencode.json`: OpenCode 설정 파일로, 로컬 프록시(`dohyun_litellm`) 연결 정보가 포함되어 있습니다. (전역 설정을 원할 경우 `~/.config/opencode/opencode.json` 경로로 복사하여 사용 가능합니다.)

## ▶️ 실행 방법 (How to Run)

### 1) API Key 설정

- `.env.example` 파일을 복사하여 `.env` 파일을 생성합니다.
- Grok Code Fast 1 모델을 무료로 사용하려면 [https://opencode.ai/zen](https://opencode.ai/zen)에서 키를 발급받아 `OPENCODE_ZEN_API_KEY`에 입력합니다.
- (선택사항) Google AI Studio ([https://aistudio.google.com/api-keys](https://aistudio.google.com/api-keys))에서 발급받은 키를 `GEMINI_API_KEY`에 입력합니다.

### 2) 컨테이너 시작

```bash
docker compose up -d
```

### 3) OpenCode 실행 및 모델 선택

```bash
opencode
```

- OpenCode TUI 상에서 설정된 모델 프로바이더(`litellm by dohyun (local)`)를 선택하여 사용합니다.

## 🔎 관측 (Observability)

Phoenix UI에 접속하여 실시간으로 LLM 요청/응답 로그를 확인합니다:
👉 [http://localhost:6006](http://localhost:6006)

## 🧪 Proxy 직접 호출 테스트 (Direct Proxy Call)

### 1) 모델 리스트 확인

현재 프록시에서 사용 가능한 전체 모델 목록을 조회합니다.

```bash
curl -s -H "Authorization: Bearer sk-1234567890" http://localhost:4000/v1/models | jq
```

### 2) Gemini 모델 테스트 (Google SDK/API 스타일)

`gemini-2.5-flash-lite` 모델을 Google AI 스타일의 엔드포인트로 호출하는 예시입니다.

```bash
curl -X POST "http://localhost:4000/v1beta/models/gemini-2.5-flash-lite:generateContent" \
  -H "Authorization: Bearer sk-1234567890" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [
      { "parts": [{ "text": "안녕, 너는 누구니?" }], "role": "user" }
    ]
  }' | jq
```

### 3) Grok Code 모델 테스트 (OpenAI 스타일)

`grok-code` 모델을 대상으로 OpenAI 호환 엔드포인트(`/v1/chat/completions`)를 호출하는 예시입니다.

```bash
curl -X POST "http://localhost:4000/v1/chat/completions" \
  -H "Authorization: Bearer sk-1234567890" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "grok-code",
    "messages": [
      { "role": "user", "content": "안녕, 너는 누구니?" }
    ]
  }' | jq
```

## ❗ 트러블슈팅 (Troubleshooting)

- `opencode.json`의 `baseURL`이 프록시 주소(`http://localhost:4000/v1`)와 일치하는지 확인하세요.
- `.env` 파일에 유효한 API Key가 설정되어 있는지 확인하세요.
- LiteLLM 로그 확인:
  ```bash
  docker logs -f litellm
  ```
