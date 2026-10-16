# Karabiner-Elementsを導入すると、macOS標準の「修飾キー」設定がそのキーボードで効かなくなる

## 結論

Karabiner-Elementsは、物理キーボードの入力を自分で受け取り、変換したうえで自前の仮想キーボードとしてmacOSに渡す。macOS標準の「修飾キー」設定（システム設定 > キーボード > キーボードショートカット > 修飾キー）は、物理キーボードのvendor/product IDに紐づいているため、仮想キーボード経由の入力には当たらない。macOS側の設定が消えるわけではなく、迂回されて効かなくなる。

このキーボードの修飾キーの割り当ては、Karabinerの「Simple Modifications」でデバイス指定して設定し直す。

## 仕組み

```
通常:           物理キーボード → macOS
Karabiner導入後: 物理キーボード → Karabiner本体（キー変換）→ 仮想キーボード → macOS
```

- キー割り当ての判断をするのはKarabinerの本体プロセス（Core-Service）。仮想キーボードは、変換後の入力をmacOSに渡す出力口。
- macOSには、もともとデフォルトの仮想キーボードは存在しない。Karabinerは、物理キーボードとmacOSの間に割り込む。
- マウス操作（`mouse_key`）も同様に、仮想ポインティングデバイス経由で出力される。
- 「物理キーボードを排他的に掴む」という細部は、既知の挙動としての理解で、公式ドキュメントでの再確認はしていない。

## 症状

- Karabiner導入前はCaps LockをControlとして使えていたが、導入後に効かなくなった（例: `Caps Lock + K`でのカナ変換）
- システム設定の「修飾キー」画面には、古い設定が残ったまま表示される

## 確認方法（読み取りのみ）

```bash
# macOS側の修飾キー設定（キーは<vendor>-<product>-0の形式、値は10進数のHID用法コード）
defaults -currentHost read -g | grep -A8 -i modifiermapping

# 接続中デバイスのvendor/product IDと、Karabinerの仮想デバイス
karabiner_cli --list-connected-devices
```

- 物理キーボードのID（今回は7247-141）と、Karabinerの仮想キーボードのID（1452-593）は別。
- 値は`30064771xxx`の形で、下位8ビットがキーコード。例: `...129`=Caps Lock（0x39）、`...296`=左Control（0xE0）、`...299`=左Command（0xE3）、`...300`=右Control（0xE4）、`...303`=右Command（0xE7）。

## 対処法

`~/.config/karabiner/karabiner.json`のプロファイルに、デバイス指定の`simple_modifications`を追加する（Karabinerの設定画面の「Simple Modifications」でデバイスを選んで設定してもよい）。

```json
"devices": [
  {
    "identifiers": { "is_keyboard": true, "is_pointing_device": false, "product_id": 141, "vendor_id": 7247 },
    "simple_modifications": [
      { "from": { "key_code": "caps_lock" },     "to": [{ "key_code": "right_control" }] },
      { "from": { "key_code": "left_control" },  "to": [{ "key_code": "left_command" }] },
      { "from": { "key_code": "right_control" }, "to": [{ "key_code": "right_command" }] }
    ]
  }
]
```

- 上は、このキーボード（サンワサプライ SKB-SL32CBK）でmacOS側に設定していた3つの割り当てを、そのまま再現した例。
- Karabinerは設定ファイルの変更を自動で読み込む。

## 発見の経緯

Mac miniにKarabiner-Elementsを導入（キーボードでマウス操作するルール）した直後に、Caps Lock+Kでのカナ変換が効かなくなったことで発覚。詳細は`tasks/doing/20261016-karabiner-keyboard-mouse`のWORKLOG.md（2026-10-16）を参照。
