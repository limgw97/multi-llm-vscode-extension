[한국어](README.md) | [English](README.en.md) | [日本語](README.ja.md)

# LLM Orchestrator (仮称)

> 複数のLLMを一つのワークフローの中で協業させ、その間の情報伝達とコンテキスト管理をシステムが担う開発環境。

## なぜ作るのか

複数のLLMを開発過程で同時に使っていると、結局人間がこうした作業をすることになる。

```
LLM Aに質問 → 結果をコピー → LLM Bに貼り付け → Bのレビューをコピー
→ 再びAに貼り付け → 人間が結果を比較 → 必要な内容を整理
→ 再び別のAIに伝える …
```

この過程で、コンテキストの喪失、情報の抜け漏れ、古いバージョンの伝達、コピー&ペーストのミス、
人間が間に入って解釈することで生じる歪み、不必要な調整コストが繰り返し発生する。
AI自体のエクスポート機能も、元のコンテキストを100%保存する保証はない。

根底にある問いはシンプルだ。

> **「なぜ自分がAI同士のメッセンジャー役をやらなければならないのか?」**

このプロジェクトはこの問いから出発する。人間がAIの間を行き来して情報を伝えるのではなく、
**AI同士が必要な情報を直接やり取りして協業し、人間は目標設定・判断・選択・承認だけを担当する**
ようにする。

## 何を作るのか

単純なMulti-LLM Chat AppでもAIコーディングツールでもない。目指すのは:

> 複数のLLMを一つのワークフローの中で協業させ、その間のコンテキストと情報伝達を自動化する
> **LLM orchestration環境**。

開発を最初のユースケースとするが、それに限定しない。同じorchestration構造は、企画/DX、研究、
一般事務など「複数のソースを集めて分析・検証し、人間が判断する」あらゆるワークフローに適用可能。

## コア原則

| # | 原則 |
|---|---|
| 1 | Multi-LLM自体が目的ではない — 核心はAI間coordinationの自動化 |
| 2 | 人間がAI間で情報を伝達する過程を取り除く |
| 3 | チャット履歴をそのまま渡す代わりに**Structured Shared Context**を使う |
| 4 | AIは互いの結果を直接レビューし修正できるべき |
| 5 | Cross-checkは単純な多数決ではなく**検証**である |
| 6 | Fact-checkでは結論ではなく**根拠そのもの**をcross-checkする |
| 7 | WeightはTruth ScoreではなくPriority/Influenceである (`Weight ≠ Truth`) |
| 8 | モデルの選択権は常にユーザーにある |
| 9 | ProviderとModelは分離して扱う |
| 10 | モデル名をcoreにハードコーディングしない — **Dynamic Model Discovery** |
| 11 | Token/Costはユーザーが直接制御できる要素 |
| 12 | Workflowの強度はQuick / Review / Debate / Deepで調整可能 |
| 13 | **人間は常に最終意思決定者である** |
| 14 | 初期の入口はVS Code Extension、Coreはそれと分離して設計する |
| 15 | 長期的にはstandaloneのLLM workspaceへ拡張できるようにする |

## 人間とAIの役割分担

**AIが担当**: 資料調査・要件分析・企画・システム設計・実装・コードレビュー・反論・比較・検証・修正案の提示・事実確認

**開発者が担当**: 目標設定・重要な要件の決定・トレードオフの判断・結果の比較・最終選択・承認

```
AIの意見 → 開発者のレビュー → 選択 → 承認 → 適用
```

AIが開発者を代替する構造ではなく、複数のAIを使う上で必要な**中間の伝達作業を取り除く構造**である。

## 開発ワークフロー(例)

```
Idea → Requirements → Research → Design → Architecture
→ Implementation Plan → Code → Review → Revision → Verification
```

企画から開発・検証まで一つのLLM環境の中でつながることを目指す。例えば:

1. アイデアをざっくり説明
2. LLMたちが要件を具体化し、曖昧な部分を洗い出す
3. 関連資料を調査
4. 複数のLLMが設計案を作成 → 互いにレビュー → トレードオフを比較
5. 開発者が設計を選択 → Project Contextに保存
6. 実装計画 → コード生成 → 別のLLMがレビュー → 元のLLMがフィードバックを反映
7. テスト/検証 → 開発者の最終承認

## Cross-check: 多数決ではなく検証

複数モデルの結果を単に並べて見せるのではなく、同じissueをclusteringし、
モデルごとの賛成/反対とその根拠を一緒に表示する。

```
Issue #12 — service.py:47 — Potential None handling bug
  GPT     → Agree
  Claude  → Agree
  Gemini  → Disagree (根拠: ...)
```

2つが賛成し1つが反対しているからといって、自動的に多数派が正しいとは扱わない。
最終判断は根拠を見た人間が行う。

## Fact-check: 根拠をcross-checkする

モデルの結論ではなく**根拠(Source/Evidence)**を互いに検証させる。
システム内部では以下の4つを絶対に一つのスコアにまとめない。

- Priority(設定された優先度)
- Confidence(モデルが示した確信度)
- Evidence(実際の根拠)
- Consensus(モデル間の合意の有無)

## Provider / Model / Dynamic Discovery

```
Provider Adapter → Model Discovery → Model Registry → Workflow
```

- **Provider**(OpenAI、Anthropic、Google …)と**Model**(GPT、Claude Sonnet、Claude Opus、Gemini …)を分離して設計する。ユーザーが選択する単位はモデル。
- モデル名をアプリケーションにハードコーディングしない。Provider Adapterが利用可能なモデルを動的に取得し、Coreはgenericなmodel abstractionとして扱う。
- 新しいモデルが出てもcoreを修正せず、discoveryを通じて表示されることを目標とする。

## Shared Project Context

チャット履歴をそのまま渡す代わりに、構造化されたプロジェクトコンテキストを維持する。

```
Project Context
├── Requirements
├── Constraints
├── Architecture
├── Decisions        (根拠 / モデルごとの賛否 / 承認者を含む)
├── Research
├── Source Evidence
├── Current Code
├── Open Issues
└── Change History
```

この構造がそのまま「AIたちの共通コンテキスト」となり、何をなぜ決めたのか、誰が承認したのかを
追跡可能にする。

## Workflowの強度(コスト/レイテンシ制御)

| Mode | 説明 |
|---|---|
| ⚡ Quick | LLM1つ、速い応答、低コスト |
| 🔍 Review | Generate → LLM2〜3個でReview |
| ⚔️ Debate | 複数モデルが意見を提示 → 反論 → 再検討 |
| 🧠 Deep | Generate → Review → Debate → Revise → Re-review → Verify |

すべての作業をDeepで処理すると非効率なため、初期はユーザーが明示的にモードを選択する。

## アーキテクチャの方向性

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

VS Codeは製品そのものではなく最初のclientである。長期的にCLI / Web / Desktopなどへ拡張できるよう、
Coreは最初からVS Codeと分離して設計する。

## 利用方式(BYOK)

ユーザーが自分のAPIキーで各providerと直接連携する方式(BYOK)。このプロジェクトはAIを
再販売・プロキシするサービスではなく、provider間を調整する**ソフトウェア/クライアント**である。

## MVPスコープ(初稿)

- VS Code Extension
- BYOK
- Provider 2〜3個
- Dynamic Model Discovery
- Shared Context
- Generate / Review / Revise ワークフロー
- Human Approval

## 拡張可能性

```
個人開発ツール
   → Multi-LLM Development Workspace
   → LLM Collaboration Workspace
   → General LLM Integrated Workspace
   → IT / DX / 製造 / 研究 / 企画などB2B適用
```

業界ごとにUIとconnectorだけが変わり、Core orchestrationは共有できるという仮説のもとに設計する。
ただし実際の事業化可能性は別途、顧客検証が必要。

## 現在の状況

企画初稿段階。Problem Definition、MVPスコープ、Core Engineアーキテクチャ(Provider Adapter、
Model Registry)、Shared Project Contextスキーマを順次具体化中。

---

*このREADMEは初稿であり、プロジェクトの進行にあわせて更新され続ける。*
