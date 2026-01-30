# Pal: Personal Agent that Learns

## メタデータ
- **調査日**: 2026-01-29（深掘り: 2026-01-30）
- **ソース**:
  - <a href="https://x.com/ashpreetbedi/status/2016702682925334818" target="_blank">X: @ashpreetbedi - Building Pal</a>
  - <a href="https://github.com/agno-agi/agno" target="_blank">GitHub: agno-agi/agno</a>
  - <a href="https://docs.agno.com" target="_blank">Agno Documentation</a>
- **タグ**: #ai-agent #personal-ai #second-brain #learning-system #agno #memory-vs-learning

---

## TL;DR - これはRAGではない

> **「Memory is a noun. Learning is a verb.」**
> - メモリは静的：事実のデータベース、「あなたが言ったこと」を保存
> - ラーニングは動的：進化し、複利で成長し、「それが何を意味するか」を理解

**Palの核心**: 1000日目のエージェントは1日目より**根本的に優れている**

---

## 問題提起: ほとんどのエージェントはステートレス

### 現状のLLMエージェントの問題

```
┌─────────────────────────────────────────┐
│  従来のエージェント                      │
│                                         │
│  推論 → 応答 → 忘れる                   │
│                                         │
│  セッション履歴は役立つが、              │
│  会話が終われば消える                    │
│                                         │
│  1000日目のエージェントも               │
│  1日目と同じ能力                         │
└─────────────────────────────────────────┘
```

**作者 Ashpreet Bedi の言葉**:
> "Claude's memory feels magical. It's natural, contextual, never announces 'saving to memory'. It just knows you. But you can't build with it. Claude's memory is a consumer product feature. The API gives you nothing. If you want learning for your agents, you're on your own."

---

## RAG vs Learning: 本質的な違い

| 観点 | RAG (Retrieval) | Learning (Agno) |
|------|-----------------|-----------------|
| **性質** | 静的 | 動的 |
| **動作** | 検索して返す | 理解し、進化する |
| **データ** | ドキュメントをロード | インサイトを抽出 |
| **時間経過** | 変化なし | 複利で成長 |
| **ユーザー間** | 独立 | **集合知を共有** |

### RAGの限界
- ドキュメントをロードして検索できる**だけ**
- エージェントは何も**発見**していない
- エージェントは**賢くなっていない**

### Learningの違い
1. セッションを跨いでユーザーを記憶
2. 会話からインサイトを**抽出**
3. 自身の決定とフィードバックから**学習**
4. あるユーザーの知識が別のユーザーに**恩恵をもたらす**

---

## 知識の複利効果（Compound Knowledge）

### 具体例

```
セッション1 - エンジニアA（月曜日）:
┌────────────────────────────────────────┐
│ 「クラウドのEgressコストを削減したい」  │
└────────────────────────────────────────┘
        ↓
   エージェントがインサイトを抽出・保存
   「Egressコストはクラウド選定の重要考慮点」
        ↓
セッション2 - エンジニアB（1週間後、別ユーザー）:
┌────────────────────────────────────────┐
│ 「データパイプライン用のクラウドを選定中」│
│ 「考慮すべき点は？」                    │
└────────────────────────────────────────┘
        ↓
   エージェントが学習済みインサイトを自動提示
   「Egressコストも考慮してください」
        ↓
   ★ 明示的な引き継ぎなしで、Bが恩恵を受ける
```

**これがRAGと根本的に異なる点**:
- RAGは「誰かがドキュメントに書いた情報」を検索
- Learningは「対話から得られた暗黙知」を蓄積・共有

---

## 6種類のLearning Store（LearningMachine）

Agnoの学習システムは6つのストアを統合管理：

| Store | 内容 | スコープ | 用途 |
|-------|------|----------|------|
| **User Profile** | 名前、好み等の構造化データ | ユーザー毎 | パーソナライズ |
| **User Memory** | ユーザーについての非構造化観察 | ユーザー毎 | コンテキスト |
| **Session Context** | ゴール、計画、進捗、サマリー | セッション毎 | タスク継続性 |
| **Entity Memory** | 事実、イベント、関係性 | 設定可能 | ナレッジグラフ |
| **Learned Knowledge** | インサイト、パターン、ベストプラクティス | 設定可能 | **集合知** |
| **Decision Log** | 意思決定とその理由 | エージェント毎 | 監査・デバッグ |

### 学習モード

| モード | 動作 |
|--------|------|
| **ALWAYS** | 各応答後に自動抽出（デフォルト） |
| **AGENTIC** | エージェントがツールで保存判断 |
| **PROPOSE** | 提案→ユーザー承認→保存 |

---

## なぜDuckDBなのか

### 2つのストレージの役割

```
┌─────────────────────────────────────────────────────────┐
│                    Storage Layer                         │
├──────────────────────────┬──────────────────────────────┤
│       DuckDB             │      Learning System         │
│    (ユーザーデータ)       │      (システム知識)           │
├──────────────────────────┼──────────────────────────────┤
│ notes                    │ テーブルスキーマ              │
│ bookmarks                │ リサーチ結果                 │
│ people                   │ エラー修正パターン           │
│ meetings                 │ ユーザー間の共有インサイト    │
│ projects                 │ ベストプラクティス           │
└──────────────────────────┴──────────────────────────────┘
```

### DuckDB選定の理由

1. **動的テーブル作成** - 「Pal creates tables on the fly」
   - 事前定義なし、必要に応じてテーブルを生成
   - `notes`, `bookmarks`, `people` 等を動的に構築

2. **組み込み型** - セットアップ不要、単一ファイル

3. **SQL標準準拠** - LLMがSQL生成しやすい

4. **分析クエリに強い** - Parquet、CSV等を直接クエリ可能

5. **軽量** - Dockerコンテナ内でも動作

---

## Agnoフレームワークの差別化

### 他のエージェントフレームワークとの違い

| 観点 | 他フレームワーク | Agno |
|------|-----------------|------|
| 学習 | 自分で構築が必要 | **統合Learning System** |
| 本番運用 | 別途構築 | **AgentOS内蔵** |
| データ | ベンダー依存の場合あり | **完全なデータ主権** |
| MCP | 部分対応 | **フルサポート** |

### データ主権

> "Everything runs in your infrastructure. Your database. Your vector store. Your cloud. No vendor has access to your learnings."

- ホスト型メモリサービスではない
- SQLでクエリ可能、ダッシュボード構築可能
- いつでもエクスポート可能

### 技術スタック

| レイヤー | 機能 |
|----------|------|
| **Framework** | 学習、ツール、知識、ガードレール付きエージェント構築 |
| **Runtime (AgentOS)** | 本番環境での実行基盤（FastAPIベース） |
| **Control Plane** | モニタリング・管理UI |

---

## GPU Poor Learning

**ファインチューニングやRLHF不要**

- データベースとプロンプトエンジニアリングだけで実現
- GPU リソースなしで学習可能
- 推論コストのみで運用可能

```python
# 学習は推論時に行われる
agent = Agent(
    model="groq/llama-3.3-70b-versatile",  # 推論用モデル
    learning=LearningConfig(
        mode=LearningMode.ALWAYS,  # 毎回学習
        stores=[
            UserProfile(),
            UserMemory(),
            LearnedKnowledge(),  # ← ここに集合知が蓄積
        ]
    )
)
```

---

## 作者について

### Ashpreet Bedi
- **現職**: Agno創業者
- **前職**: Airbnb, Facebook (Meta)
- **X**: <a href="https://x.com/ashpreetbedi" target="_blank">@ashpreetbedi</a>（16K フォロワー）

### 開発動機（本人の言葉）

> "My information is scattered everywhere. Notes in text files. Bookmarks across three different browsers. People I meet (six or seven a day) living in my head until I forget them. Research I did last week that I cannot find anymore."

> "I wanted one place to dump things. Message it on Slack. WhatsApp it. Talk to it. Then have it remember, organize, and pull things back when I need them."

---

## キーインサイト

1. **Memory vs Learning** - 静的な記憶ではなく、動的に進化する学習
2. **複利効果** - ユーザーAの知識がユーザーBに恩恵をもたらす集合知
3. **6種類のStore** - 多層的な学習管理（Profile, Memory, Context, Entity, Knowledge, Decision）
4. **RAGの限界突破** - 検索ではなく理解、データではなくインサイト
5. **データ主権** - 自分のインフラで完結、ベンダーロックインなし
6. **GPU Poor** - ファインチューニング不要、推論のみで学習実現

---

## 無料運用構成

| コンポーネント | 有料版 | 無料代替 |
|---------------|--------|----------|
| LLM | OpenAI GPT-5.2 | Groq llama-3.3-70b-versatile |
| Embeddings | OpenAI | Ollama nomic-embed-text |
| Web検索 | Exa API | Exa無料枠 or 無効化 |

---

## 関連
- [[Agno Framework]]
- [[LearningMachine Architecture]]
- [[Memory vs Learning]]
- [[Compound Knowledge]]
- [[DuckDB]]
- [[MCP Model Context Protocol]]
