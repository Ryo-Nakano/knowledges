# Claude Codeの純正スケジュール機能の使い分け（Desktop Local / Cloud Routines / bridge環境Routine）

## 結論

**2026-10-01更新**: ローカルの作業ディレクトリ（未コミットの変更を含む）を、Gmail等のclaude.aiコネクタと組み合わせて無人で定期実行したい場合は、**Cloud Routineの実行環境にbridge環境（`claude rc` が動いているマシン）を指定する**方法で実現できる。実機で検証済み（詳細は後述の④）。

一方、Desktop appの「Scheduled tasks（Local）」は、2026-10-01時点でもこの環境（Desktop app on Mac＋workspaceはlima VM上）では起動しなかった。通常のCloud Routines（クラウド環境）は動くが、GitHub上の内容しか見えない。

（2026年9月時点の旧結論: Claude Code純正機能ではローカル向けの無人自動化は実現できず、OSレベルのcronから `claude -p` を叩くしかない、としていた。ただし `claude -p` ではGmail等のコネクタが使えない）

## 経路ごとの整理

### ① Desktop app の Local scheduled tasks（ローカル実行）

- PCが起動していてアプリが開いている間のみ動作する、という仕様上の制約に加え、実行時に**必ず失敗する既知バグ**が報告されている
- [anthropics/claude-code#47899](https://github.com/anthropics/claude-code/issues/47899) — タスクのディスパッチ（キック）は成功するがエージェント実行が始まらず、セッションが「Running」のままハングする。一度発生すると以降の全実行が「Skipped」扱いになる恒久障害。エラーメッセージ・出力ファイルなし。環境: Windows 11 / Claude Code Desktop / Cowork mode
- [anthropics/claude-code#48691](https://github.com/anthropics/claude-code/issues/48691) — Coworkクラウドサンドボックスのプロビジョニング失敗（`useradd: cannot create directory /sessions/...`）が原因で実行不能になるケース
- [anthropics/claude-code#56480](https://github.com/anthropics/claude-code/issues/56480) — 「Run now」が動作しない、として別途報告されている類似issue（#47899と同一バグの別報告である可能性があるが未確認。詳細は[claude_code_headless_remote_mcp_limitation.md](claude_code_headless_remote_mcp_limitation.md)も参照）
- ローカルファイル・ローカルMCPサーバーへのアクセスは可能（公式ドキュメントにも「ローカルファイル・ツールへのアクセスが必要ならDesktop tasksを使え」と明記）だが、上記バグにより実行自体が成立しない
- 2026-10-01の再検証: 上記の#47899等は「修正」ではなく、放置による自動クローズだった。9月にも未解決の新しい報告がある（[#95966](https://github.com/anthropics/claude-code/issues/95966) 予定時刻に何も起きない、[#93948](https://github.com/anthropics/claude-code/issues/93948) 実行が止まったまま復旧しない）。実機（Desktop app on Mac、workspaceはlima VM上）でも、自動実行・Run nowともセッションが作られず、「ルーティンを実行できませんでした」というトーストが出た。タスク定義はMac側（`/Users/.../.claude/scheduled-tasks/`）に保存されるため、作業フォルダがVM側にある構成に対応していない可能性もある

### ② Android app

- スケジュール機能自体が非対応。公式ドキュメントのプラットフォーム対応表でMobileは該当機能すべて「❌」

### ③ CLI/Web UI経由の Cloud Routines（クラウド実行）

- PCがオフでも動作する（最短実行間隔1時間）が、**毎回GitHubリポジトリを新規クローンして実行する完全に隔離されたクラウドサンドボックス**であり、ローカルの作業ディレクトリ・未コミットの変更・ローカルファイルには一切アクセスできない
- 使えるツールもclaude.aiアカウントに接続済みのMCPコネクタ（Gmail/Slack等）のみ。ローカルMCPサーバーは`.mcp.json`にコミットしない限り使えない
- 実行そのものは（Desktop Localのような「ディスパッチしても起動しない」総倒れ的な障害ではなく）ケースバイケースで失敗する不具合が報告されている：
  - [anthropics/claude-code#52533](https://github.com/anthropics/claude-code/issues/52533) / [#50955](https://github.com/anthropics/claude-code/issues/50955) — `/schedule`実行時に "remote claude.ai accountに接続できない" 認証エラーで失敗
  - [anthropics/claude-code#29022](https://github.com/anthropics/claude-code/issues/29022) — `create_scheduled_task` MCPツールがセッションに正しく注入されない
  - [anthropics/claude-code#36327](https://github.com/anthropics/claude-code/issues/36327) — スケジュール実行時にMCPサーバー/コネクタが使えない
- 2026-10-01の実機検証: RemoteTrigger API（Claude Codeの `RemoteTrigger` ツール）で作成した1回限りのRoutineは、Gmailコネクタの読み取り・リポジトリの読み取りとも成功した。claude.aiに接続済みのコネクタ（Gmail/Calendar等）は、作成時に自動で付与された。ただし9月以降も、未解決の報告（#93050 コネクタが使えない、#97266/#95384 権限確認待ちで止まる）が出ているので注意

### ④ bridge環境のCloud Routine（手元のマシンで実行）— 2026-10-01追加

- マシン上で `claude rc`（Remote Control）を起動しておくと、そのマシン・ディレクトリが「bridge」種別の実行環境として登録される（例: `lima-ubuntu:workspace:ac51`）。Routine作成時に `environment_id` でこれを指定すると、スケジュールの管理はクラウド側のまま、**実行は手元のマシン上で行われる**
- 実機で確認したこと（2026-10-01）:
  - 作業ディレクトリは `claude rc` を起動したディレクトリになる。未コミットのファイルも読める
  - claude.aiのGmailコネクタが使える（`mcp__claude_ai_Gmail__*`）
  - 手元のファイル（例: `~/.config/` 配下の認証情報）を使うスクリプトもBashで実行できる（例: LINE通知）
  - 実行のたびに claude.ai/code 上にセッションが作られる（プッシュ通知は来ない）
- 注意点:
  - **`claude rc` のCLIログインが切れると、無言で失敗する**。実例: 9/7から起動しっぱなしの `claude rc` が、いつの間にか期限切れになっていた（`claude auth status` が `loggedIn: false`）。実行ログには `Failed to authenticate: OAuth session expired and could not be refreshed` と出ていた。`claude auth login` で再ログインし、`claude rc` を再起動すれば直る。認証は、表示されたURLをブラウザで開いて返ってきたコードを貼る方式なので、別の端末からでもできる
  - マシンの再起動で `claude rc` が止まると動かない（tmux等で常駐させる）
  - 生存確認として、「該当なし」の回も通知を送る設計にすると、失敗に気づける

## 使い分けの判断基準

| ユースケース | 使える経路 |
|---|---|
| GitHub上にpush済みの内容 + claude.ai接続済みコネクタ（Gmail/Slack等）で完結する自動化 | ③ Cloud Routines（クラウド環境） |
| ローカルの作業ディレクトリ・未コミット状態・ローカルの認証情報を使う定期自動化（コネクタも使いたい） | ④ bridge環境のCloud Routine（`claude rc` の常駐が前提） |
| コネクタを使わず、ローカルだけで完結する定期自動化 | OSレベルのcron＋`claude -p`、またはスクリプト単体（SUUMO監視など） |

## Routineの設定でできないこと（2026-10-01時点、RemoteTrigger API）

- **モデル**は `job_config.ccr.session_context.model` で固定できる
- **effort（推論の深さ）**は指定できない。`session_context.effort` を付けても保存されない（エラーにはならない）
- **環境変数**も渡せない。`session_context.environment_variables` はHTTP 400（"not supported on triggers"）になり、`job_config.ccr.environment_variables` は保存されない。関連issue: [#98449](https://github.com/anthropics/claude-code/issues/98449)
- **削除**はAPIではできず、Web UI（https://claude.ai/code/routines）からのみ
- 一度実行した1回限りのRoutineは `run_once_at` を更新すれば再実行できる

## 代替アプローチ（コネクタ不要の場合）

OSレベルのcron（Windowsならタスクスケジューラ）から`claude -p "<プロンプト>"`を直接叩く。Claude Code純正のスケジュール機能に依存しないため、上記の既知バグ群の影響を受けない。ただしこの方式には別の制約（`claude -p`のヘッドレス実行ではGmail等アカウントレベルのリモートMCPコネクタが引き継がれない）があるため、[claude_code_headless_remote_mcp_limitation.md](claude_code_headless_remote_mcp_limitation.md)も合わせて参照すること。

## 発見の経緯

ユーザーがClaude Desktop app / Android appからClaude Codeを利用する中で「スケジュールタスクが動かない」と相談したのを起点に、claude-code-guideサブエージェントで公式ドキュメントとGitHub issueを調査して判明。関連する実装事例は`tasks/doing/20260902-gmail-action-mail-triage`のWORKLOG.md（2026-09-02〜03）を参照。

2026-10-01の再検証と④の実機検証は、`tasks/operating/20261001-gmail-task-sync-watcher` のWORKLOG.mdを参照。
