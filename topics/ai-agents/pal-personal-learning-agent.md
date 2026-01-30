# Pal: Personal Agent that Learns

## メタデータ
- **調査日**: 2026-01-29
- **ソース**:
  - https://x.com/ashpreetbedi/status/2016702682925334818
  - https://github.com/agno-agi/agno
  - https://github.com/agno-agi/agentos-railway-template
- **タグ**: #ai-agent #personal-ai #second-brain #duckdb #groq #ollama

## 概要
Palは個人用AIエージェントシステムで、散在する情報を統合し、ユーザーのために学習・記憶する「第二の脳」として機能する。DuckDBでデータを保存し、別の学習システムで知識パターンを管理する二層構造を持つ。

## 詳細

### アーキテクチャ
```
┌─────────────────────────────────────┐
│           Pal Agent                 │
├─────────────────────────────────────┤
│  LLM: Groq (llama-3.3-70b)          │
│  Embeddings: Ollama (nomic-embed)   │
│  Web Search: Exa API                │
├─────────────────────────────────────┤
│  Data Storage: DuckDB (user data)   │
│  Learning System: 知識パターン管理    │
└─────────────────────────────────────┘
```

### エージェント構成
- **pal.py** (5.9KB) - コアエージェントモジュール
- **knowledge_agent.py** (2.3KB) - 知識管理機能
- **mcp_agent.py** (1.7KB) - MCP（Model Context Protocol）統合

### 無料運用構成
オリジナルはOpenAI GPT-5.2を使用するが、以下で無料運用可能：
- **LLM**: Groq llama-3.3-70b-versatile（無料枠）
- **Embeddings**: Ollama nomic-embed-text（ローカル）
- **Web検索**: Exa API

### デプロイ方法
1. **ローカル（Docker）**
   - Docker Compose で起動
   - DB: user=ai, pass=ai, database=ai（デフォルト）

2. **クラウド（Railway）**
   - agno-agi/agentos-railway-template を使用
   - ワンクリックデプロイ可能

### 将来の統合予定
- Slack
- Gmail
- Calendar
- Linear
- WhatsApp

## キーインサイト
- 散在する情報（各種アプリ、SNS、メモ）を一箇所に統合する「第二の脳」コンセプト
- DuckDB + 学習システムの二層アーキテクチャで、データと知識パターンを分離
- Groq + Ollama の組み合わせで完全無料運用が可能
- agno-agi フレームワークをベースに構築

## 関連
- [[Groq Free Tier]]
- [[Ollama Local LLM]]
- [[DuckDB]]
- [[MCP Model Context Protocol]]
