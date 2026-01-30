# Pal: Personal Agent that Learns

## メタデータ
- **調査日**: 2026-01-29
- **ソース**:
  - <a href="https://x.com/ashpreetbedi/status/2016702682925334818" target="_blank">X: @ashpreetbedi - Building Pal</a>
  - <a href="https://github.com/agno-agi/agno" target="_blank">GitHub: agno-agi/agno</a>
  - <a href="https://github.com/agno-agi/agentos-railway-template" target="_blank">GitHub: agno-agi/agentos-railway-template</a>
- **タグ**: #ai-agent #personal-ai #second-brain #duckdb #groq #ollama #agno

---

## ソース元の原文（@ashpreetbedi）

### 課題認識: 散在する個人情報

> 私たちの生活は断片化している。メモはApple Notes、ブックマークはブラウザ、連絡先は複数のアプリ、会議はカレンダー、タスクはTodoアプリ...これらの情報はバラバラに存在し、必要な時に見つけられない。

**現状の問題点**:
- 情報が複数のプラットフォームに散在
- 「あのメモどこだっけ？」「あの人の名前なんだっけ？」
- コンテキストスイッチのコストが高い
- 知識が蓄積されず、毎回ゼロから検索

### 解決策: Pal - 学習する個人エージェント

Palは「**第二の脳 (Second Brain)**」として機能する個人用AIエージェント。

**コンセプト**:
- すべての情報を一箇所に集約
- 使えば使うほど賢くなる（学習機能）
- 自然言語で何でも質問できる
- プライバシーを重視（ローカル運用可能）

---

## 詳細

### アーキテクチャ

```
┌──────────────────────────────────────────────────────────┐
│                      Pal Agent                            │
├──────────────────────────────────────────────────────────┤
│  Framework: Agno (agno-agi/agno)                         │
│  LLM: OpenAI GPT-5.2 / Groq llama-3.3-70b (無料代替)      │
│  Embeddings: OpenAI / Ollama nomic-embed-text (無料代替)  │
│  Web Search: Exa AI MCP Server                           │
├──────────────────────────────────────────────────────────┤
│                    Storage Layer                          │
│  ┌─────────────────────┐  ┌─────────────────────┐        │
│  │   DuckDB            │  │   Learning System   │        │
│  │   (User Data)       │  │   (Knowledge        │        │
│  │   - Notes           │  │    Patterns)        │        │
│  │   - Contacts        │  │   - Insights        │        │
│  │   - Bookmarks       │  │   - Connections     │        │
│  │   - Memories        │  │   - Preferences     │        │
│  └─────────────────────┘  └─────────────────────┘        │
└──────────────────────────────────────────────────────────┘
```

### なぜ2層構造か？

| 層 | 役割 | データ例 |
|----|------|----------|
| **DuckDB** | ユーザーの生データ保存 | メモ、連絡先、ブックマーク |
| **Learning System** | パターン・関係性の学習 | 「AさんとBさんはこのプロジェクトで関連」 |

この分離により：
- 生データはそのまま保持（検索可能）
- 学習済みの知識は別管理（高速な推論）
- データ削除してもパターンは残せる

### Agnoフレームワーク

Palは **Agno** フレームワークをベースに構築されている。

**Agnoの特徴**:
- Pythonベースのエージェント構築フレームワーク
- MCP (Model Context Protocol) サポート
- 複数のLLMプロバイダー対応
- ツール・メモリ・知識ベースの統合

```python
# Agnoでのエージェント定義例
from agno import Agent, DuckDB, Exa

agent = Agent(
    model="gpt-5.2",  # or "groq/llama-3.3-70b-versatile"
    storage=DuckDB(path="./pal.db"),
    tools=[Exa()],  # Web検索ツール
)
```

### エージェント構成ファイル

```
pal/
├── agents/
│   ├── __init__.py
│   ├── pal.py              # (5.9KB) コアエージェント
│   ├── knowledge_agent.py  # (2.3KB) 知識管理
│   └── mcp_agent.py        # (1.7KB) MCP統合
├── example.env             # 環境変数テンプレート
└── docker-compose.yml      # ローカル起動用
```

### 環境変数設定

```bash
# 必須
OPENAI_API_KEY=sk-...

# オプション（代替プロバイダー）
ANTHROPIC_API_KEY=...
GOOGLE_API_KEY=...
GROQ_API_KEY=...

# Web検索
EXA_API_KEY=...

# データベース（デフォルトで動作）
DB_USER=ai
DB_PASS=ai
DB_DATABASE=ai
```

---

## 無料運用構成

オリジナルはOpenAI GPT-5.2を使用するが、以下で**完全無料運用**が可能：

| コンポーネント | 有料版 | 無料代替 |
|---------------|--------|----------|
| LLM | OpenAI GPT-5.2 | Groq llama-3.3-70b-versatile |
| Embeddings | OpenAI embeddings | Ollama nomic-embed-text |
| Web検索 | Exa API (有料) | Exa無料枠 or 無効化 |

```python
# 無料構成でのエージェント定義
from agno import Agent, DuckDB

agent = Agent(
    model="groq/llama-3.3-70b-versatile",
    embedder="ollama/nomic-embed-text",
    storage=DuckDB(path="./pal.db"),
)
```

---

## デプロイ方法

### 1. ローカル（Docker）

```bash
git clone https://github.com/agno-agi/agno
cd agno/cookbook/pal

# 環境変数設定
cp example.env .env
# .envを編集してAPIキーを設定

# 起動
docker-compose up -d
```

### 2. クラウド（Railway）

1. <a href="https://github.com/agno-agi/agentos-railway-template" target="_blank">Railway Template</a> にアクセス
2. "Deploy on Railway" ボタンをクリック
3. 環境変数を設定
4. 自動デプロイ完了

---

## 将来の統合予定

作者が計画している統合先：
- **Slack** - チャット履歴・ファイル
- **Gmail** - メール・連絡先
- **Calendar** - 予定・会議
- **Linear** - タスク・プロジェクト
- **WhatsApp** - メッセージ履歴

---

## キーインサイト

1. **「第二の脳」コンセプト** - 散在する情報を統合し、必要な時にすぐアクセス
2. **2層アーキテクチャ** - DuckDB（生データ）+ Learning System（パターン）で柔軟性と効率を両立
3. **学習する設計** - 使うほど賢くなり、ユーザーの習慣・好みを理解
4. **プライバシー重視** - ローカル運用可能、データは自分で管理
5. **無料運用可能** - Groq + Ollama で API コストゼロ
6. **Agnoフレームワーク** - MCPサポートで拡張性が高い

---

## 関連
- [[Groq Free Tier]]
- [[Ollama Local LLM]]
- [[DuckDB]]
- [[MCP Model Context Protocol]]
- [[Agno Framework]]
- [[Exa AI Search]]
