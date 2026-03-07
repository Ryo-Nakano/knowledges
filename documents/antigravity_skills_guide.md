# Antigravity Skills ガイド

> **調査日：** 2026-03-06  
> **情報源：** [antigravity.google/docs/skills](https://antigravity.google/docs/skills)（公式ドキュメント）

---

## 概要

**Skills** は Antigravity エージェントの能力を拡張するためのモジュール型パッケージ。特定のタスクや専門知識をエージェントに提供し、複雑なプロセスを標準化された手順でガイドする。

エージェントはスキルを**必要なときだけ**読み込む（progressive disclosure）ため、コンテキストウィンドウを効率的に使える。

---

## スキルが配置できる場所

| スコープ | パス | 使い分け |
|---|---|---|
| **ワークスペース固有** | `<workspace-root>/.agent/skills/<skill-name>/` | 特定プロジェクト内でのみ有効 |
| **グローバル** | `~/.gemini/antigravity/skills/<skill-name>/` | すべてのプロジェクトで有効 |

> **優先順位：** 同名スキルが両方に存在する場合、ワークスペース側が優先される。

### 配置例：グローバルスキル

```
~/.gemini/antigravity/skills/auto-japanese-committer/SKILL.md
~/.gemini/antigravity/skills/readable-code-checker/SKILL.md
~/.gemini/antigravity/skills/spec-writer/SKILL.md
~/.gemini/antigravity/skills/subagent-creator/SKILL.md
```

### 配置例：ワークスペーススキル

```
<project-root>/.agent/skills/knowledge-interviewer/SKILL.md
<project-root>/.agent/skills/spec-driven-task-assistant/SKILL.md
<project-root>/.agent/skills/task-execute-supporter/SKILL.md
```

### ❌ 認識されない場所

上記2つ以外のパス（例：`knowledges/skills/` など）に配置してもAntigravityからは**認識されない**。

---

## ディレクトリ構成

```
.agent/skills/
└── my-skill/              ← スキル名のフォルダ
    ├── SKILL.md           ← 必須（メイン指示ファイル）
    ├── scripts/           ← 任意（ヘルパースクリプト）
    ├── examples/          ← 任意（使用例・参照実装）
    └── resources/         ← 任意（テンプレート・アセット）
```

---

## SKILL.md のフォーマット

### フロントマター（YAML）

公式仕様で定義されているフィールドは **2つだけ**：

| フィールド | 必須 | 説明 |
|---|---|---|
| `name` | **任意** | スキルの一意識別子。小文字・ハイフン区切り。省略するとフォルダ名が使われる |
| `description` | **必須** | スキルが何をするか・いつ使うかの説明。エージェントがルーティング判断に使う |

> **注意：** `triggers`、`version`、`author` などの追加フィールドは公式には定義されていない。

### 完全なファイル例

```markdown
---
name: code-review
description: Reviews code changes for bugs, style issues, and best practices. Use when reviewing PRs or checking code quality.
---

# Code Review Skill

When reviewing code, follow these steps:

1. **Analyze the context**: Understand the purpose of the changes.
2. **Check for bugs**: Look for logical errors or edge cases.
3. **Style and best practices**: Ensure the code follows the project's style guide.
4. **Security**: Check for common vulnerabilities.

## How to provide feedback
Always be constructive and provide specific examples for any suggested changes.
```

### 本文（Markdown）

- フロントマターの後は**標準Markdown**で記述する
- エージェントへの詳細な指示、手順、コンベンションを書く
- `scripts/` などのサポートファイルは相対パスで参照できる

---

## エージェントがスキルを使う仕組み（Progressive Disclosure）

```
会話開始
    ↓
1. Discovery（発見）
   全スキルの name + description を読む
    ↓
2. Activation（活性化）
   タスクに関連するスキルの SKILL.md 全文を読む
    ↓
3. Execution（実行）
   SKILL.md の指示に従ってタスクを実行する
```

- ユーザーが明示的に「このスキルを使って」と言わなくても、`description` の内容からエージェントが自動的に判断して呼び出す
- スキル名を直接言及することで強制的に呼び出すこともできる

---

## ベストプラクティス

| 項目 | 推奨事項 |
|---|---|
| **スコープ** | 1スキル = 1タスク。複数の責務を1つにまとめない |
| **description の書き方** | 三人称で、エージェントが関連性を判断できるキーワードを含める |
| **スクリプト** | ブラックボックスとして扱い、`--help` で使い方をエージェントが確認できるようにする |
| **複雑なタスク** | 意思決定ツリーやチェックリストをMarkdown本文に含める |

---

## 関連リンク

- [公式ドキュメント: Skills](https://antigravity.google/docs/skills)
