# Claude Codeの純正スケジュール機能（Desktop Local / Cloud Routines）は現状どちらも既知バグを抱えている

## 結論

2026年9月時点、Claude Code純正のスケジュール実行機能（Desktop appの「Scheduled tasks（Local）」と、CLI/Web UI/Desktop appから作成できる「Cloud Routines」）は、経路によって異なる既知バグ・制約を抱えており、無条件に信頼できる状態ではない。特に**ローカルの作業ディレクトリ（未コミットの変更を含む）に対して無人で定期実行したいユースケースは、現状どちらの経路でも実現できない**。確実に動かしたい場合はOSレベルのcron（Windowsならタスクスケジューラ）から`claude -p`を直接叩く方式に頼るのが現実的。

## 経路ごとの整理

### ① Desktop app の Local scheduled tasks（ローカル実行）

- PCが起動していてアプリが開いている間のみ動作する、という仕様上の制約に加え、実行時に**必ず失敗する既知バグ**が報告されている
- [anthropics/claude-code#47899](https://github.com/anthropics/claude-code/issues/47899) — タスクのディスパッチ（キック）は成功するがエージェント実行が始まらず、セッションが「Running」のままハングする。一度発生すると以降の全実行が「Skipped」扱いになる恒久障害。エラーメッセージ・出力ファイルなし。環境: Windows 11 / Claude Code Desktop / Cowork mode
- [anthropics/claude-code#48691](https://github.com/anthropics/claude-code/issues/48691) — Coworkクラウドサンドボックスのプロビジョニング失敗（`useradd: cannot create directory /sessions/...`）が原因で実行不能になるケース
- [anthropics/claude-code#56480](https://github.com/anthropics/claude-code/issues/56480) — 「Run now」が動作しない、として別途報告されている類似issue（#47899と同一バグの別報告である可能性があるが未確認。詳細は[claude_code_headless_remote_mcp_limitation.md](claude_code_headless_remote_mcp_limitation.md)も参照）
- ローカルファイル・ローカルMCPサーバーへのアクセスは可能（公式ドキュメントにも「ローカルファイル・ツールへのアクセスが必要ならDesktop tasksを使え」と明記）だが、上記バグにより実行自体が成立しない

### ② Android app

- スケジュール機能自体が非対応。公式ドキュメントのプラットフォーム対応表でMobileは該当機能すべて「❌」

### ③ CLI/Web UI経由の Cloud Routines（クラウド実行）

- PCがオフでも動作する（最短実行間隔1時間）が、**毎回GitHubリポジトリを新規クローンして実行する完全に隔離されたクラウドサンドボックス**であり、ローカルの作業ディレクトリ・未コミットの変更・ローカルファイルには一切アクセスできない
- 使えるツールもclaude.aiアカウントに接続済みのMCPコネクタ（Gmail/Slack等）のみ。ローカルMCPサーバーは`.mcp.json`にコミットしない限り使えない
- 実行そのものは（Desktop Localのような「ディスパッチしても起動しない」総倒れ的な障害ではなく）ケースバイケースで失敗する不具合が報告されている：
  - [anthropics/claude-code#52533](https://github.com/anthropics/claude-code/issues/52533) / [#50955](https://github.com/anthropics/claude-code/issues/50955) — `/schedule`実行時に "remote claude.ai accountに接続できない" 認証エラーで失敗
  - [anthropics/claude-code#29022](https://github.com/anthropics/claude-code/issues/29022) — `create_scheduled_task` MCPツールがセッションに正しく注入されない
  - [anthropics/claude-code#36327](https://github.com/anthropics/claude-code/issues/36327) — スケジュール実行時にMCPサーバー/コネクタが使えない

## 使い分けの判断基準

| ユースケース | 使える経路 |
|---|---|
| GitHub上にpush済みの内容 + claude.ai接続済みコネクタ（Gmail/Slack等）で完結する自動化 | Cloud Routines（ただし②の不具合群に注意） |
| ローカルの作業ディレクトリ・未コミット状態に対する定期自動化 | Claude Code純正機能では**現状実現不可**。OSレベルのcron/タスクスケジューラから`claude -p`を直接叩く方式で代替する |

## 代替アプローチ（推奨）

OSレベルのcron（Windowsならタスクスケジューラ）から`claude -p "<プロンプト>"`を直接叩く。Claude Code純正のスケジュール機能に依存しないため、上記の既知バグ群の影響を受けない。ただしこの方式には別の制約（`claude -p`のヘッドレス実行ではGmail等アカウントレベルのリモートMCPコネクタが引き継がれない）があるため、[claude_code_headless_remote_mcp_limitation.md](claude_code_headless_remote_mcp_limitation.md)も合わせて参照すること。

## 発見の経緯

ユーザーがClaude Desktop app / Android appからClaude Codeを利用する中で「スケジュールタスクが動かない」と相談したのを起点に、claude-code-guideサブエージェントで公式ドキュメントとGitHub issueを調査して判明。関連する実装事例は`tasks/doing/20260902-gmail-action-mail-triage`のWORKLOG.md（2026-09-02〜03）を参照。
