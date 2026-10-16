# MacからMacへVS Code Remote - SSHで接続する設定と、ハマりポイント（macOS 26、鍵認証のみ）

## 結論

同一LAN内のMac（クライアント）から別のMac（サーバー）へ、鍵認証のみでSSH・VS Code Remote - SSHをつなぐ構成は、次の手順で成立した（MacBook Air → Mac mini M4、macOS 26 Tahoe、2026-10-16に実機で確認）。ネット上の手順書は、ホスト名などのプレースホルダや、環境に合わない前提を含むことがあるため、**実機の値を先に確認してから適用する**。

## 手順

### サーバー側（接続される側）
1. 「システム設定 > 一般 > 共有 > リモートログイン」をオンにする。「アクセスを許可」は使用ユーザーだけに絞る。
2. 「リモートユーザのフルディスクアクセスを許可」（リモートログインの`i`内）は、**最初はオフ**にする。作業フォルダがDesktop/Documents/Downloads・外付けディスクなどの保護フォルダ外であれば不要。SSH経由で`Operation not permitted`が出たときだけオンにする（最小権限）。
3. 接続先の名前は、手順書の例（`Mac-mini.local`等）ではなく、実機で確認した`LocalHostName`から作る: `scutil --get LocalHostName`の値に`.local`を付ける。
4. 非対話のSSH（`ssh host 'cmd'`）や拡張機能の裏処理でも`brew`・`node`等が見つかるよう、`~/.zshenv`にPATHを通す（下の「ハマりポイント」参照）。

### クライアント側（接続する側）
1. 鍵は用途が分かる名前で作る（例: `~/.ssh/mac-mini_ed25519`）。**パスフレーズは必ず設定する**。
   ```bash
   ssh-keygen -t ed25519 -C "macbook-air-to-mac-mini" -f ~/.ssh/mac-mini_ed25519
   ssh-add --apple-use-keychain ~/.ssh/mac-mini_ed25519
   ```
2. `~/.ssh/config`にホストを定義する。
   ```sshconfig
   Host mac-mini
       HostName <LocalHostName>.local
       User <サーバー側のユーザー名>
       IdentityFile ~/.ssh/mac-mini_ed25519
       IdentitiesOnly yes
       UseKeychain yes
       AddKeysToAgent yes
   ```
3. 公開鍵を登録する: `ssh-copy-id mac-mini`（初回のみサーバーのログインパスワードが必要）。
4. VS Codeの拡張機能`Remote - SSH`（`ms-vscode-remote.remote-ssh`）を入れ、`Remote-SSH: Connect to Host...`で`mac-mini`を選ぶ。

### パスワード認証の無効化（鍵認証が通った後）
macOSの`/etc/ssh/sshd_config`は`Include /etc/ssh/sshd_config.d/*`を持つ。`100-macos.conf`は`UsePAM yes`だけを設定している。sshdは**先に読んだ設定が優先**なので、`100-macos.conf`より前に読まれる名前（`050-`始まり）でドロップインを作る。

```bash
sudo tee /etc/ssh/sshd_config.d/050-disable-password-auth.conf <<'EOF'
PasswordAuthentication no
KbdInteractiveAuthentication no
EOF
sudo sshd -t && echo "設定OK"
```

- 再起動は不要で、新しい接続から有効になる。戻すときは当該ファイルを`sudo rm`する。
- 確認: サーバー上から`ssh -o PubkeyAuthentication=no -o BatchMode=yes <user>@localhost true`を実行し、`Permission denied (publickey)`になれば、パスワードのみの接続は拒否されている。そのうえでクライアントから鍵で新規接続できることも確認する。
- 鍵を失った場合は、サーバーの画面から直接操作して復旧する（リモートからは入れない）。

## ハマりポイント

### 1. 手順書のプレースホルダ・前提を、実機の値に置き換え忘れる
- ホスト名: 手順書の`Mac-mini.local`は実機の名前（例: `sks-ryoheinoMac-mini`）と異なっていた。`scutil --get LocalHostName`で確認する。
- ユーザー名・IP・鍵名も同様。

### 2. 非対話のSSHでは`~/.zprofile`が読まれず、PATHが通らない
- `ssh host 'which brew node'`のような非対話実行のPATHは`/usr/bin:/bin:/usr/sbin:/sbin`だけになり、`brew`・`node`・`claude`が見つからなかった（実機で再現）。
- 対策は、サーバー側の`~/.zshenv`（すべてのzsh起動で読まれる）にPATHを書くこと。重複しない書き方にする。
  ```zsh
  typeset -U path
  path=(/opt/homebrew/bin /opt/homebrew/sbin $HOME/.local/bin $HOME/.npm-global/bin $path)
  ```
- 確認: `env -i HOME=$HOME zsh -c 'which brew node'`（非対話・非ログインのシェルを再現）。
- 手順書は`cat >> ~/.zshenv`で追記する形だったが、再実行すると重複して追記される。冪等な書き方にする。
- VS Code統合ターミナルや、Remote - SSHサーバー側の環境解決が非ログインシェルで動くかは、公式から確認できなかった。ただ、上の症状は`ssh host 'cmd'`で再現するため、`.zshenv`に通しておけば安全。

### 3. 鍵名を後から変えると、`~/.ssh/config`に旧名の設定が残る
- 設定追記コマンドを`grep -q '^Host mac-mini$' ... || cat >> config`の形にして、鍵名を変える前後で2回流したところ、**後のブロックが「既にあるから」とスキップされ、旧鍵名（`id_ed25519`）のまま残った**。`no such identity`の警告と、`IdentitiesOnly`が付かないことで気づいた。
- 鍵名は先に決めてから`config`に書く。直すときは、該当ブロックだけを置換する（他のホスト設定を壊さないため）。
  ```bash
  perl -0pi -e 's/(Host mac-mini\n(?:[ \t]+.*\n)*?[ \t]+)IdentityFile ~\/\.ssh\/id_ed25519\n/$1IdentityFile ~\/.ssh\/mac-mini_ed25519\n    IdentitiesOnly yes\n/' ~/.ssh/config
  ssh -G mac-mini | grep -iE '^(identityfile|identitiesonly|hostname|user) '
  ```

### 4. Git認証は、サーバー側の既存鍵で足りるなら`ForwardAgent`は不要
- サーバー側にGitHub用のSSH鍵（`~/.ssh/github_ed25519`など）が既にあれば、サーバー上のGit操作はそれで通る。`ForwardAgent yes`は、サーバーが侵害されたときにクライアントの鍵を使われる恐れがあるため、必要がなければ付けない。

### 5. 接続できなくなったとき
- VS Code: `Remote-SSH: Kill VS Code Server on Host...`を実行。直らなければ`ssh <host> 'rm -rf ~/.vscode-server'`で再構築（次回接続時に自動で再インストールされる）。

## 制約

- 同一LAN内のみ。`.local`（mDNS）の名前とプライベートIPは、外のネットワークからは届かない。外出先から使うには、Tailscaleのような仮想ネットワークなどが別途必要。
- 引越し等でルーターが変わると、`.local`の名前解決が不安定になることがある。その場合はルーターでIPを固定割当にし、`HostName`を固定IPに書き換える。

## 出典・確度

- 実機確認: 2026-10-16（macOS 26、OpenSSH_10.3p1（Apple製）、MacBook Air → Mac mini M4）。タスク: `tasks/done/20261016-vscode-remote-ssh-mac-mini/`（WORKLOGに経緯）。
- 「リモートユーザのフルディスクアクセスを許可」の実在: [Apple Mac User Guide](https://support.apple.com/en-ca/guide/mac-help/mchlp1066)。
- 未確認: Remote - SSHサーバー側が環境をどのシェルで解決するか（公式の記載なし）。
