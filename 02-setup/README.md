# 02. 랭플로우 설치 방법

## 설치 옵션

Langflow는 4가지 방법으로 설치할 수 있습니다:

| 방법 | 추천 대상 | 난이도 |
|------|----------|--------|
| Desktop 앱 | 가장 간편하게 시작하고 싶은 분 | ⭐ |
| Docker | 환경 격리가 필요한 분 | ⭐⭐ |
| pip (Python 패키지) | Python 개발 환경이 갖춰진 분 | ⭐⭐ |
| 소스 설치 | 소스 코드를 수정하고 싶은 분 | ⭐⭐⭐ |

---

## 사전 요구사항

### Python 버전
- **macOS/Linux**: Python 3.10 ~ 3.13
- **Windows**: Python 3.10 ~ 3.12

### 하드웨어
- **최소**: 듀얼코어 CPU, 2GB RAM
- **권장**: 멀티코어 CPU, 4GB+ RAM

### 패키지 매니저
- `uv` (권장) 또는 `pip`

---

## 방법 1: Desktop 앱 (가장 간편)

[Langflow Desktop 다운로드 페이지](https://www.langflow.org/desktop)에서 운영체제에 맞는 설치 파일을 다운로드하여 설치합니다.

> **참고**: Windows 사용자는 Microsoft C++ Build Tools가 필요할 수 있습니다.

---

## 방법 2: Docker (권장)

Docker가 설치되어 있다면, 한 줄로 실행할 수 있습니다:

```bash
docker run -p 7860:7860 langflowai/langflow:latest
```

실행 후 브라우저에서 접속:
```
http://localhost:7860
```

### Docker Compose 사용

```yaml
# docker-compose.yml
version: "3.8"
services:
  langflow:
    image: langflowai/langflow:latest
    ports:
      - "7860:7860"
    environment:
      - LANGFLOW_DATABASE_URL=sqlite:///./langflow.db
    volumes:
      - langflow_data:/app/langflow

volumes:
  langflow_data:
```

```bash
docker compose up -d
```

---

## 방법 3: pip (Python 패키지)

### Step 1: 가상환경 만들기

```bash
# uv 사용 (권장)
uv venv langflow-env

# 또는 venv 사용
python -m venv langflow-env
```

### Step 2: 가상환경 활성화

```bash
# macOS/Linux
source langflow-env/bin/activate

# Windows
langflow-env\Scripts\activate
```

### Step 3: Langflow 설치

```bash
# uv 사용 (권장)
uv pip install langflow

# 또는 pip 사용
pip install langflow
```

특정 버전을 설치하려면:
```bash
uv pip install langflow==1.4.22
```

### Step 4: Langflow 실행

```bash
# uv 사용
uv run langflow run

# 또는 직접 실행
langflow run
```

실행 후 브라우저에서 접속:
```
http://127.0.0.1:7860
```

---

## 업그레이드

```bash
# 최신 버전으로 업그레이드
uv pip install langflow -U

# 의존성 포함 재설치
uv pip install langflow --force-reinstall
```

---

## 설치 확인

Langflow가 정상적으로 실행되면 브라우저에서 다음과 같은 화면을 볼 수 있습니다:

1. **로그인/회원가입** 화면 (첫 실행 시)
2. **My Projects** 대시보드
3. **New Flow** 버튼으로 새 프로젝트 생성 가능

---

## API 키 설정

외부 LLM을 사용하려면 API 키를 설정해야 합니다:

1. Langflow UI 상단의 **Settings** 클릭
2. **Model Providers** 섹션에서 사용할 제공자 선택 (예: OpenAI)
3. API 키 입력 후 저장

### OpenAI API 키 발급

1. [platform.openai.com](https://platform.openai.com)에서 계정 생성
2. API Keys 메뉴에서 새 키 생성
3. Langflow Settings > Model Providers > OpenAI에 키 입력

---

## 문제 해결

### 포트 충돌
```bash
# 다른 포트로 실행
langflow run --port 7861
```

### 메모리 부족
- Docker 사용 시 Docker Desktop에서 메모리 할당량을 4GB 이상으로 설정

### Python 버전 호환
```bash
# Python 버전 확인
python --version
```

---

## 다음 단계

설치가 완료되었다면 [03-basic-flows](../03-basic-flows/)에서 첫 번째 플로우를 만들어봅시다!
