# AGENTS.md

Meetbowl(회의 예약 → 화상 회의 → STT → AI 회의록 → 메일 공유) 통합 레포. 서비스 5개는 git 서브모듈이다.

## 받기
- `git clone --recurse-submodules https://github.com/Dawit-lee/be25-fin-thanos-meetbowl.git`
- 이미 받았으면: `git submodule update --init`
- 세부 규칙은 각 서브모듈의 `AGENTS.md`가 우선한다. 이 파일은 입구만 안내한다.

## 명령 (각 폴더 안에서 실행)
- `meetbowl-be` (Spring Boot, Gradle 멀티모듈): `./gradlew check`(Spotless 포맷 + 테스트 + ArchUnit), 포맷 맞추기 `./gradlew spotlessApply`, 실행 `./gradlew :app-api:bootRun`
  - 테스트에 Redis(6379)가 필요하다: `cd meetbowl-infra && docker compose up -d redis`
- `meetbowl-ai` (FastAPI, Python 3.12): `GOOGLE_API_KEY=dummy uv run pytest -q` (키가 없으면 테스트 2개가 실패)
- `meetbowl-stt` (Node, TypeScript): `npm ci && npm test && npm run build`
- `meetbowl-fe` (Vue, Vite): `npm ci && npm test && npm run build`
- `meetbowl-infra` (Docker Compose): `cp .env.example .env && docker compose up -d`, STT까지 `docker compose --profile stt up -d`

## 구조
- `meetbowl-be` 인증·회의·회의록·메일·워크스페이스. `app-api` → `application` → `domain` ← `infrastructure`
- `meetbowl-ai` 요약·임베딩·RAG 챗봇(Qdrant)
- `meetbowl-stt` LiveKit 오디오 수신 → STT → Transcript 이벤트
- `meetbowl-fe` 사용자 화면
- `meetbowl-infra` MariaDB·Redis·RabbitMQ·LiveKit·Qdrant 실행 환경
- `file/pdf` 기획서·명세서 등 산출물

## 규칙
- MariaDB에 접근하는 서버는 `meetbowl-be` 하나다. ai·stt는 be API나 메시지로만 저장을 요청한다.
- 서비스 간 이벤트 이름·Envelope은 `meetbowl-be`·`meetbowl-stt`의 `docs/conventions.md`를 따른다. 바꾸면 보내는 쪽·받는 쪽을 함께 고친다.
- 코드 변경은 해당 서브모듈 레포에 PR로 올리고, 이 레포는 서브모듈 커밋 포인터만 올린다.

## 하지 말 것
- `.env`, `OPENAI_API_KEY`, 내부 토큰(`MEETBOWL_INTERNAL_TOKEN`)을 커밋하지 않는다.
- 운영 DB에 직접 SQL을 실행하지 않는다.
- be의 ArchUnit 규칙(예: Controller → Repository 직접 의존)을 우회하지 않는다.
