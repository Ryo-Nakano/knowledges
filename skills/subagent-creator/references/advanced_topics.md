# 発展的トピック

サブエージェントの基本的な作成方法は `SKILL.md` を参照。このドキュメントでは、より高度な機能について解説する。

> **Note:** この情報は **2026年3月時点** の仕様に基づく。いずれも実験的機能であり、今後変更される可能性がある。

---

## 1. Hooks によるセキュリティ強化

サブエージェントは YOLO モードで動作するため、特に `run_shell_command` や `write_file` を含む場合にセキュリティリスクがある。**Hooks** を使うと、ツール実行の前後にカスタムスクリプトを挟んでリスクを軽減できる。

### 主なフックイベント

| イベント | タイミング | 用途 |
|---|---|---|
| `BeforeTool` | ツール実行前 | 引数の検証、危険な操作のブロック |
| `AfterTool` | ツール実行後 | 結果の検証、テストの実行、ログ記録 |
| `BeforeAgent` | エージェント起動前 | コンテキストの注入 |
| `AfterAgent` | エージェント完了後 | 結果の後処理 |

### 設定例

`~/.gemini/settings.json` に以下のようにフックを設定する：

```json
{
  "hooks": {
    "BeforeTool": [
      {
        "command": "/path/to/validate_command.sh",
        "tools": ["run_shell_command"]
      }
    ]
  }
}
```

この例では、`run_shell_command` が実行される前に `validate_command.sh` スクリプトが呼ばれ、コマンド内容を検証・ブロックできる。

### 活用パターン

- **コマンドのホワイトリスト制御：** `BeforeTool` フックで許可されたコマンドのみ実行を許可
- **機密情報の検出：** `AfterTool` フックで出力にシークレットが含まれていないか検査
- **操作ログの記録：** フックでツール実行の履歴を外部ファイルに記録

> ⚠️ フックはユーザー権限で任意のコードを実行するため、信頼できるスクリプトのみ設定すること。プロジェクトレベルのフックは、Gemini CLI がフィンガープリントを検出して変更時に警告を表示する。

---

## 2. MCP サーバー連携

MCP（Model Context Protocol）サーバーを使うと、Gemini CLI にカスタムツールを追加できる。サブエージェントの `tools` フィールドでは、MCP サーバー経由で追加されたツール名も指定可能。

### MCP サーバーの設定

`~/.gemini/settings.json`（グローバル）または `.gemini/settings.json`（プロジェクト）で設定する：

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/mcp-server/index.js"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

### サブエージェントでの利用

MCP サーバーが提供するツール名を、サブエージェントの `tools` フィールドに追加する：

```yaml
---
name: my-agent
description: ...
kind: local
tools:
  - read_file
  - grep_search
  - my_custom_tool_from_mcp
---
```

> 💡 MCP サーバーのツール名は、MCP サーバー側で定義される。利用可能なツール名は `/tools` コマンドで確認できる。

---

## 3. リモートエージェント（A2A プロトコル）

`kind: remote` を使うと、A2A（Agent-to-Agent）プロトコルに対応した外部エージェントサービスをサブエージェントとして利用できる。

### テンプレート

```yaml
---
name: remote-agent-example
description: 外部サービスXの処理を委譲する専門エージェント
kind: remote
agent_card_url: https://example.com/.well-known/agent.json
---
```

### ローカルエージェントとの違い

| 項目 | ローカル (`kind: local`) | リモート (`kind: remote`) |
|---|---|---|
| 定義場所 | `.gemini/agents/*.md` | 同左 |
| 処理の実行 | ローカル環境 | 外部サービス |
| `tools` フィールド | 必要 | 不要（外部サービスが管理） |
| `agent_card_url` | 不要 | **必須** |
| システムプロンプト | ファイル本文で定義 | 外部サービスが管理 |

> ⚠️ リモートエージェントは外部サービスに依存するため、ネットワーク障害やサービスの仕様変更の影響を受ける。信頼できるサービスのみ利用すること。
