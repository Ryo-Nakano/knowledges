# Lima VM上でAppArmorのuser namespace制限をsysctlで恒久的に無効化した

## 結論

このLima VM（Mac上のApple Virtualization、開発全般に利用する個人環境で本タスク専用ではない）では、`/etc/sysctl.d/20-apparmor.conf`に以下を設定し、`sudo sysctl --system`で適用済み。

```
kernel.apparmor_restrict_unprivileged_userns = 0
```

これはマシン全体・OS再起動後も持続する**恒久的なセキュリティ設定の緩和**である。Codex CLI以外の用途でも、非特権ユーザーによるuser namespace作成を伴う操作（他のサンドボックスツール等）に影響しうる点に注意。

## 背景・原因

- Ubuntu 24.04以降は、AppArmorのデフォルト設定で非特権ユーザーによるuser namespace作成を制限している（`kernel.apparmor_restrict_unprivileged_userns=1`が既定値。`apparmor`パッケージ自身が`/usr/lib/sysctl.d/10-apparmor.conf`で出荷時設定しており、Lima固有の設定ではないことを確認済み）。
- Codex CLI（`codex exec`）は内部でbubblewrap（`bwrap`）によるサンドボックスを使用しており、この制限下では`bwrap: setting up uid map: Permission denied` / `bwrap: loopback: Failed RTM_NEWADDR: Operation not permitted`で**あらゆるシェルコマンド実行が失敗する**（`-s workspace-write`等のサンドボックスモードによらず発生）。
- Claude Code経由（ネストしたサンドボックス）だけでなく、ユーザー自身の通常ターミナルから直接`codex exec`を実行しても同一エラーが再現することを確認済み（2026-09-08）。これによりホストOS（Lima VM）自体のAppArmor設定が原因であり、Claude Code固有の問題ではないと確定した。

## 対応の妥当性（deep research裏取り済み）

- Ubuntu 24.04以降で広く知られた既知の事象。Codex CLI側もGitHub Issue（#14919, #15496, #16334, #15434, #17525, #18185等）を複数抱えている。
- OpenAI公式ドキュメント（sandboxing関連ページ）でも対処法として言及されている。
- Lima VMコミュニティでは「VM境界（ハイパーバイザによる隔離）を前提に、VM内部ではsysctlで当該制限を無効化するのが事実上の標準的な運用」とされている（VM自体がホストMacから隔離されているため、VM内でのuserns制限緩和による実害は限定的という理解）。
- 詳細な調査プロセス・一次情報は`tasks/doing/20260906-codex-compatibility/references/ubuntu_apparmor_userns_restrictions.md`を参照。

## 動作確認

`sysctl --system`適用後、`codex exec -s workspace-write "echo hello_from_codex_sandbox_test"`が実際にシェルコマンドを実行し成功することを確認済み。

## 元に戻す場合

`/etc/sysctl.d/20-apparmor.conf`を削除し、`sudo sysctl --system`を再実行すれば既定値（制限あり）に戻る。ただしCodex CLIのbubblewrapサンドボックスは再び機能しなくなる。

## 発見・対応の経緯

`tasks/doing/20260906-codex-compatibility`（Codex CLI互換化）のlongrun実行中、Codex CLI実機検証のすべてのステップがこの制限でブロックされることが判明。ユーザーへの平易な説明→deep research依頼→ユーザー承認、という手順を経て対応した。詳細な経緯は同タスクの`longrun/ESCALATIONS.md`（2026-09-08）を参照。
