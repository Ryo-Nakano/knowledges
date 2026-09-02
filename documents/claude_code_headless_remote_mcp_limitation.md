# Claude Codeのヘッドレス実行（claude -p）ではアカウントレベルのリモートMCPコネクタが使えない

## 結論
`claude -p`によるヘッドレス実行では、Gmail・Google CalendarのようなアカウントレベルのリモートMCPコネクタ（`claude.ai Gmail`等、`claude mcp list`に`claude.ai <サービス名>`として表示されるもの）は一切引き継がれない。無人cron等でこれらのコネクタを使うタスクを組みたい場合、この制約を前提に設計する必要がある。

## 詳細

- `claude mcp list`をインタラクティブセッションで実行すると、Gmail・Calendarはアカウントレベルのリモートコネクタとして`✔ Connected`で表示される
- 同じ環境で、ヘッドレスの`claude -p`実行から`claude mcp list`を実行すると、これらは一切表示されない（Playwrightのようにローカルプロセスとして`-s user`スコープで明示登録したMCPサーバーだけが表示される）
- `claude mcp add --transport http gmail https://gmailmcp.googleapis.com/mcp/v1 -s user`で手動登録を試みても、「Incompatible auth server: does not support dynamic client registration」で接続に失敗する。GmailのMCPエンドポイントは標準的なMCPのOAuth動的クライアント登録（DCR）フローに対応しておらず、Claude Codeアプリ内でのみ完結する専用の認証機構に依存している模様
- なお、この手動登録の試行により、インタラクティブセッション側の`claude.ai Gmail`接続が一時的に上書きされ消えるという副作用が発生した（`claude mcp remove gmail -s user`で復旧可能）。同じURLを持つエントリを`-s user`で追加すると、既存の接続が上書きされうる点に注意

## 代替アプローチ

- MCPを介さず、対象サービスのREST APIを直接叩く構成にする（OAuthクライアントを作成しリフレッシュトークンを認証情報ファイルとして保存し、ヘッドレススクリプトから直接API呼び出しする）
- あるいはClaude Desktopアプリの「Scheduled tasks（Routines）」機能を使う（ただしこちらは2026年9月時点でRun nowが動作しない既知の未解決バグがある。[anthropics/claude-code#56480](https://github.com/anthropics/claude-code/issues/56480)）

## 発見の経緯
`tasks/doing/20260902-gmail-action-mail-triage`（Gmail定期チェック＆要アクション自動抽出）の実装中、scheduled-tasks方式が上記Issueの不具合で使えず、フォールバックのOSレベルcron+ヘッドレス`claude -p`方式に切り替えた際にこの制約が判明した。詳細な検証過程は同タスクのWORKLOG.md（2026-09-02〜03）を参照。
