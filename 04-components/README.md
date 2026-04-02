# 04. 주요 컴포넌트 종류

## 개요

Langflow의 컴포넌트는 크게 **코어 컴포넌트**와 **번들 컴포넌트**로 나뉩니다.

- **코어 컴포넌트**: Langflow에 내장된 범용 컴포넌트 (Inputs, Outputs, Prompts 등)
- **번들 컴포넌트**: 외부 서비스와 연동하는 컴포넌트 (OpenAI, Pinecone 등 서비스 제공자별로 그룹화)

포트의 **색상**은 데이터 타입의 호환성을 나타내며, 같은 색상의 포트끼리 연결할 수 있습니다.

---

## 1. Inputs & Outputs (입출력)

플로우의 시작과 끝을 담당하는 컴포넌트입니다.

### Chat Input
- 사용자의 채팅 메시지를 받아 플로우에 전달
- Playground의 채팅 인터페이스와 연결

### Chat Output
- 플로우의 최종 결과를 사용자에게 출력
- Playground에서 응답으로 표시

### 사용 예시

```
[Chat Input] ──▶ ... (처리) ... ──▶ [Chat Output]
```

---

## 2. Prompts (프롬프트)

LLM에게 보낼 지시문(프롬프트)을 작성하는 컴포넌트입니다.

### 주요 기능
- **템플릿 변수**: `{variable_name}` 형식으로 동적 값을 삽입
- 변수를 추가하면 자동으로 해당 변수에 대한 입력 포트가 생성

### 예시 템플릿

```
당신은 {role} 전문가입니다.
다음 질문에 {language}로 답변하세요.

질문: {question}
```

이 프롬프트에서 `{role}`, `{language}`, `{question}`은 각각 입력 포트로 노출됩니다.

---

## 3. Models (언어 모델)

LLM API를 호출하는 컴포넌트입니다. Langflow는 다양한 LLM 제공자를 지원합니다.

### 지원 제공자 (일부)
| 제공자 | 대표 모델 |
|--------|----------|
| OpenAI | GPT-4o, GPT-4o-mini |
| Anthropic | Claude 4 Sonnet |
| Google | Gemini |
| Ollama | Llama, Mistral (로컬) |

### 설정 항목
- **Model Name**: 사용할 모델 선택
- **Temperature**: 응답의 창의성 조절 (0.0 ~ 1.0)
- **Max Tokens**: 최대 응답 길이
- **API Key**: 제공자별 인증 키

---

## 4. Memory (메모리)

대화 기록을 저장하고 관리하는 컴포넌트입니다. 메모리를 사용하면 LLM이 이전 대화 맥락을 기억할 수 있습니다.

### 활용 방식
- **대화 지속성**: 이전 대화 내용을 LLM에 컨텍스트로 전달
- **세션 관리**: 사용자별 대화 세션 분리

### 메모리가 포함된 플로우 예시

```
┌────────────┐    ┌──────────┐    ┌──────────┐    ┌─────────────┐
│ Chat Input │───▶│  Prompt  │───▶│   LLM    │───▶│ Chat Output │
└────────────┘    └──────────┘    └──────────┘    └─────────────┘
                       ▲
                       │
                  ┌──────────┐
                  │  Memory  │
                  └──────────┘
```

Memory 컴포넌트의 출력을 Prompt의 변수로 연결하면, 이전 대화 내용이 프롬프트에 포함됩니다.

---

## 5. Tools (도구)

에이전트가 사용할 수 있는 기능들을 정의하는 컴포넌트입니다.

### 기본 제공 도구 (일부)
| 도구 | 설명 |
|------|------|
| Calculator | 수학 계산 수행 |
| URL Tool | 웹 페이지 내용 가져오기 |
| Search API | 검색 엔진 API 호출 |
| Python REPL | Python 코드 실행 |

### 도구의 동작 방식

도구는 단독으로 사용되지 않고, **Agent 컴포넌트에 연결**하여 사용합니다. 에이전트가 사용자의 질문을 분석하고, 적절한 도구를 자율적으로 선택하여 호출합니다.

```
┌────────────┐    ┌───────────────┐    ┌─────────────┐
│ Chat Input │───▶│    Agent      │───▶│ Chat Output │
└────────────┘    └───────────────┘    └─────────────┘
                    ▲    ▲    ▲
                    │    │    │
               ┌────┘    │    └────┐
               │         │         │
          ┌────────┐ ┌───────┐ ┌────────┐
          │  Calc  │ │  URL  │ │ Search │
          └────────┘ └───────┘ └────────┘
```

---

## 6. Agents (에이전트)

**에이전트**는 LLM을 기반으로 자율적으로 판단하고 도구를 활용하는 컴포넌트입니다.

### 에이전트 vs 일반 LLM 호출

| 특성 | 일반 LLM 호출 | 에이전트 |
|------|-------------|---------|
| 도구 사용 | X | O |
| 자율적 판단 | X | O |
| 다단계 추론 | X | O |
| 사용 패턴 | 정해진 순서대로 실행 | 상황에 따라 도구 선택 |

### 에이전트 설정 항목
- **Language Model**: 사용할 LLM 선택
- **Tools**: 에이전트가 사용할 도구 목록
- **System Message**: 에이전트의 역할과 행동 지침

---

## 7. Data (데이터)

데이터를 처리하고 변환하는 컴포넌트 그룹입니다.

### 주요 컴포넌트
- **File Loader**: 파일(PDF, TXT 등)을 로드
- **Text Splitter**: 긴 텍스트를 작은 청크(chunk)로 분할
- **Data Processor**: 데이터 구조 변환

---

## 8. Embeddings & Vector Stores

RAG(Retrieval-Augmented Generation) 파이프라인에 사용되는 컴포넌트입니다.

### Embeddings
텍스트를 벡터(숫자 배열)로 변환합니다.
- OpenAI Embeddings
- Hugging Face Embeddings
- Ollama Embeddings

### Vector Stores
벡터를 저장하고 유사도 검색을 수행합니다.
- Chroma
- Pinecone
- FAISS
- Qdrant

### RAG 플로우 예시

```
[문서 로드] ──▶ [텍스트 분할] ──▶ [Embedding] ──▶ [Vector Store에 저장]
                                                           │
[질문 입력] ──▶ [Embedding] ──▶ [Vector Store에서 검색] ◀──┘
                                        │
                                        ▼
                              [검색 결과 + 질문] ──▶ [LLM] ──▶ [답변 출력]
```

---

## 핵심 정리

| 카테고리 | 역할 | 대표 컴포넌트 |
|----------|------|-------------|
| Inputs/Outputs | 플로우 입출력 | Chat Input, Chat Output |
| Prompts | LLM 지시문 작성 | Prompt |
| Models | LLM API 호출 | OpenAI, Anthropic |
| Memory | 대화 기록 관리 | Memory |
| Tools | 에이전트용 도구 | Calculator, URL Tool |
| Agents | 자율적 도구 사용 | Agent |
| Data | 데이터 처리 | File Loader, Text Splitter |
| Embeddings | 텍스트 벡터화 | OpenAI Embeddings |
| Vector Stores | 벡터 저장/검색 | Chroma, Pinecone |

---

## 다음 단계

API 연동과 커스텀 컴포넌트를 배우려면 [05-advanced](../05-advanced/)로 이동하세요.
