# 05. API 연동 & Custom Component

## Part 1: API 연동

Langflow에서 만든 플로우를 외부 애플리케이션에서 호출할 수 있습니다.

---

### API 엔드포인트

플로우를 실행하는 기본 엔드포인트:

```
POST /api/v1/run/{flow_id}
```

### 인증

Langflow 1.5 이상에서는 API 키 인증이 필요합니다.

**API 키 생성 방법:**
1. Langflow UI > **Settings** > **Langflow API Keys**
2. **Add New** 클릭
3. 키 이름 지정 후 생성
4. 생성된 키를 안전한 곳에 저장

---

### cURL로 플로우 호출

```bash
curl --request POST \
  --url "http://localhost:7860/api/v1/run/YOUR_FLOW_ID" \
  --header "Content-Type: application/json" \
  --header "x-api-key: YOUR_LANGFLOW_API_KEY" \
  --data '{
    "input_value": "안녕하세요, 오늘 날씨 어때요?",
    "output_type": "chat",
    "input_type": "chat"
  }'
```

### Python으로 플로우 호출

```python
import requests

LANGFLOW_URL = "http://localhost:7860"
FLOW_ID = "YOUR_FLOW_ID"
API_KEY = "YOUR_LANGFLOW_API_KEY"

def run_flow(message: str) -> str:
    """Langflow 플로우를 API로 호출합니다."""
    response = requests.post(
        f"{LANGFLOW_URL}/api/v1/run/{FLOW_ID}",
        headers={
            "Content-Type": "application/json",
            "x-api-key": API_KEY,
        },
        json={
            "input_value": message,
            "output_type": "chat",
            "input_type": "chat",
        },
    )
    response.raise_for_status()
    data = response.json()
    return data

# 사용
result = run_flow("파이썬에서 리스트 컴프리헨션이 뭐야?")
print(result)
```

---

### Tweaks: 런타임 파라미터 오버라이드

**Tweaks**를 사용하면 플로우를 수정하지 않고도 API 호출 시 특정 컴포넌트의 파라미터를 일시적으로 변경할 수 있습니다.

```bash
curl --request POST \
  --url "http://localhost:7860/api/v1/run/YOUR_FLOW_ID" \
  --header "Content-Type: application/json" \
  --header "x-api-key: YOUR_LANGFLOW_API_KEY" \
  --data '{
    "input_value": "안녕하세요",
    "output_type": "chat",
    "input_type": "chat",
    "tweaks": {
      "ChatInput-abc123": {
        "should_store_message": false
      },
      "OpenAIModel-def456": {
        "temperature": 0.1
      }
    }
  }'
```

> **참고**: 컴포넌트 ID(`ChatInput-abc123`)는 Langflow UI의 **Share > API access**에서 확인할 수 있습니다.

---

### 플로우 ID에 별명(Alias) 지정

숫자로 된 Flow ID 대신 기억하기 쉬운 이름을 사용할 수 있습니다:

1. 플로우의 **Input Schema** 설정에서 **Endpoint Name** 지정
2. 예: `my-chatbot`으로 설정하면 `/api/v1/run/my-chatbot`으로 호출 가능

---

### 웹 페이지에 챗봇 임베딩

만든 플로우를 웹 사이트에 채팅 위젯으로 삽입할 수 있습니다:

```html
<!DOCTYPE html>
<html>
<head>
  <title>내 AI 챗봇</title>
</head>
<body>
  <h1>AI 챗봇 데모</h1>

  <!-- Langflow 채팅 위젯 -->
  <script
    src="https://cdn.jsdelivr.net/gh/langflow-ai/langflow-embedded-chat@main/dist/build/static/js/bundle.min.js">
  </script>
  <langflow-chat
    host_url="https://your-server.com"
    flow_id="YOUR_FLOW_ID"
    api_key="YOUR_API_KEY">
  </langflow-chat>
</body>
</html>
```

**필수 속성:**
- `host_url`: Langflow 서버 주소 (HTTPS, 뒤에 `/` 없이)
- `flow_id`: 플로우 ID
- `api_key`: Langflow API 키

---

## Part 2: Custom Component 기초

Langflow의 기본 컴포넌트만으로 부족할 때, Python으로 직접 커스텀 컴포넌트를 만들 수 있습니다.

---

### Custom Component 구조

커스텀 컴포넌트는 `Component` 클래스를 상속하여 만듭니다:

```python
from langflow.custom import Component
from langflow.io import StrInput, Output
from langflow.schema import Data


class MyFirstComponent(Component):
    # 메타데이터
    display_name = "My First Component"
    description = "텍스트를 대문자로 변환하는 컴포넌트"
    icon = "type"  # Lucide 아이콘 이름

    # 입력 정의
    inputs = [
        StrInput(
            name="input_text",
            display_name="입력 텍스트",
        ),
    ]

    # 출력 정의
    outputs = [
        Output(
            name="result",
            display_name="변환 결과",
            method="transform",
        ),
    ]

    # 처리 로직
    def transform(self) -> Data:
        result = self.input_text.upper()
        self.status = f"변환 완료: {result}"
        return Data(data={"output": result})
```

---

### 핵심 요소 설명

#### 클래스 속성 (메타데이터)

| 속성 | 설명 |
|------|------|
| `display_name` | UI에 표시될 컴포넌트 이름 |
| `description` | 컴포넌트 설명 |
| `icon` | [Lucide](https://lucide.dev/) 아이콘 이름 |
| `documentation` | 문서 링크 URL |

#### 입력 타입 (Inputs)

```python
from langflow.io import (
    StrInput,          # 한 줄 텍스트
    MultilineInput,    # 여러 줄 텍스트
    IntInput,          # 정수
    BoolInput,         # 불리언 토글
    DropdownInput,     # 선택 메뉴
    FileInput,         # 파일 업로드
    DataInput,         # Data 객체
)
```

입력 필드의 `name`은 `self.<name>`으로 접근할 수 있습니다.

#### 출력 (Outputs)

각 출력은 `method` 속성으로 실행할 함수를 지정합니다:

```python
outputs = [
    Output(
        name="processed",
        display_name="처리 결과",
        method="process_data",  # self.process_data() 호출
    ),
]
```

---

### 다양한 입력 타입 예시

```python
from langflow.custom import Component
from langflow.io import StrInput, IntInput, BoolInput, DropdownInput, Output
from langflow.schema import Data


class TextProcessor(Component):
    display_name = "Text Processor"
    description = "다양한 옵션으로 텍스트를 처리합니다"

    inputs = [
        StrInput(
            name="text",
            display_name="텍스트",
        ),
        DropdownInput(
            name="mode",
            display_name="처리 모드",
            options=["uppercase", "lowercase", "title"],
            value="uppercase",  # 기본값
        ),
        IntInput(
            name="repeat_count",
            display_name="반복 횟수",
            value=1,
        ),
        BoolInput(
            name="add_prefix",
            display_name="접두사 추가",
            value=False,
        ),
    ]

    outputs = [
        Output(
            name="result",
            display_name="결과",
            method="process",
        ),
    ]

    def process(self) -> Data:
        text = self.text

        # 모드에 따른 변환
        if self.mode == "uppercase":
            text = text.upper()
        elif self.mode == "lowercase":
            text = text.lower()
        elif self.mode == "title":
            text = text.title()

        # 반복
        text = text * self.repeat_count

        # 접두사
        if self.add_prefix:
            text = "[Processed] " + text

        return Data(data={"output": text})
```

---

### 반환 타입

| 타입 | 용도 | 예시 |
|------|------|------|
| `Data` | 구조화된 데이터 | `Data(data={"key": "value"})` |
| `Message` | 채팅 메시지 | `Message(text="Hello!", sender="System")` |
| `DataFrame` | 표 형식 데이터 | `DataFrame(pd.DataFrame(...))` |

---

### 에러 처리

```python
def process(self) -> Data:
    if not self.text:
        raise ValueError("텍스트를 입력해주세요.")

    try:
        result = some_operation(self.text)
        self.status = f"성공: {len(result)} 글자 처리됨"
        return Data(data={"output": result})
    except Exception as e:
        self.status = f"에러: {str(e)}"
        return Data(data={"error": str(e)})
```

---

### Custom Component 배포

Docker 환경에서 커스텀 컴포넌트를 사용하려면:

1. 컴포넌트 파일을 카테고리 폴더에 저장:
```
custom_components/
└── processors/
    ├── __init__.py
    └── text_processor.py
```

2. `__init__.py`에 등록:
```python
from .text_processor import TextProcessor

__all__ = ["TextProcessor"]
```

3. 환경 변수로 경로 지정:
```bash
docker run -p 7860:7860 \
  -v ./custom_components:/app/custom_components \
  -e LANGFLOW_COMPONENTS_PATH=/app/custom_components \
  langflowai/langflow:latest
```

---

## 핵심 정리

| 주제 | 핵심 내용 |
|------|----------|
| API 엔드포인트 | `POST /api/v1/run/{flow_id}` |
| 인증 | `x-api-key` 헤더에 API 키 |
| Tweaks | 런타임에 컴포넌트 파라미터 오버라이드 |
| 임베딩 | `langflow-chat` 웹 컴포넌트로 사이트에 삽입 |
| Custom Component | `Component` 클래스 상속, inputs/outputs 정의 |

---

## 다음 단계

배운 내용을 실습해보려면 [06-exercises](../06-exercises/)로 이동하세요.
