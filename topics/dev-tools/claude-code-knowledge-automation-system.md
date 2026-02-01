# Claude Code ナレッジ管理自動化システム実装ガイド

## メタデータ
- **調査日**: 2026-02-01
- **ソース**:
  - <a href="https://github.com/msy7822-ux/inmybrain" target="_blank">inmybrain repository</a>
  - Claude Code Skills & Hooks実装
- **タグ**: #claude-code #automation #knowledge-management #skills #hooks

## 概要

開発中に発見したナレッジを自動的にGitHubリポジトリ（inmybrain）に記録するシステム。手動記録用の `/kb` skill とセッション終了時の自動リマインダー（Stop hook）を組み合わせたハイブリッドアプローチ。

## アーキテクチャ

### Skill vs Hook の使い分け

| 機能 | Skill (`/kb`) | Hook (Stop) |
|------|---------------|-------------|
| **実行タイミング** | ユーザーが明示的に呼び出し | セッション終了時に自動実行 |
| **用途** | 重要なナレッジを即座に記録 | 記録漏れを防ぐリマインダー |
| **処理内容** | 完全な記録フロー（質問→保存→push） | リマインダー表示のみ |
| **柔軟性** | 高い（詳細な対話可能） | 低い（bashスクリプト） |

### システムフロー

```
開発中
  ↓
重要な発見
  ↓
ユーザーが `/kb` 実行 ──→ 即座に記録
  ↓
（または）
  ↓
セッション終了 ──→ Stop hook ──→ リマインダー表示 ──→ `/kb` 実行
```

## /kb Skill の実装

### ファイル構成

```
~/.claude/skills/knowledge-base/
└── SKILL.md
```

### SKILL.md の構造

```markdown
---
name: kb
description: Record development knowledge...
---

# Knowledge Base Recording Skill

## 使用タイミング
- リサーチで重要な発見があった時
- 実装で得たノウハウを記録したい時
...

## 実装手順
1. ユーザーからの情報収集
2. inmybrainリポジトリの準備
3. ナレッジファイルの作成
4. README.mdの更新
5. コミット＆プッシュ
6. 完了報告
```

### 実装のポイント

#### 1. 情報収集フェーズ

質問すべき項目:
- **タイトル**: ナレッジの簡潔な名前
- **カテゴリ**: 保存先ディレクトリ（既存 or 新規）
- **タグ**: 検索用キーワード（複数可）
- **ソース**: URL や参考資料（任意）

#### 2. リポジトリ準備

```bash
# リポジトリがなければクローン
if [ ! -d /tmp/inmybrain ]; then
  cd /tmp && git clone https://github.com/msy7822-ux/inmybrain.git
fi

# 最新の状態に更新
cd /tmp/inmybrain && git pull origin main
```

#### 3. ファイル作成

テンプレートに従って構造化されたマークダウンを生成:

```markdown
# {{タイトル}}

## メタデータ
- **調査日**: {{YYYY-MM-DD}}
- **ソース**: {{URL}}
- **タグ**: {{#tag1 #tag2}}

## 概要
{{1-2文での要約}}

## 詳細
{{本文}}

## キーインサイト
- {{ポイント1}}
- {{ポイント2}}

## 関連
- [[{{関連トピック}}]]
```

#### 4. README.md更新

新しいエントリをカテゴリ別に追加:

```markdown
### Dev Tools
- [Claude Code ナレッジ管理自動化システム](topics/dev-tools/claude-code-knowledge-automation-system.md) - skillとhookを組み合わせたナレッジ記録システム
```

#### 5. Git操作

```bash
cd /tmp/inmybrain
git add .
git commit -m "Add: {{title}}"
git push origin main
```

## Stop Hook の実装

### 設定ファイル: ~/.claude/settings.json

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\n# Knowledge base recording reminder\ninput=$(cat)\n\nif git rev-parse --git-dir > /dev/null 2>&1; then\n  # Check if there were significant changes\n  added_files=$(git diff --name-status HEAD 2>/dev/null | grep '^A' | wc -l | tr -d ' ')\n  modified_files=$(git diff --name-status HEAD 2>/dev/null | grep '^M' | wc -l | tr -d ' ')\n  total_changes=$((added_files + modified_files))\n  \n  if [ \"$total_changes\" -gt 3 ]; then\n    echo \"\" >&2\n    echo \"━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\" >&2\n    echo \"🧠 Knowledge Base Reminder\" >&2\n    echo \"━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\" >&2\n    echo \"📝 今回のセッションで重要な学びはありましたか？\" >&2\n    echo \"\" >&2\n    echo \"記録する場合: /kb\" >&2\n    echo \"スキップする場合: そのまま終了\" >&2\n    echo \"━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\" >&2\n  fi\nfi\n\necho \"$input\""
          }
        ]
      }
    ]
  }
}
```

### Hook の動作

1. **トリガー条件**: セッション終了時
2. **判定ロジック**: Git変更が3ファイル以上
3. **アクション**: リマインダーをstderrに出力
4. **ユーザー選択**: `/kb` 実行 or 無視

### 実装のポイント

#### なぜStop hookか？

- **PostToolUse**: ツール実行後に毎回発火 → ノイズが多い
- **PreToolUse**: ツール実行前に発火 → タイミングが早すぎる
- **Stop**: セッション終了時のみ → 最適なタイミング

#### 変更量の判定

```bash
total_changes=$((added_files + modified_files))

if [ "$total_changes" -gt 3 ]; then
  # リマインダー表示
fi
```

閾値を3ファイル以上に設定することで、小さな変更では通知されない。

## 使用フロー

### パターン1: 即座に記録

```
1. 開発中に重要な発見
2. `/kb` と入力
3. Claude が質問（タイトル、カテゴリ、タグ等）
4. 回答
5. 自動でinmybrainにpush
6. 完了通知
```

### パターン2: セッション終了時に記録

```
1. セッション終了
2. Stop hook が発火
3. リマインダー表示
   「📝 今回のセッションで重要な学びはありましたか？」
4. 必要なら `/kb` 実行
5. 不要ならそのまま終了
```

## キーインサイト

### 1. ハイブリッドアプローチの利点

- **手動記録（/kb）**: 重要な発見を逃さず即座に記録
- **自動リマインダー（Stop hook）**: 記録漏れを防ぐセーフティネット
- 両者の組み合わせで記録精度が向上

### 2. Skillの設計原則

**良いSkillの条件**:
- 明確な単一責任（ナレッジ記録のみ）
- ユーザーとの対話を含む（質問→回答→実行）
- 複雑なロジックが必要な処理
- 再利用可能な手順

**Hookの限界**:
- Bashスクリプトのみ（複雑な処理が困難）
- ユーザーとの対話不可
- 条件分岐や判定のみに適している

### 3. テンプレート駆動の重要性

ナレッジファイルを統一フォーマットで作成することで:
- 検索性が向上
- 後から見返しやすい
- 自動化しやすい

### 4. Git操作の自動化

手動でのcommit/pushは忘れがち。Skillに組み込むことで:
- 記録の確実性が向上
- ユーザーの負担軽減
- 一貫性のあるコミットメッセージ

## トラブルシューティング

### 問題1: リポジトリのクローン失敗

**原因**: GitHub認証エラー

**解決策**:
```bash
# SSH鍵の確認
ssh -T git@github.com

# HTTPSの場合はPersonal Access Token確認
git config --global credential.helper
```

### 問題2: プッシュ失敗（競合）

**原因**: リモートとローカルの差分

**解決策**:
```bash
cd /tmp/inmybrain
git pull --rebase origin main
git push origin main
```

### 問題3: Stop hookが発火しない

**原因**: 変更ファイル数が閾値未満

**確認方法**:
```bash
git diff --name-status HEAD | wc -l
```

**調整**: `~/.claude/settings.json` の閾値を変更

```bash
if [ "$total_changes" -gt 3 ]; then  # ← この数値を調整
```

### 問題4: Skillが認識されない

**原因**: スキルディレクトリの配置ミス

**確認方法**:
```bash
ls -la ~/.claude/skills/knowledge-base/
# SKILL.md が存在するか確認
```

**解決策**: 正しい場所に配置し、Claude Codeを再起動

## ベストプラクティス

### 1. カテゴリの命名規則

- **kebab-case**: `ai-agents`, `dev-tools`
- **単数形**: `ai-agent` ではなく `ai-agents`
- **具体的**: `tools` ではなく `dev-tools`

### 2. タグの付け方

- **技術名**: `#claude-code`, `#typescript`
- **概念**: `#automation`, `#knowledge-management`
- **3-5個程度**: 多すぎると検索性が低下

### 3. ファイル名の規則

- **kebab-case**: `claude-code-knowledge-automation-system.md`
- **説明的**: 内容が分かる名前
- **一意性**: 他のファイルと重複しない

### 4. README.mdの管理

- **カテゴリ別**: トピックをグループ化
- **簡潔な説明**: 1行で概要を記述
- **相対パス**: `topics/dev-tools/...` 形式

## 拡張アイデア

### 1. 自動タグ生成

Claude が内容を分析して適切なタグを提案:

```
User: /kb
Claude: タイトルと内容から以下のタグを提案します:
  #claude-code #automation #git
  追加・削除しますか？
```

### 2. 関連ナレッジの自動リンク

既存のナレッジを検索して関連リンクを自動追加:

```markdown
## 関連
- [[Claude Code Hooks リファレンス]]
- [[Git Automation Patterns]]
```

### 3. 週次レポート生成

Stop hookで1週間分のナレッジをまとめたレポートを生成:

```markdown
# 2026-02-01週 学習レポート

## 今週追加されたナレッジ
- Claude Code ナレッジ管理自動化システム
- ...

## カテゴリ別統計
- dev-tools: 3件
- ai-agents: 1件
```

### 4. Obsidian連携

inmybrainをObsidian vaultとして扱い、グラフビューで知識の関連性を可視化。

## 関連

- [[Git Automation Patterns]]
- [[Claude Code Skills Development Guide]]
- [[Knowledge Management Best Practices]]
