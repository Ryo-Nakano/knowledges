# Gmail MCPで特定メールが検索できない不具合とIMAPフォールバック

claude.ai標準のGmail connector（Gmail MCP）では、特定のメールだけが検索結果から黙って外れることがある。エラーは出ず、0件が返るだけで終わる。回避策として、IMAP（アプリパスワード認証）でGmail本体を直接読む。

## 症状（2026-10-05に確認）
- 送信元・ドメイン・件名の一部など、どの条件で検索しても対象メールが0件になる。Gmailアプリでは見える
- 同じ受信箱の前後の時刻のメールは検索に出る。インデックス全体が遅れているのではなく、特定のメールだけが外れている
- 再現例: `from:torus.tokyu-hl.jp newer_than:3d` はMCPでは0件だったが、IMAPでは2件ヒットした（自動配信メール）
- 原因は確定していない。Anthropic側の修正や公式回答はない

## 既知のIssue（GitHub）
| Issue | 症状 | 状態（2026-10-05時点） |
|---|---|---|
| [claude-ai-mcp #349](https://github.com/anthropics/claude-ai-mcp/issues/349) | 特定送信者のメールだけ`search_threads`が0件。再接続しても、ラベルを作り直しても直らない | Closed as not planned |
| [claude-ai-mcp #988](https://github.com/anthropics/claude-ai-mcp/issues/988) | `get_message`はID指定なら成功する。`search_threads`はスレッドを黙って除外する | Open |
| [claude-code #82548](https://github.com/anthropics/claude-code/issues/82548) | 検索インデックスが実際のメールボックスより数時間〜数日遅れる。ID指定での取得は成功する | Closed as not planned |

## 回避策: IMAP + アプリパスワード
- 前提: 個人Gmailで2段階認証がオンになっていること。セキュリティキーだけで2段階認証を設定している場合や、高度な保護機能を使っている場合はアプリパスワードを発行できない
- 発行: https://myaccount.google.com/apppasswords でアプリ名を入れて作成する。16文字のパスワードが一度だけ表示される。4文字ごとに空白が入って表示されるが、認証では空白を除いて使う
- Gmail側でIMAPを有効にする設定は不要だった（2026-10時点）
- 実装の要点（Python標準の`imaplib`）:
  - 「すべてのメール」フォルダの名前は表示言語によって変わるため、`LIST`の結果から`\All`属性を持つフォルダを探す
  - `SELECT`は`readonly=True`で開き、本文は`BODY.PEEK[]`で取得する（既読にしない）
  - Gmail検索式は`UID SEARCH CHARSET UTF-8 X-GM-RAW <literal>`で使える。日本語の検索語も通る
  - `X-GM-MSGID`を16進表記にすると、Gmail APIやMCPの`get_message`で使うメッセージIDになる
- 注意: アプリパスワードは読み取りだけでなく送信もできる強い権限を持つ。Git管理外の場所に保存し、不要になったら同じ画面で削除して無効化する。Googleアカウントのパスワードを変えると自動で無効になる

## Gmail API直叩きとの比較
- Gmail APIはGoogle CloudでのOAuth設定が必要。OAuth同意画面が「テスト」状態のままだと、リフレッシュトークンが7日で切れる
- IMAPはアプリパスワードを発行するだけで使えるため、個人利用ならこちらが手軽

## 運用上の使い分け
- 基本はMCPを使う。期待するメールが0件だった場合や、ユーザーが「届いているはず」と言った場合に限りIMAPで取り直す
- 自動巡回では、0件が正常なのか不具合なのかを区別できないため、この切り替え条件は使えない

## 参照
- このワークスペースでの実装: `scripts/gmail-imap-fetch.py`（使い方はAGENTS.md「Gmail取得のフォールバック」）
- 経緯: `tasks/done/20261005-gmail-imap-fallback/`
