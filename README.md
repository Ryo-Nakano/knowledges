# knowledges

Antigravity / Gemini CLI のカスタムスキル・サブエージェント・ドキュメントを集約したナレッジリポジトリ。

---

## 📁 ディレクトリ構成

```
knowledges/
├── .agent/
│   └── skills/               # Antigravity に認識されるスキル（プロジェクトスコープ）
│       ├── qiita-writer/
│       └── skill-creator/
├── agents/                   # Gemini CLI サブエージェント定義
│   └── code-reviewer.md
├── docs/                     # このリポジトリに関するドキュメント
│   ├── qiita_subagent_creator_draft.md
│   └── subagent_creator_reference.md
├── documents/                # Antigravity / Gemini CLI の参考資料
│   ├── antigravity_skills_guide.md
│   ├── gof_design_patterns_complete_guide.md
│   └── readable_code_complete_guide.md
├── skills/                   # Gemini CLI 向けスキル定義
│   ├── qiita-writer/
│   ├── readable-code-checker/
│   ├── skill-creator/
│   ├── spec-writer/
│   └── subagent-creator/
└── system_instructions/      # AI モデル向けシステムプロンプト集
```

---

## 🛠️ スキル一覧

スキルは `SKILL.md` を持つディレクトリとして定義される。エージェントは `description` フィールドを元に、必要なスキルを自動的に判断して呼び出す。

> **配置場所について**  
> `skills/` 以下は Gemini CLI 向け。Antigravity から使うには `.agent/skills/` への配置が必要。  
> 詳細は [`documents/antigravity_skills_guide.md`](documents/antigravity_skills_guide.md) を参照。

### [`subagent-creator`](skills/subagent-creator/SKILL.md)

Gemini CLI のカスタムサブエージェントの作成・設定・管理を支援する。

- 環境確認（`settings.json` の `enableAgents` 設定）
- 要件ヒアリング（目的・ツール・配置場所）
- サブエージェント定義ファイル（`.md`）の生成
- 動作確認手順の案内・トラブルシューティング

### [`qiita-writer`](skills/qiita-writer/SKILL.md)

Qiita でいいね・ストックを獲得できる技術記事の執筆を支援する。

- ヒアリング → 企画書作成 → 合意形成 → 本文執筆の4フェーズで進行
- タイトルの付け方・文体・視覚的要素のガイドライン内蔵
- 投稿前チェックリストと投稿タイミング推奨付き

### [`skill-creator`](skills/skill-creator/SKILL.md)

Antigravity のカスタムスキル（`SKILL.md`）の新規作成を支援する。

- キックオフ → ヒアリング → ドラフト生成 → ファイル作成の4フェーズで進行
- スキル名・配置場所・ワークフロー粒度を提案型インタビューで決定
- 生成した `SKILL.md` をそのままファイルに書き出す

### [`readable-code-checker`](skills/readable-code-checker/SKILL.md)

『リーダブルコード』の原則に従ってコードをレビューし、リファクタリング案を提案する。

- [`documents/readable_code_complete_guide.md`](documents/readable_code_complete_guide.md) をガイドラインとして参照
- 指摘箇所・理由・Before/After 形式で改善案を提示
- ユーザー許可のもとコードへの直接適用も可能

### [`spec-writer`](skills/spec-writer/SKILL.md)

新機能追加やバグ修正に際し、仕様書を作成する。

- 「推測しない・忖度しない」を原則としたヒアリング
- プロジェクトの `specs/_template.md` に沿った仕様書を生成
- `specs/` ディレクトリへのファイル出力まで対応

---

## 🤖 サブエージェント一覧

`agents/` ディレクトリに配置された Gemini CLI 向けサブエージェント定義。

### [`code-reviewer`](agents/code-reviewer.md)

『リーダブルコード』の原則に基づいてコードを自律的にレビューし、レポートを出力する。

- `readable-code-checker` スキルをアクティブ化してガイドラインを取得
- ファイル・ディレクトリを走査し、改善点を発見次第レポートを出力
- レビューと提案のみ担当（ファイル変更は行わない）

---

## 📄 ドキュメント / 参考資料

| ファイル | 内容 |
|---|---|
| [`documents/antigravity_skills_guide.md`](documents/antigravity_skills_guide.md) | Antigravity Skills の仕様・配置・ベストプラクティスまとめ |
| [`documents/gof_design_patterns_complete_guide.md`](documents/gof_design_patterns_complete_guide.md) | GoF デザインパターン完全ガイド |
| [`documents/readable_code_complete_guide.md`](documents/readable_code_complete_guide.md) | 『リーダブルコード』全観点まとめ |
| [`docs/subagent_creator_reference.md`](docs/subagent_creator_reference.md) | `subagent-creator` スキルのリファレンスドキュメント |
| [`docs/qiita_subagent_creator_draft.md`](docs/qiita_subagent_creator_draft.md) | `subagent-creator` に関する Qiita 記事ドラフト |

---

## 🚀 クイックスタート

### Antigravity でスキルを使う

このリポジトリをワークスペースとして開いた状態であれば、`.agent/skills/` 配下のスキル（`qiita-writer`, `skill-creator`）はすでに有効になっている。

```
# 例：Qiita 記事を書きたい場合
「Qiita記事を書きたい」と話しかける

# 例：新しいスキルを作りたい場合
「スキルを作りたい」と話しかける
```

### Gemini CLI でスキルを使う

`skills/` 以下のスキルを Gemini CLI から使うには、対象スキルを以下へコピーする。

```bash
# プロジェクトスコープ（このリポジトリ内のみ）
cp -r skills/<スキル名>/ .agent/skills/

# グローバルスコープ（全プロジェクトで使用）
cp -r skills/<スキル名>/ ~/.gemini/antigravity/skills/
```
