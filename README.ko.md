[한국어](README.ko.md) | [English](README.md) | [日本語](README.ja.md)

# LLM Orchestrator (가칭)

> 여러 LLM을 하나의 workflow 안에서 협업시키고, 그 사이의 정보 전달과 맥락 관리를 시스템이 담당하는 개발 환경.

## 왜 만드는가

여러 LLM을 개발 과정에서 동시에 쓰다 보면 결국 사람이 이런 일을 하게 된다.

```
LLM A에게 질문 → 결과 복사 → LLM B에게 전달 → B의 리뷰 복사
→ 다시 A에게 전달 → 사람이 결과 비교 → 필요한 내용 정리
→ 다시 다른 AI에게 전달 …
```

이 과정에서 context loss, 정보 누락, 오래된 버전 전달, 복사/붙여넣기 실수,
사람이 중간에서 해석하며 생기는 왜곡, 불필요한 coordination overhead가 반복적으로 발생한다.
AI 자체의 export 기능도 원래 맥락을 100% 보존한다고 보장하기 어렵다.

가장 근본적인 질문은 하나다.

> **"왜 내가 AI들 사이에서 메신저 역할을 해야 하지?"**

이 프로젝트는 이 질문에서 출발한다. 사람이 AI 사이를 오가며 정보를 전달하는 대신,
**AI들이 서로 필요한 정보를 직접 주고받고 협업하게 하고, 사람은 목표 설정·판단·선택·승인만 담당**하도록 만든다.

## 무엇을 만드는가

단순한 Multi-LLM Chat App이나 AI 코딩 도구가 아니다. 목표는:

> 여러 LLM을 하나의 workflow에서 협업시키고, 그 사이의 context와 정보 전달을 자동화하는 **LLM orchestration 환경**.

개발을 첫 use case로 삼지만, 그 자체에 한정하지 않는다. 동일한 orchestration 구조는
기획/DX, 연구, 일반 사무 등 "여러 소스를 모아 분석하고 검증한 뒤 사람이 판단하는" 모든 워크플로우에 적용 가능하다.

## 핵심 원칙

| # | 원칙 |
|---|---|
| 1 | Multi-LLM 자체가 목적이 아니다 — 핵심은 AI 간 coordination의 자동화 |
| 2 | 사람이 AI 사이에서 정보를 전달하는 과정을 제거한다 |
| 3 | Chat history를 단순 전달하는 대신 **Structured Shared Context**를 사용한다 |
| 4 | AI는 서로 직접 결과를 검토하고 수정할 수 있어야 한다 |
| 5 | Cross-check는 단순 다수결이 아니라 **검증**이다 |
| 6 | Fact-check에서는 결론이 아니라 **근거 자체**를 cross-check한다 |
| 7 | Weight는 Truth Score가 아니라 **Priority/Influence**다 (`Weight ≠ Truth`) |
| 8 | 모델 선택권은 항상 사용자에게 있다 |
| 9 | Provider와 Model은 분리해서 다룬다 |
| 10 | 모델명을 core에 하드코딩하지 않는다 — **Dynamic Model Discovery** |
| 11 | Token/Cost는 사용자가 직접 제어할 수 있는 요소다 |
| 12 | Workflow 강도는 Quick / Review / Debate / Deep로 조절 가능하다 |
| 13 | **Human은 항상 최종 의사결정자다** |
| 14 | 초기 진입점은 VS Code Extension, Core는 그와 분리해서 설계한다 |
| 15 | 장기적으로 standalone LLM workspace로 확장 가능해야 한다 |

## 인간과 AI의 역할 분리

**AI가 담당**: 자료조사 · 요구사항 분석 · 기획 · 시스템 설계 · 구현 · 코드 리뷰 · 반박 · 비교 · 검증 · 수정안 제시 · 사실관계 확인

**개발자가 담당**: 목표 설정 · 중요 요구사항 결정 · trade-off 판단 · 결과 비교 · 최종 선택 · 승인

```
AI 의견 → 개발자 검토 → 선택 → 승인 → 적용
```

AI가 개발자를 대체하는 구조가 아니라, 여러 AI를 쓰는 데 필요한 **중간 전달 작업을 제거하는 구조**다.

## 개발 Workflow (예시)

```
Idea → Requirements → Research → Design → Architecture
→ Implementation Plan → Code → Review → Revision → Verification
```

기획부터 개발·검증까지 하나의 LLM 환경 안에서 이어지는 것을 지향한다. 예시 흐름:

1. 아이디어를 대략 설명
2. LLM들이 요구사항을 구체화하고 모호한 부분을 파악
3. 관련 자료 조사
4. 여러 LLM이 설계안 작성 → 서로 검토 → trade-off 비교
5. 개발자가 설계 선택 → Project Context에 저장
6. 구현 계획 → 코드 생성 → 다른 LLM이 리뷰 → 원 저자 LLM이 반영
7. 테스트/검증 → 개발자 최종 승인

## Cross-check: 다수결이 아니라 검증

여러 모델의 결과를 나란히 보여주는 게 아니라, 같은 이슈를 clustering하고
모델별 동의/반대와 근거를 함께 보여준다.

```
Issue #12 — service.py:47 — Potential None handling bug
  GPT     → Agree
  Claude  → Agree
  Gemini  → Disagree (근거: ...)
```

2개가 동의하고 1개가 반대한다고 자동으로 "다수 쪽이 맞다"고 처리하지 않는다.
최종 판단은 근거를 본 인간이 한다.

## Fact-check: 근거를 cross-check한다

모델의 결론이 아니라 **근거(Source/Evidence)**를 서로 검증하게 한다.
시스템 내부적으로 다음 네 가지를 절대 하나의 점수로 합치지 않는다:

- Priority (설정된 우선순위)
- Confidence (모델이 표명한 확신도)
- Evidence (실제 근거)
- Consensus (모델 간 합의 여부)

## Provider / Model / Dynamic Discovery

```
Provider Adapter → Model Discovery → Model Registry → Workflow
```

- **Provider**(OpenAI, Anthropic, Google …)와 **Model**(GPT, Claude Sonnet, Claude Opus, Gemini …)을 분리해서 설계한다. 사용자가 선택하는 단위는 모델이다.
- 모델명을 애플리케이션에 하드코딩하지 않는다. Provider Adapter가 동적으로 사용 가능한 모델을 조회하고, Core는 generic model abstraction으로 다룬다.
- 새 모델이 출시돼도 core 수정 없이 discovery를 통해 노출되는 것이 목표.

## Shared Project Context

Chat history 전달 대신 구조화된 프로젝트 컨텍스트를 유지한다.

```
Project Context
├── Requirements
├── Constraints
├── Architecture
├── Decisions      (근거 / 모델별 찬반 / 승인자 포함)
├── Research
├── Source Evidence
├── Current Code
├── Open Issues
└── Change History
```

이 구조가 곧 "AI들의 공통 context"가 되며, 어떤 결정을 왜 했는지·누가 승인했는지를 추적 가능하게 한다.

## Workflow 강도 (Cost/Latency 제어)

| Mode | 설명 |
|---|---|
| ⚡ Quick | 1개 LLM, 빠른 응답, 낮은 비용 |
| 🔍 Review | Generate → 2~3개 LLM Review |
| ⚔️ Debate | 여러 모델이 의견 제시 → 반박 → 재검토 |
| 🧠 Deep | Generate → Review → Debate → Revise → Re-review → Verify |

모든 작업을 Deep으로 처리하면 비효율적이므로, 초기에는 사용자가 명시적으로 모드를 선택한다.

## 아키텍처 방향

```
VS Code Extension
        │
        ▼
   Core Engine
        │
 ┌──────┼──────┐
 ▼      ▼      ▼
Model  Context Workflow
Registry Store  Engine
        │
        ▼
Provider Adapters
```

VS Code는 제품 자체가 아니라 첫 번째 client다. 장기적으로 CLI / Web / Desktop 등으로 확장 가능하도록
Core는 처음부터 VS Code와 분리해서 설계한다.

## 사용 방식 (BYOK)

사용자가 자신의 API 키로 각 provider와 직접 연동하는 방식(BYOK)이다. 이 프로젝트는 AI를
재판매하거나 프록시하는 서비스가 아니라, provider들 사이를 조율하는 **소프트웨어/클라이언트**다.

## MVP 범위 (초안)

- VS Code Extension
- BYOK
- Provider 2~3개
- Dynamic Model Discovery
- Shared Context
- Generate / Review / Revise workflow
- Human Approval

## 확장 가능성

```
개인 개발 도구
   → Multi-LLM Development Workspace
   → LLM Collaboration Workspace
   → General LLM Integrated Workspace
   → IT / DX / 제조 / 연구 / 기획 등 B2B 적용
```

산업별로 UI와 connector만 달라지고 Core orchestration은 공유 가능하다는 가설 하에 설계한다.
단, 실제 사업화 가능성은 별도의 고객 검증이 필요하다.

## 현재 상태

기획 초안 단계. Problem Definition, MVP 범위, Core Engine 아키텍처(Provider Adapter, Model Registry),
Shared Project Context 스키마를 순차적으로 구체화하는 중.

---

*이 README는 초안이며, 프로젝트 진행에 따라 계속 갱신된다.*
