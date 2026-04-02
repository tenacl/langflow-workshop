# 06. 실습 과제

이전 챕터에서 배운 내용을 직접 실습해봅시다. 난이도 순서대로 3개의 과제를 준비했습니다.

---

## 과제 1: 번역 챗봇 만들기 (난이도: ⭐ 쉬움)

### 목표
사용자가 입력한 한국어를 영어로 번역하는 챗봇 플로우를 만듭니다.

### 사용할 컴포넌트
- Chat Input
- Prompt
- OpenAI (또는 다른 LLM)
- Chat Output

### 단계별 가이드

1. **새 플로우 생성**: New Flow > Blank Flow

2. **컴포넌트 배치**: 위의 4개 컴포넌트를 캔버스에 드래그

3. **Prompt 작성**:
```
당신은 전문 번역가입니다.
사용자가 입력한 한국어 텍스트를 자연스러운 영어로 번역하세요.
번역만 출력하고, 추가 설명은 하지 마세요.

한국어 입력: {user_message}
영어 번역:
```

4. **컴포넌트 연결**:
```
Chat Input → Prompt(user_message) → OpenAI → Chat Output
```

5. **LLM 설정**: 모델과 API 키 설정

6. **테스트**: Playground에서 한국어를 입력하고 영어 번역이 나오는지 확인

### 확인 포인트
- [ ] 한국어 입력 시 영어 번역이 정상 출력되는가?
- [ ] 번역만 출력되고 추가 설명이 없는가?
- [ ] 긴 문장도 정상적으로 번역되는가?

### 추가 도전
- 영어→한국어 역번역도 지원하도록 Prompt를 수정해보세요
- DropdownInput으로 대상 언어를 선택할 수 있게 만들어보세요

---

## 과제 2: 도구를 활용하는 에이전트 (난이도: ⭐⭐ 중간)

### 목표
Calculator 도구와 URL 도구를 활용하여, 수학 계산과 웹 정보 검색을 동시에 수행할 수 있는 에이전트를 만듭니다.

### 사용할 컴포넌트
- Chat Input
- Agent
- Calculator (Tool)
- URL Tool
- Chat Output

### 단계별 가이드

1. **Simple Agent 템플릿으로 시작**: New Flow > Simple Agent

2. **Agent 설정**:
   - Language Model: `gpt-4o` 또는 `gpt-4o-mini` 선택
   - API 키 확인

3. **도구 연결 확인**:
   - Calculator 도구가 Agent에 연결되어 있는지 확인
   - URL Tool이 Agent에 연결되어 있는지 확인

4. **테스트 시나리오**:

```
# 시나리오 1: 수학 계산
사용자: 127 * 365는 얼마야?
기대: Calculator 도구를 사용하여 정확한 계산 결과 출력

# 시나리오 2: 웹 정보
사용자: https://example.com 페이지의 내용을 요약해줘
기대: URL Tool을 사용하여 웹 페이지 내용을 가져와서 요약

# 시나리오 3: 복합 질문
사용자: 2024년은 몇 시간이야? (365일 * 24시간으로 계산해줘)
기대: Calculator를 사용하여 8760 계산
```

### 확인 포인트
- [ ] 수학 질문에 Calculator 도구를 사용하는가?
- [ ] URL 관련 질문에 URL Tool을 사용하는가?
- [ ] 일반 질문에는 도구 없이 직접 답변하는가?

### 추가 도전
- Search API 도구를 추가로 연결해보세요
- Agent의 System Message를 수정하여 항상 한국어로 응답하도록 설정해보세요

---

## 과제 3: API로 플로우 호출하기 (난이도: ⭐⭐⭐ 중간~어려움)

### 목표
과제 1에서 만든 번역 챗봇을 API로 호출하는 Python 스크립트를 작성합니다.

### 사전 준비
- 과제 1의 번역 챗봇 플로우가 실행 중인 상태
- Langflow API 키 생성 완료

### 단계별 가이드

#### Step 1: API 키 생성
1. Langflow UI > Settings > Langflow API Keys
2. **Add New** 클릭, 이름 입력 후 생성
3. 키를 복사하여 저장

#### Step 2: Flow ID 확인
1. 번역 챗봇 플로우 열기
2. **Share** > **API access** 클릭
3. Flow ID 확인 (URL에서도 확인 가능)

#### Step 3: Python 스크립트 작성

```python
"""
langflow_translator.py
Langflow 번역 챗봇 API 호출 예시
"""

import os
import requests


# 설정
LANGFLOW_URL = "http://localhost:7860"
FLOW_ID = "YOUR_FLOW_ID"  # 실제 Flow ID로 교체
API_KEY = os.environ.get("LANGFLOW_API_KEY", "YOUR_API_KEY")


def translate(text: str) -> dict:
    """Langflow 번역 플로우를 API로 호출합니다."""
    response = requests.post(
        f"{LANGFLOW_URL}/api/v1/run/{FLOW_ID}",
        headers={
            "Content-Type": "application/json",
            "x-api-key": API_KEY,
        },
        json={
            "input_value": text,
            "output_type": "chat",
            "input_type": "chat",
        },
    )
    response.raise_for_status()
    return response.json()


def main():
    print("=== Langflow 번역기 ===")
    print("한국어를 입력하면 영어로 번역합니다.")
    print("종료하려면 'quit'를 입력하세요.\n")

    while True:
        user_input = input("한국어 > ").strip()
        if user_input.lower() == "quit":
            print("종료합니다.")
            break
        if not user_input:
            continue

        try:
            result = translate(user_input)
            # 응답에서 텍스트 추출 (응답 구조에 따라 조정 필요)
            print(f"영어   > {result}\n")
        except requests.exceptions.RequestException as e:
            print(f"에러: {e}\n")


if __name__ == "__main__":
    main()
```

#### Step 4: Tweaks로 Temperature 조절

```python
def translate_with_tweaks(text: str, temperature: float = 0.3) -> dict:
    """Tweaks를 사용하여 LLM 파라미터를 오버라이드합니다."""
    response = requests.post(
        f"{LANGFLOW_URL}/api/v1/run/{FLOW_ID}",
        headers={
            "Content-Type": "application/json",
            "x-api-key": API_KEY,
        },
        json={
            "input_value": text,
            "output_type": "chat",
            "input_type": "chat",
            "tweaks": {
                # 컴포넌트 ID는 Share > API access에서 확인
                "OpenAIModel-abc123": {
                    "temperature": temperature,
                }
            },
        },
    )
    response.raise_for_status()
    return response.json()
```

#### Step 5: 실행

```bash
# 환경 변수로 API 키 설정
export LANGFLOW_API_KEY="your-api-key-here"

# 스크립트 실행
python langflow_translator.py
```

### 확인 포인트
- [ ] API 호출이 정상적으로 이루어지는가?
- [ ] 번역 결과가 정상적으로 반환되는가?
- [ ] Tweaks로 temperature 변경 시 응답이 달라지는가?
- [ ] API 키가 없을 때 적절한 에러가 발생하는가?

### 추가 도전
- cURL로도 같은 API를 호출해보세요
- 여러 문장을 한 번에 번역하는 배치 함수를 만들어보세요
- 플로우에 Endpoint Name을 설정하고 별명으로 호출해보세요

---

## 실습 완료 체크리스트

| 과제 | 핵심 학습 내용 | 완료 |
|------|-------------|------|
| 과제 1 | Prompt 템플릿, 컴포넌트 연결 | ☐ |
| 과제 2 | Agent, Tools, 자율적 도구 선택 | ☐ |
| 과제 3 | API 호출, Tweaks, 인증 | ☐ |

---

## 더 배우고 싶다면

- [Langflow 공식 문서](https://docs.langflow.org) - 전체 레퍼런스
- [Langflow GitHub](https://github.com/langflow-ai/langflow) - 소스 코드 및 이슈
- [Langflow Discord](https://discord.gg/langflow) - 커뮤니티 Q&A

수고하셨습니다! 🎉
