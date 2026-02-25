# **オブジェクト指向における再利用のためのデザインパターン（GoF本）完全要約**

**著者**: Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides（通称 Gang of Four）

**初版**: 1994年

**原題**: *Design Patterns: Elements of Reusable Object-Oriented Software*

## **本書の概要と目的**

GoF本は、オブジェクト指向設計において繰り返し現れる「よくある問題とその解決策」を23のパターンとして体系化した書籍。パターンとは「特定のコンテキストにおける問題に対する汎用的な解決策」であり、コードそのものではなく設計の「テンプレート」として機能する。

### **本書が伝えようとしていること**

* **再利用性の高い設計**を実現するための原則と実践  
* 設計の意図を共通言語（パターン名）で表現することで**チーム内のコミュニケーション**を効率化  
* 変更に強い、拡張しやすいソフトウェアを作るための**思考の型**を提供  
* 経験豊富な設計者の知見を形式化し、初中級者でも活用できるようにする

### **本書が繰り返し強調する設計原則**

| 原則 | 内容 |
| :---- | :---- |
| **インタフェースに対してプログラムする** | 実装ではなく抽象（インタフェース・抽象クラス）に依存せよ |
| **継承よりコンポジションを優先する** | クラス継承は強い結合を生む。オブジェクトの委譲を活用せよ |
| **変わるものを変わらないものから分離する** | 変化しやすい部分を局所化し、拡張の影響範囲を限定せよ |
| **Open/Closed Principle** | 拡張に対して開いており、修正に対して閉じているべき |

### **パターンの分類軸**

GoFはパターンを直交する2つの軸（目的とスコープ）で分類している。

**目的（Purpose）による分類**

* **生成（Creational）**: オブジェクトの生成に関するパターン（5種）  
* **構造（Structural）**: クラス・オブジェクトの構造的な組み合わせに関するパターン（7種）  
* **振る舞い（Behavioral）**: オブジェクト間の責任の割り当てや通信に関するパターン（11種）

**スコープ（Scope）による分類**

* **クラススコープ**: クラスとそのサブクラス間の関係を扱い、継承を使って静的（コンパイル時）に解決する。  
* **オブジェクトスコープ**: 実行時にインスタンス化されたオブジェクト間の動的な関係性を扱い、コンポジション（委譲）を使って動的に解決する。

#### **目的とスコープの直交分類マトリクス**

| 目的 / スコープ | クラススコープ（継承ベース・静的解決） | オブジェクトスコープ（コンポジションベース・動的解決） |
| :---- | :---- | :---- |
| **生成型 (Creational)** | Factory Method | Abstract Factory, Builder, Prototype, Singleton |
| **構造型 (Structural)** | Adapter (Class Version) | Adapter (Object Version), Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| **振る舞い型 (Behavioral)** | Interpreter, Template Method | Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Visitor |

**💡 設計哲学の洞察**:

上記のマトリクスから、23パターンの大多数が「オブジェクトスコープ」に分類されていることがわかる。これは、GoFが基本原則として強調した「**クラス継承よりも、オブジェクトのコンポジション（委譲）を優先せよ**」という強いアーキテクチャ上のメッセージを如実に表している。

## **生成に関するパターン（Creational Patterns）**

オブジェクトの**生成プロセスを抽象化**し、クライアントが具体的なクラスに依存しないようにする。

### **1. Abstract Factory（抽象ファクトリ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Kit |
| **アナロジー** | 家具の製造工場（モダン・クラシックなど統一された様式の家具一式を生産） |

#### **どんなパターンか**

関連するオブジェクト群（＝ファミリー）を、具体的なクラスを指定せずに生成するためのインタフェースを提供する。ファクトリ自体を抽象化し、ファミリーごとに具体的なファクトリを用意する。

#### **何が実現できるか**

* 関連するオブジェクト群の整合性を保証できる（異なるファミリーの部品が混在しない）  
* クライアントコードを具体的な製品クラスから完全に分離できる  
* ファミリー全体の切り替えが容易になる

#### **どういう場面で使うか**

* UIウィジェットのテーマ切り替え（Windows風 vs macOS風）  
* DBドライバの切り替え（MySQL用クエリビルダ vs PostgreSQL用）  
* テスト環境と本番環境でオブジェクト生成を切り替えたいとき

#### **トレードオフと注意点**

新しい種類の製品を追加するとき、すべての具体的ファクトリを修正する必要があるため、製品の種類が頻繁に増える場合は負荷が高い（**インターフェースの硬直化**）。

#### **JavaScriptサンプルコード**

```javascript
// 抽象ファクトリ（インタフェース相当）
// JavaScript に interface はないので、規約として共通メソッド名を定義する

// --- 具体的ファクトリ① Windows風 ---
class WindowsFactory {
  createButton() { return new WindowsButton(); }
  createCheckbox() { return new WindowsCheckbox(); }
}

// --- 具体的ファクトリ② macOS風 ---
class MacFactory {
  createButton() { return new MacButton(); }
  createCheckbox() { return new MacCheckbox(); }
}

// --- 具体的製品 ---
class WindowsButton { render() { return "[ Windows Button ]"; } }
class MacButton     { render() { return "( Mac Button )";     } }
class WindowsCheckbox { render() { return "[x] Windows Checkbox"; } }
class MacCheckbox     { render() { return "(x) Mac Checkbox";     } }

// --- クライアント（どのファクトリかを意識しない） ---
function buildUI(factory) {
  const button   = factory.createButton();
  const checkbox = factory.createCheckbox();
  console.log(button.render());   // ファミリーによって結果が変わる
  console.log(checkbox.render());
}

buildUI(new WindowsFactory());
// => [ Windows Button ]
// => [x] Windows Checkbox

buildUI(new MacFactory());
// => ( Mac Button )
// => (x) Mac Checkbox
```

### **2. Builder（ビルダ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | サンドイッチのカスタマイズ注文（手順は同じだが具材の選択で結果が変わる） |

#### **どんなパターンか**

複雑なオブジェクトの**構築プロセスと表現を分離**する。同じ構築プロセスで異なる表現（形式）のオブジェクトを生成できるようにする。Directorが構築手順を制御し、Builder が各ステップを実装する。

#### **何が実現できるか**

* 複雑な初期化処理を段階的に行える  
* 同じ手順で異なる表現のオブジェクトを生成できる  
* コンストラクタへの引数爆発（telescoping constructor）を回避できる

#### **どういう場面で使うか**

* 設定ファイルの複数フォーマット出力（XML, JSON, Markdown）  
* クエリビルダ（SELECT文を段階的に組み立てる）  
* 複雑なドメインオブジェクトの生成（必須・任意項目が多い場合）

#### **トレードオフと注意点**

全体のコード量が増加し、システムにビルダー層という新たな複雑さが追加される。

#### **JavaScriptサンプルコード**

```javascript
// --- Builder ---
class QueryBuilder {
  constructor() {
    this.query = { table: "", conditions: [], columns: ["*"], limit: null };
  }
  from(table)   { this.query.table = table;              return this; }
  select(...cols){ this.query.columns = cols;             return this; }
  where(cond)   { this.query.conditions.push(cond);      return this; }
  limitTo(n)    { this.query.limit = n;                  return this; }

  build() {
    let sql = `SELECT ${this.query.columns.join(", ")} FROM ${this.query.table}`;
    if (this.query.conditions.length)
      sql += ` WHERE ${this.query.conditions.join(" AND ")}`;
    if (this.query.limit !== null)
      sql += ` LIMIT ${this.query.limit}`;
    return sql;
  }
}

// --- 利用側：メソッドチェーンで段階的に組み立てる ---
const sql = new QueryBuilder()
  .from("users")
  .select("id", "name", "email")
  .where("age > 20")
  .where("active = 1")
  .limitTo(10)
  .build();

console.log(sql);
// => SELECT id, name, email FROM users WHERE age > 20 AND active = 1 LIMIT 10
```

### **3. Factory Method（ファクトリメソッド）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | クラスパターン |
| **別名** | Virtual Constructor |
| **アナロジー** | 自動車工場の車種別製造ライン（抽象的な車作りを各専用ラインに委ねる） |

#### **どんなパターンか**

オブジェクト生成のためのインタフェースを定義し、どのクラスをインスタンス化するかは**サブクラスに決定させる**。Creatorクラスはファクトリメソッドを定義し、サブクラスがそれをオーバーライドして具体的な製品を返す。

#### **何が実現できるか**

* 生成するクラスの決定をサブクラスに委ねることで柔軟な拡張が可能  
* クライアントが具体的な製品クラスを知らずに済む  
* フレームワーク側でオブジェクト生成の型を決め、アプリ側で具体化できる

#### **どういう場面で使うか**

* フレームワークの設計（テンプレートメソッドとセットで使われることが多い）  
* ロガーやコネクションの生成をサブクラスごとに変えたいとき  
* プラグインアーキテクチャでの製品生成

#### **トレードオフと注意点**

製品の種類が増えるたびに、対応する新しいクリエイターのサブクラスを作成しなければならないため、**クラスの爆発的増加**を招く恐れがある。

#### **JavaScriptサンプルコード**

```javascript
// --- 抽象Creator（スーパークラス） ---
class Notifier {
  // ファクトリメソッド（サブクラスがオーバーライドする）
  createChannel() {
    throw new Error("createChannel() must be implemented");
  }
  // テンプレートメソッドとよく併用される
  send(message) {
    const channel = this.createChannel();
    channel.deliver(message);
  }
}

// --- 具体的Creator ---
class EmailNotifier extends Notifier {
  createChannel() { return new EmailChannel(); }
}
class SlackNotifier extends Notifier {
  createChannel() { return new SlackChannel(); }
}

// --- 具体的Product ---
class EmailChannel {
  deliver(msg) { console.log(`[Email] ${msg}`); }
}
class SlackChannel {
  deliver(msg) { console.log(`[Slack] ${msg}`); }
}

// --- 利用側：具体的なクラスを意識しない ---
function notify(notifier, message) {
  notifier.send(message);
}

notify(new EmailNotifier(), "デプロイが完了しました");
// => [Email] デプロイが完了しました
notify(new SlackNotifier(), "デプロイが完了しました");
// => [Slack] デプロイが完了しました
```

### **4. Prototype（プロトタイプ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | 3Dプリンター（マスターモデルのデータを読み込み、最初から作らずに複製する） |

#### **どんなパターンか**

生成するオブジェクトの種類を、プロトタイプとなるインスタンスを使って**クローン（複製）することで指定**する。クラスからnewするのではなく、既存インスタンスをコピーして新しいオブジェクトを作る。

#### **何が実現できるか**

* クラスを知らなくてもオブジェクトを複製できる  
* 初期化コストの高いオブジェクトを効率的に生成できる  
* 実行時に動的にオブジェクトの種類を追加・変更できる

#### **どういう場面で使うか**

* ゲームのキャラクターや敵の大量生成（原型から複製）  
* 設定済みオブジェクトのテンプレートとして使いたいとき  
* クラス階層を増やしたくないが種類を増やしたいとき

#### **トレードオフと注意点**

循環参照を持つ複雑なオブジェクトグラフをクローンする際、「シャローコピー」と「ディープコピー」のどちらを採用するかという、**複雑なメモリ管理と実装の技術的難易度**が課題となる。

#### **JavaScriptサンプルコード**

```javascript
// --- Prototype ---
class EnemyCharacter {
  constructor(type, hp, skills) {
    this.type   = type;
    this.hp     = hp;
    this.skills = [...skills]; // ディープコピーの例
  }

  clone() {
    return new EnemyCharacter(this.type, this.hp, this.skills);
  }
}

// --- プロトタイプのレジストリ（テンプレート管理） ---
const prototypes = {
  goblin:  new EnemyCharacter("Goblin",  30, ["bite", "scratch"]),
  dragon:  new EnemyCharacter("Dragon", 500, ["fire", "fly", "roar"]),
};

// --- 利用側：new せずにクローンで大量生成 ---
const g1 = prototypes.goblin.clone();
const g2 = prototypes.goblin.clone();
g2.hp = 50; // 個別に調整してもプロトタイプには影響しない

console.log(g1.hp); // => 30
console.log(g2.hp); // => 50
console.log(prototypes.goblin.hp); // => 30（元は不変）
```

### **5. Singleton（シングルトン）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | 一国における唯一の大統領（誰もが唯一の窓口を通して決定にアクセスする） |

#### **どんなパターンか**

あるクラスのインスタンスが**1つだけ**存在することを保証し、そのインスタンスへのグローバルなアクセスポイントを提供する。

#### **何が実現できるか**

* システム全体で共有される唯一のリソースを管理できる  
* インスタンスの無秩序な生成を防げる

#### **どういう場面で使うか**

* 設定管理クラス、ロガー、DBコネクションプール  
* キャッシュ管理  
* イベントバス、ワークキューなどのグローバル状態管理

#### **トレードオフと注意点**

テストがしにくい、グローバル状態を持つため隠れた依存関係が生まれやすい、マルチスレッド環境での並行処理の安全性確保に注意が必要など批判も多い。現代では DI（依存性注入）との組み合わせや代替が検討されることが多い。

#### **JavaScriptサンプルコード**

```javascript
// --- Singleton ---
class Logger {
  constructor() {
    if (Logger._instance) return Logger._instance;
    this.logs = [];
    Logger._instance = this;
  }

  log(message) {
    const entry = `[${new Date().toISOString()}] ${message}`;
    this.logs.push(entry);
    console.log(entry);
  }

  getLogs() { return [...this.logs]; }
}

// --- 利用側 ---
const logger1 = new Logger();
const logger2 = new Logger();

logger1.log("アプリ起動");
logger2.log("DBに接続");

console.log(logger1 === logger2); // => true（同一インスタンス）
console.log(logger1.getLogs().length); // => 2（どちらから参照しても同じ履歴）
```

## **構造に関するパターン（Structural Patterns）**

クラスやオブジェクトを**より大きな構造へ組み合わせる**方法を扱う。

### **6. Adapter（アダプタ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | クラス・オブジェクトパターン |
| **別名** | Wrapper |
| **アナロジー** | USB規格の変換アダプター（形状やプロトコルを変換して繋ぐ） |

#### **どんなパターンか**

インタフェースの互換性がないクラスを、クライアントが期待するインタフェースに変換する。既存クラスを「包んで」別のインタフェースとして見せる。

#### **何が実現できるか**

* 既存のクラスを変更せずに再利用できる  
* 異なるインタフェースを持つクラスを統一的に扱える

#### **どういう場面で使うか**

* サードパーティライブラリを既存のコードベースに組み込む  
* レガシーコードを新しいインタフェースに合わせたいとき  
* 外部APIのレスポンス形式を内部モデルに変換するとき

#### **トレードオフと注意点**

新しいレイヤーが挟まるため、呼び出しの階層が深くなり、システム全体のコード構造が複雑化する。

#### **JavaScriptサンプルコード**

```javascript
// --- 既存クラス（変更不可のサードパーティ想定） ---
class LegacyPaymentAPI {
  makePayment(amountInCents) {
    console.log(`旧API決済: ${amountInCents}セント`);
  }
}

// --- クライアントが期待するインタフェース ---
// pay(amountInYen) を呼び出したい

// --- Adapter ---
class PaymentAdapter {
  constructor() {
    this.legacyAPI = new LegacyPaymentAPI();
  }
  // 円をセントに変換して旧APIに委譲
  pay(amountInYen) {
    const cents = Math.round(amountInYen * 0.67); // 簡易換算
    this.legacyAPI.makePayment(cents);
  }
}

// --- 利用側：旧APIを意識せず新インタフェースで呼ぶ ---
const payment = new PaymentAdapter();
payment.pay(1500);
// => 旧API決済: 1005セント
```

### **7. Bridge（ブリッジ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Handle/Body |
| **アナロジー** | 万能リモコン（共通の抽象化）と多種多様な電子機器（独立した実装） |

#### **どんなパターンか**

**抽象化と実装を分離**し、両者を独立して変化・拡張できるようにする。継承の代わりにコンポジションを使い、抽象化レイヤーと実装レイヤーをブリッジ（橋）でつなぐ。

#### **何が実現できるか**

* 抽象と実装のクラス階層を独立して拡張できる  
* クラスの組み合わせ爆発を防げる  
* 実行時に実装を切り替えられる

#### **どういう場面で使うか**

* 複数のOS向けに同じUIウィジェットを実装する場合  
* 描画システムで「形状」と「描画API（OpenGL/DirectX）」を独立させたいとき  
* デバイスドライバの抽象化

#### **トレードオフと注意点**

初期の設計難易度が高く、抽象化層と実装層の2つの階層を設計しなければならないため、コード量が増加する。

#### **JavaScriptサンプルコード**

```javascript
// --- 実装層（Implementor）：通知チャネル ---
class EmailSender  { send(msg) { console.log(`[Email]  ${msg}`); } }
class SlackSender  { send(msg) { console.log(`[Slack]  ${msg}`); } }

// --- 抽象層（Abstraction）：通知の種類 ---
class Notification {
  constructor(sender) { this.sender = sender; } // Bridgeで実装を保持
}

class AlertNotification extends Notification {
  notify(msg) { this.sender.send(`🚨 ALERT: ${msg}`); }
}
class InfoNotification extends Notification {
  notify(msg) { this.sender.send(`ℹ️ INFO: ${msg}`); }
}

// --- 利用側：「通知の種類」と「チャネル」を独立して組み合わせる ---
const emailAlert = new AlertNotification(new EmailSender());
const slackInfo  = new InfoNotification(new SlackSender());

emailAlert.notify("サーバーが落ちました");
// => [Email]  🚨 ALERT: サーバーが落ちました
slackInfo.notify("デプロイが完了しました");
// => [Slack]  ℹ️ INFO: デプロイが完了しました
```

### **8. Composite（コンポジット）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | ツリー構造 |
| **アナロジー** | 企業の組織図（個々の従業員と、従業員を束ねる部門を同じように扱う） |

#### **どんなパターンか**

オブジェクトをツリー構造に組み合わせ、**個々のオブジェクトとオブジェクトの集合を同一に扱える**ようにする。「部分-全体」の階層構造を表現する。

#### **何が実現できるか**

* クライアントが葉ノードと複合ノードを区別せずに扱える  
* 再帰的な構造を自然に表現できる

#### **どういう場面で使うか**

* ファイルシステム（ファイル と ディレクトリを同じように操作）  
* UI コンポーネントツリー  
* 組織図、メニュー階層  
* 数式ツリー（項 と 演算子を同一視）

#### **トレードオフと注意点**

システムが極端に汎用的になるため、「特定の複合要素に特定の種類の葉しか追加できない」といった厳密な型制約をコンパイル時に強制することが困難になる。

#### **JavaScriptサンプルコード**

```javascript
// --- 共通インタフェース ---
class FileSystemItem {
  getSize() { throw new Error("getSize() must be implemented"); }
  print(indent = "") { throw new Error("print() must be implemented"); }
}

// --- 葉ノード（Leaf）：ファイル ---
class File extends FileSystemItem {
  constructor(name, size) { super(); this.name = name; this.size = size; }
  getSize()         { return this.size; }
  print(indent = "") { console.log(`${indent}📄 ${this.name} (${this.size}KB)`); }
}

// --- 複合ノード（Composite）：ディレクトリ ---
class Directory extends FileSystemItem {
  constructor(name) { super(); this.name = name; this.children = []; }
  add(item)         { this.children.push(item); return this; }
  getSize()         { return this.children.reduce((sum, c) => sum + c.getSize(), 0); }
  print(indent = "") {
    console.log(`${indent}📁 ${this.name}/`);
    this.children.forEach(c => c.print(indent + "  "));
  }
}

// --- 利用側：ファイルもディレクトリも同じ口で扱う ---
const root = new Directory("project")
  .add(new File("README.md", 2))
  .add(new Directory("src")
    .add(new File("index.js", 10))
    .add(new File("app.js", 20)))
  .add(new File("package.json", 1));

root.print();
// => 📁 project/
// =>   📄 README.md (2KB)
// =>   📁 src/
// =>     📄 index.js (10KB)
// =>     📄 app.js (20KB)
// =>   📄 package.json (1KB)
console.log(`合計: ${root.getSize()}KB`); // => 合計: 33KB
```

### **9. Decorator（デコレータ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Wrapper |
| **アナロジー** | プレーンなコーヒーへのミルクやホイップなどの動的なトッピング追加 |

#### **どんなパターンか**

オブジェクトに**動的に新しい責任（機能）を追加**する。サブクラス化よりも柔軟な機能拡張の代替手段。対象オブジェクトと同じインタフェースを持つラッパーとして実装する。

#### **何が実現できるか**

* クラス定義を変えずに機能を追加できる  
* 複数のデコレータを重ねることで組み合わせが自在  
* 継承による機能追加と比べてクラス数を抑えられる

#### **どういう場面で使うか**

* I/O ストリームへのバッファリング・暗号化・圧縮の追加（Java の InputStream が典型例）  
* ロギング・キャッシュ・バリデーションの横断的付加  
* ミドルウェアチェーン（Webフレームワークのリクエスト処理）

#### **トレードオフと注意点**

システム内に多数の小規模なラッパーオブジェクトが生成されるため、デバッグ時にオブジェクトの実際の状態を追跡することが難しくなる。

#### **JavaScriptサンプルコード**

```javascript
// --- 基底コンポーネント ---
class Coffee {
  cost()        { return 300; }
  description() { return "コーヒー"; }
}

// --- 基底デコレータ ---
class CoffeeDecorator {
  constructor(coffee) { this.coffee = coffee; }
  cost()        { return this.coffee.cost(); }
  description() { return this.coffee.description(); }
}

// --- 具体的デコレータ ---
class MilkDecorator extends CoffeeDecorator {
  cost()        { return this.coffee.cost() + 50; }
  description() { return this.coffee.description() + " + ミルク"; }
}
class WhipDecorator extends CoffeeDecorator {
  cost()        { return this.coffee.cost() + 100; }
  description() { return this.coffee.description() + " + ホイップ"; }
}
class SyrupDecorator extends CoffeeDecorator {
  cost()        { return this.coffee.cost() + 80; }
  description() { return this.coffee.description() + " + シロップ"; }
}

// --- 利用側：デコレータを自由に重ねる ---
let order = new Coffee();
order = new MilkDecorator(order);
order = new WhipDecorator(order);
order = new SyrupDecorator(order);

console.log(order.description()); // => コーヒー + ミルク + ホイップ + シロップ
console.log(order.cost());        // => 530
```

### **10. Facade（ファサード）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | 全自動コーヒーマシン（複雑な抽出工程を1つのボタン操作に隠蔽する） |

#### **どんなパターンか**

複雑なサブシステムに対して、**シンプルな統一インタフェース**を提供する。サブシステムの詳細を隠蔽し、使いやすい窓口を作る。

#### **何が実現できるか**

* クライアントとサブシステムの結合を疎にできる  
* サブシステムの使い方を単純化できる  
* 複雑なサブシステムの相互作用を一箇所に集約できる

#### **どういう場面で使うか**

* ライブラリやフレームワークの利用窓口の設計  
* 複数のサービスを束ねるAPI層（BFF: Backend for Frontend）  
* 複雑な初期化手順をカプセル化したいとき

#### **トレードオフと注意点**

Facade自体に機能を追加しすぎると、それがすべてのサブシステムと密結合する保守不可能な**ゴッド・オブジェクト**に変貌する危険性がある。

#### **JavaScriptサンプルコード**

```javascript
// --- 複雑なサブシステム群 ---
class Inventory {
  check(itemId) { console.log(`在庫確認: ${itemId}`); return true; }
}
class PaymentGateway {
  charge(amount) { console.log(`決済処理: ¥${amount}`); return true; }
}
class ShippingService {
  arrange(address) { console.log(`配送手配: ${address}`); }
}
class NotificationService {
  sendConfirmation(email) { console.log(`確認メール送信: ${email}`); }
}

// --- Facade：複雑な手順を1メソッドに隠蔽 ---
class OrderFacade {
  constructor() {
    this.inventory    = new Inventory();
    this.payment      = new PaymentGateway();
    this.shipping     = new ShippingService();
    this.notification = new NotificationService();
  }

  placeOrder({ itemId, amount, address, email }) {
    if (!this.inventory.check(itemId)) throw new Error("在庫なし");
    if (!this.payment.charge(amount))  throw new Error("決済失敗");
    this.shipping.arrange(address);
    this.notification.sendConfirmation(email);
    console.log("✅ 注文完了");
  }
}

// --- 利用側：サブシステムを一切意識しない ---
const facade = new OrderFacade();
facade.placeOrder({ itemId: "SKU-001", amount: 3980, address: "東京都...", email: "user@example.com" });
// => 在庫確認: SKU-001
// => 決済処理: ¥3980
// => 配送手配: 東京都...
// => 確認メール送信: user@example.com
// => ✅ 注文完了
```

### **11. Flyweight（フライウェイト）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | ねじ工場のマスター設計図共有（共通仕様を共有し、位置などの外部状態だけ個別管理） |

#### **どんなパターンか**

多数の細粒度オブジェクトを効率的にサポートするため、**共有によってメモリ使用量を削減**する。オブジェクトの状態を「内部状態（共有可能）」と「外部状態（共有不可）」に分けて管理する。

#### **何が実現できるか**

* 大量の類似オブジェクトのメモリ消費を大幅に削減できる  
* インスタンス数を減らしてパフォーマンスを向上できる

#### **どういう場面で使うか**

* テキストエディタでの文字オブジェクト管理（同じ文字は1インスタンスで共有）  
* ゲームの粒子システム（パーティクルの種類ごとに共有）  
* 地図アプリケーションの木・建物などの同種オブジェクト

#### **トレードオフと注意点**

状態の外部化と分離を管理するための追加ロジックが必要となり、システム構造が複雑化する。メモリ節約と計算コスト増加のトレードオフ評価が必要。

#### **JavaScriptサンプルコード**

```javascript
// --- Flyweight：共有される内部状態（樹種ごとの見た目情報） ---
class TreeType {
  constructor(name, color, texture) {
    this.name = name; this.color = color; this.texture = texture;
  }
  draw(x, y) {
    console.log(`🌳 [${this.name}] color:${this.color} at (${x}, ${y})`);
  }
}

// --- Flyweightファクトリ：同種はインスタンスを共有する ---
class TreeTypeFactory {
  constructor() { this._types = {}; }
  getType(name, color, texture) {
    const key = `${name}_${color}_${texture}`;
    if (!this._types[key]) {
      this._types[key] = new TreeType(name, color, texture);
      console.log(`新規 TreeType 生成: ${key}`);
    }
    return this._types[key];
  }
  count() { return Object.keys(this._types).length; }
}

// --- 外部状態（位置）は個々のTreeが持つ ---
const factory = new TreeTypeFactory();
const trees = [
  { type: factory.getType("Oak", "green", "rough"), x: 10, y: 20 },
  { type: factory.getType("Oak", "green", "rough"), x: 50, y: 80 }, // 共有
  { type: factory.getType("Pine", "dark-green", "smooth"), x: 30, y: 60 },
];

trees.forEach(t => t.type.draw(t.x, t.y));
console.log(`TreeType インスタンス数: ${factory.count()}`);
// => 新規 TreeType 生成: Oak_green_rough
// => 新規 TreeType 生成: Pine_dark-green_smooth
// => 🌳 [Oak] color:green at (10, 20)
// => 🌳 [Oak] color:green at (50, 80)
// => 🌳 [Pine] color:dark-green at (30, 60)
// => TreeType インスタンス数: 2（ツリー3本に対してインスタンスは2つのみ）
```

### **12. Proxy（プロキシ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Surrogate |
| **アナロジー** | クレジットカード（現金の実体を持たずに決済へのアクセスを代理・制御する） |

#### **どんなパターンか**

あるオブジェクトへのアクセスを制御するための**代理（代役）オブジェクト**を提供する。クライアントから見ると本物のオブジェクトと同じインタフェースを持つ。

#### **何が実現できるか**

* アクセス制御、遅延初期化、ロギング、キャッシュなどをクライアントに透過的に付加できる

#### **主なProxyの種類**

| 種類 | 説明 |
| :---- | :---- |
| **仮想プロキシ** | コストの高いオブジェクトを必要になるまで遅延初期化 |
| **リモートプロキシ** | ネットワーク越しのオブジェクトをローカルに見せる |
| **保護プロキシ** | アクセス権限を制御する |
| **スマートリファレンス** | 参照カウント、ロック管理などの付加処理を行う |

#### **どういう場面で使うか**

* 遅延ローディング（画像の遅延読み込み）  
* ORM の遅延ロード（Lazy Loading）  
* セキュリティプロキシ（ロールベースのアクセス制御）  
* RPC・RPCスタブ

#### **トレードオフと注意点**

リクエストの横取りや処理の遅延、キャッシュ管理によるオーバーヘッドが発生し、レスポンスタイムに影響を与える可能性がある。

#### **JavaScriptサンプルコード**

```javascript
// --- Real Subject：コストの高いDB検索 ---
class UserRepository {
  findById(id) {
    console.log(`💾 DB検索: user#${id}`);
    return { id, name: `User${id}`, email: `user${id}@example.com` };
  }
}

// --- Proxy：キャッシュ付き仮想プロキシ ---
class CachedUserRepository {
  constructor() {
    this.repository = new UserRepository();
    this.cache = {};
  }
  findById(id) {
    if (this.cache[id]) {
      console.log(`✅ キャッシュヒット: user#${id}`);
      return this.cache[id];
    }
    const user = this.repository.findById(id);
    this.cache[id] = user;
    return user;
  }
}

// --- 利用側：キャッシュの存在を意識しない ---
const repo = new CachedUserRepository();
repo.findById(1); // => 💾 DB検索: user#1
repo.findById(2); // => 💾 DB検索: user#2
repo.findById(1); // => ✅ キャッシュヒット: user#1（DB検索なし）
```

## **振る舞いに関するパターン（Behavioral Patterns）**

オブジェクト間の**アルゴリズムや責任の割り当て**を扱う。

### **13. Chain of Responsibility（責任の連鎖）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | カスタマーサポートの担当者エスカレーション（一次対応から専門家への順次転送） |

#### **どんなパターンか**

リクエストを処理できるオブジェクトが複数ある場合、それらを**連鎖（チェーン）としてつなぎ、リクエストを順番に渡していく**。どのオブジェクトが処理するかは動的に決まる。

#### **何が実現できるか**

* 送信者と受信者の結合を解消できる  
* チェーンの構成を実行時に柔軟に変更できる  
* 処理の担当者を送信者側で意識しなくて済む

#### **どういう場面で使うか**

* ログレベルに応じたロガーの切り替え  
* Webフレームワークのミドルウェアチェーン  
* GUI のイベントバブリング（ボタン→パネル→ウィンドウへ伝播）  
* 承認フロー（担当者→マネージャー→役員への順次エスカレーション）

#### **トレードオフと注意点**

チェーンの終端まで達してもどのオブジェクトもリクエストを処理しなかった場合、**要求が無視されて消失してしまう未処理リスク**がある。また、デバッグ時の追跡が困難。

#### **JavaScriptサンプルコード**

```javascript
// --- ハンドラの基底クラス ---
class SupportHandler {
  setNext(handler) { this.next = handler; return handler; }
  handle(issue) {
    if (this.next) return this.next.handle(issue);
    console.log(`❌ 未解決: ${issue.title}`);
  }
}

// --- 具体的ハンドラ ---
class FaqHandler extends SupportHandler {
  handle(issue) {
    if (issue.level <= 1) { console.log(`[FAQ]     解決: ${issue.title}`); return; }
    super.handle(issue);
  }
}
class SupportAgentHandler extends SupportHandler {
  handle(issue) {
    if (issue.level <= 2) { console.log(`[担当者]   解決: ${issue.title}`); return; }
    super.handle(issue);
  }
}
class ManagerHandler extends SupportHandler {
  handle(issue) {
    if (issue.level <= 3) { console.log(`[マネージャー] 解決: ${issue.title}`); return; }
    super.handle(issue);
  }
}

// --- チェーンの組み立て ---
const faq = new FaqHandler();
const agent = new SupportAgentHandler();
const manager = new ManagerHandler();
faq.setNext(agent).setNext(manager);

// --- 利用側：チェーンの先頭にリクエストを投げるだけ ---
faq.handle({ title: "パスワードを忘れた", level: 1 });
// => [FAQ]     解決: パスワードを忘れた
faq.handle({ title: "請求金額が不正", level: 2 });
// => [担当者]   解決: 請求金額が不正
faq.handle({ title: "法的問題", level: 4 });
// => ❌ 未解決: 法的問題
```

### **14. Command（コマンド）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Action, Transaction |
| **アナロジー** | レストランのウェイターが扱う注文伝票（要求をオブジェクト化し、好きなタイミングで処理する） |

#### **どんなパターンか**

リクエスト（操作）を**オブジェクトとしてカプセル化**する。これにより、リクエストをキューに入れたり、ログに記録したり、取り消し（Undo）をサポートできるようにする。

#### **何が実現できるか**

* 操作の履歴管理とUndo/Redoの実装  
* 操作のキューイング・スケジューリング  
* マクロ（複数コマンドの組み合わせ）の実現  
* 操作の送信者と受信者の分離

#### **どういう場面で使うか**

* テキストエディタのUndo/Redo  
* GUIボタンへのアクション割り当て  
* トランザクション処理  
* タスクキュー・ジョブスケジューラ

#### **トレードオフと注意点**

アクションごとに個別のコマンドクラスを作成しなければならず、小規模なクラスが大量に発生する。

#### **JavaScriptサンプルコード**

```javascript
// --- Receiver：実際の操作を持つ ---
class TextEditor {
  constructor() { this.text = ""; }
  insertText(str) { this.text += str; }
  deleteText(len)  { this.text = this.text.slice(0, -len); }
}

// --- Commandインタフェース ---
class InsertCommand {
  constructor(editor, text) { this.editor = editor; this.text = text; }
  execute() { this.editor.insertText(this.text); }
  undo()    { this.editor.deleteText(this.text.length); }
}

// --- Invoker：コマンド履歴を管理 ---
class CommandManager {
  constructor() { this.history = []; }
  execute(command) {
    command.execute();
    this.history.push(command);
  }
  undo() {
    const command = this.history.pop();
    if (command) command.undo();
  }
}

// --- 利用側 ---
const editor  = new TextEditor();
const manager = new CommandManager();

manager.execute(new InsertCommand(editor, "Hello"));
manager.execute(new InsertCommand(editor, ", World"));
console.log(editor.text); // => Hello, World

manager.undo();
console.log(editor.text); // => Hello

manager.undo();
console.log(editor.text); // => （空文字）
```

### **15. Interpreter（インタープリタ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | クラスパターン |
| **アナロジー** | 電卓における加算減算の計算規則の解析と実行 |

#### **どんなパターンか**

言語の文法をクラスとして表現し、その言語の文を解釈（評価）するインタープリタを定義する。各文法規則を1クラスで表現し、ツリー構造で文を解釈する。

#### **何が実現できるか**

* 簡単な文法であれば容易に実装・拡張できる  
* 文法ルールを追加・変更しやすい

#### **どういう場面で使うか**

* SQL や正規表現の解釈  
* 数式パーサ・計算エンジン  
* スクリプト言語・DSL（ドメイン固有言語）の実装  
* ビジネスルールエンジン

#### **トレードオフと注意点**

文法が複雑になると、規則の数に比例して**クラス数が爆発的に増大**し管理不可能になるため、複雑な言語にはパーサジェネレータを使うほうが現実的。

#### **JavaScriptサンプルコード**

```javascript
// --- 終端式（数値リテラル） ---
class NumberExpression {
  constructor(value) { this.value = value; }
  interpret() { return this.value; }
}

// --- 非終端式（加算） ---
class AddExpression {
  constructor(left, right) { this.left = left; this.right = right; }
  interpret() { return this.left.interpret() + this.right.interpret(); }
}

// --- 非終端式（乗算） ---
class MultiplyExpression {
  constructor(left, right) { this.left = left; this.right = right; }
  interpret() { return this.left.interpret() * this.right.interpret(); }
}

// --- 利用側：(3 + 5) * 2 をASTとして表現して解釈 ---
const three = new NumberExpression(3);
const five  = new NumberExpression(5);
const two   = new NumberExpression(2);

const sum    = new AddExpression(three, five);      // 3 + 5
const result = new MultiplyExpression(sum, two);    // (3 + 5) * 2

console.log(result.interpret()); // => 16
```

### **16. Iterator（イテレータ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Cursor |
| **アナロジー** | 図書館の司書による標準的な蔵書案内（利用者は配架の仕組みを知らなくてよい） |

#### **どんなパターンか**

コレクション（集合体）の内部構造を公開せずに、その要素に**順次アクセスする方法**を提供する。traversal（走査）のロジックをコレクション本体から分離する。

#### **何が実現できるか**

* コレクションの内部実装を隠蔽できる  
* 複数の走査方法を同じコレクションに対して定義できる  
* 異なる種類のコレクションを統一した方法で走査できる

#### **どういう場面で使うか**

* 配列・リスト・ツリー・グラフへの統一的な走査  
* 多くの言語でビルトインとして実装されている（Python の for文、Java の forEach など）  
* カスタムデータ構造への遅延評価的なアクセス

#### **トレードオフと注意点**

単純な配列やリストに対して適用する場合、過剰な抽象化となりコードが冗長になる可能性がある。

#### **JavaScriptサンプルコード**

```javascript
// JavaScript では Symbol.iterator を実装することで
// ビルトインの for...of や スプレッド演算子に統合できる

// --- カスタムコレクション：ツリー構造のノードを幅優先で走査するイテレータ ---
class TreeNode {
  constructor(value, children = []) {
    this.value = value; this.children = children;
  }
}

class BreadthFirstIterator {
  constructor(root) { this.queue = [root]; }

  [Symbol.iterator]() { return this; }

  next() {
    if (this.queue.length === 0) return { done: true };
    const node = this.queue.shift();
    this.queue.push(...node.children);
    return { value: node.value, done: false };
  }
}

// --- 利用側：内部構造（キュー）を意識せずに走査できる ---
const tree = new TreeNode("root", [
  new TreeNode("A", [new TreeNode("A1"), new TreeNode("A2")]),
  new TreeNode("B", [new TreeNode("B1")]),
]);

for (const value of new BreadthFirstIterator(tree)) {
  console.log(value);
}
// => root → A → B → A1 → A2 → B1（幅優先順）
```

### **17. Mediator（メディエータ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | 仲介者 |
| **アナロジー** | 航空交通管制塔（航空機同士の直接通信を防ぎ、全指示を中央に集約するハブ） |

#### **どんなパターンか**

オブジェクト間の複雑な相互作用を、**Mediatorオブジェクトに集中させる**。多対多の複雑な依存関係を、各オブジェクト → Mediator という構造に整理する。

#### **何が実現できるか**

* オブジェクト間の結合を疎にできる（各オブジェクトはMediatorだけを知ればよい）  
* オブジェクト間の通信ロジックを一箇所に集約できる  
* 個々のオブジェクトの再利用性が高まる

#### **どういう場面で使うか**

* チャットルームの実装（各ユーザーはルームを通じてメッセージを送受信）  
* UIフォームの複数コンポーネント間の相互作用管理  
* ATCシステム（航空管制）

#### **トレードオフと注意点**

すべての通信ロジックが集中するため、システムが成長するにつれてMediatorクラス自体が極めて複雑に肥大化し、**保守不可能なゴッド・オブジェクト**に陥るリスクが高い。

#### **JavaScriptサンプルコード**

```javascript
// --- Mediator：チャットルーム ---
class ChatRoom {
  constructor() { this.users = []; }
  register(user) { this.users.push(user); user.chatRoom = this; }
  send(message, sender) {
    this.users
      .filter(u => u !== sender)
      .forEach(u => u.receive(message, sender.name));
  }
}

// --- Colleague：各ユーザーはチャットルームだけを知っている ---
class User {
  constructor(name) { this.name = name; this.chatRoom = null; }
  send(message)               { this.chatRoom.send(message, this); }
  receive(message, senderName) { console.log(`[${this.name}] ← ${senderName}: ${message}`); }
}

// --- 利用側：ユーザー同士は直接知り合わない ---
const room  = new ChatRoom();
const alice = new User("Alice");
const bob   = new User("Bob");
const carol = new User("Carol");

room.register(alice);
room.register(bob);
room.register(carol);

alice.send("こんにちは！");
// => [Bob]   ← Alice: こんにちは！
// => [Carol] ← Alice: こんにちは！
```

### **18. Memento（メメント）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Token |
| **アナロジー** | ビデオゲームの安全なセーブポイント（いつでも正確な状態に復元可能） |

#### **どんなパターンか**

オブジェクトの内部状態を、そのカプセル化を壊さずに**外部に保存し、後で復元**できるようにする。Originator（状態を持つ）、Memento（状態のスナップショット）、Caretaker（メメントの管理）の3つの役で構成する。

#### **何が実現できるか**

* カプセル化を維持したままのUndo/Redo  
* オブジェクト状態のスナップショット管理

#### **どういう場面で使うか**

* エディタのUndo/Redo  
* ゲームのセーブデータ  
* トランザクションのロールバック  
* Wizardフォームでの「前のステップに戻る」

#### **トレードオフと注意点**

オブジェクトの状態が大きく、かつ頻繁にスナップショットを取得する場合、メモリやストレージのリソースを大量に消費するパフォーマンス上の懸念がある。

#### **JavaScriptサンプルコード**

```javascript
// --- Memento：状態のスナップショット（外部からは中身を見せない） ---
class EditorMemento {
  #state; // private フィールドでカプセル化を守る
  constructor(state) { this.#state = state; }
  getState()         { return this.#state; }
}

// --- Originator：状態を持ち、メメントを生成・復元する ---
class Editor {
  constructor() { this.content = ""; }
  type(text)    { this.content += text; }
  save()        { return new EditorMemento(this.content); }
  restore(memento) { this.content = memento.getState(); }
}

// --- Caretaker：メメントの履歴を管理する ---
class History {
  constructor() { this.mementos = []; }
  push(memento) { this.mementos.push(memento); }
  pop()         { return this.mementos.pop(); }
}

// --- 利用側 ---
const editor  = new Editor();
const history = new History();

editor.type("Hello");
history.push(editor.save());

editor.type(", World");
history.push(editor.save());

editor.type("!!!");
console.log(editor.content); // => Hello, World!!!

editor.restore(history.pop());
console.log(editor.content); // => Hello, World

editor.restore(history.pop());
console.log(editor.content); // => Hello
```

### **19. Observer（オブザーバ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Dependents, Publish-Subscribe |
| **アナロジー** | 新聞の自動定期購読配達（発行元は購読者の詳細を知らなくても配達できる） |

#### **どんなパターンか**

あるオブジェクト（Subject）の状態が変化したとき、それに依存するすべてのオブジェクト（Observer）に**自動的に通知・更新**される、一対多の依存関係を定義する。

#### **何が実現できるか**

* SubjectとObserverの疎結合を保てる  
* Observerを動的に追加・削除できる  
* イベント駆動型の仕組みを実現できる

#### **どういう場面で使うか**

* MVCアーキテクチャのモデルとビューの同期  
* イベントシステム・リスナー  
* リアルタイム通知（在庫変化→販売ダッシュボード更新）  
* Pub/Sub メッセージング

#### **トレードオフと注意点**

状態の変更がObserverから別のObserverの更新を連鎖的に引き起こす場合、システム全体のデータフローの予測が困難になり、**カスケード更新による無限ループやパフォーマンス低下**を招く恐れがある。

#### **JavaScriptサンプルコード**

```javascript
// --- Subject（Publisher） ---
class StockPrice {
  constructor(symbol, price) {
    this.symbol    = symbol;
    this.price     = price;
    this.observers = [];
  }
  subscribe(observer)   { this.observers.push(observer); }
  unsubscribe(observer) { this.observers = this.observers.filter(o => o !== observer); }

  setPrice(price) {
    this.price = price;
    this.notify();
  }
  notify() {
    this.observers.forEach(o => o.update(this.symbol, this.price));
  }
}

// --- Observers ---
class Dashboard {
  update(symbol, price) { console.log(`[Dashboard]  ${symbol}: ¥${price}`); }
}
class AlertSystem {
  constructor(threshold) { this.threshold = threshold; }
  update(symbol, price) {
    if (price > this.threshold)
      console.log(`[Alert] ⚠️  ${symbol} が ¥${this.threshold} を超えました！ (¥${price})`);
  }
}

// --- 利用側 ---
const stock     = new StockPrice("BASE", 1500);
const dashboard = new Dashboard();
const alert     = new AlertSystem(2000);

stock.subscribe(dashboard);
stock.subscribe(alert);

stock.setPrice(1800);
// => [Dashboard]  BASE: ¥1800
stock.setPrice(2100);
// => [Dashboard]  BASE: ¥2100
// => [Alert] ⚠️  BASE が ¥2000 を超えました！ (¥2100)
```

### **20. State（ステート）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Objects for States |
| **アナロジー** | 自動販売機（硬貨の有無や金額といった内部状態に応じてボタンの反応が完全に切り替わる） |

#### **どんなパターンか**

オブジェクトの**内部状態が変化したとき、オブジェクトの振る舞いも変化する**ようにする。状態をクラスとして独立させ、状態ごとに振る舞いを実装する。

#### **何が実現できるか**

* 状態遷移ロジックを分散させ、見通しよく管理できる  
* 巨大なswitch/if文による状態管理を排除できる  
* 新しい状態を追加する際の影響を局所化できる

#### **どういう場面で使うか**

* 注文の状態管理（受付中→処理中→発送済み→完了）  
* ネットワーク接続状態の管理  
* ゲームキャラクターの状態遷移（通常→攻撃中→死亡）  
* ワークフローエンジン

#### **トレードオフと注意点**

状態の数だけ小さな状態クラスを作成しなければならないため、システム全体のクラス数が著しく増加し、管理コストが上がる。

#### **JavaScriptサンプルコード**

```javascript
// --- Stateインタフェース（各状態の振る舞い） ---
class IdleState {
  insertCoin(machine) {
    console.log("コインが投入されました");
    machine.setState(new HasCoinState());
  }
  pressButton(machine) { console.log("先にコインを入れてください"); }
}

class HasCoinState {
  insertCoin(machine) { console.log("すでにコインが入っています"); }
  pressButton(machine) {
    console.log("🥤 商品を提供します");
    machine.setState(new IdleState());
  }
}

// --- Context：現在の状態を保持し、処理を委譲する ---
class VendingMachine {
  constructor() { this.state = new IdleState(); }
  setState(state) { this.state = state; }
  insertCoin()    { this.state.insertCoin(this); }
  pressButton()   { this.state.pressButton(this); }
}

// --- 利用側：状態に応じて同じ操作でも振る舞いが変わる ---
const machine = new VendingMachine();

machine.pressButton();  // => 先にコインを入れてください
machine.insertCoin();   // => コインが投入されました
machine.insertCoin();   // => すでにコインが入っています
machine.pressButton();  // => 🥤 商品を提供します
machine.pressButton();  // => 先にコインを入れてください（Idleに戻った）
```

### **21. Strategy（ストラテジ）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **別名** | Policy |
| **アナロジー** | 目的地への移動手段（車・徒歩・バス）を状況に応じてユーザー自身が選択するトラベルプラン |

#### **どんなパターンか**

アルゴリズムのファミリーを定義し、それぞれをカプセル化して交換可能にする。アルゴリズムをクライアントから独立して変更できるようにする。

#### **何が実現できるか**

* アルゴリズムを実行時に切り替えられる  
* 条件分岐なしにアルゴリズムのバリエーションを扱える  
* アルゴリズムをクラスとして独立させることで単体テストがしやすくなる

#### **どういう場面で使うか**

* ソートアルゴリズムの切り替え（バブル/クイック/マージ）  
* 支払い方法の切り替え（クレジットカード/ペイペイ/銀行振込）  
* ルーティングアルゴリズム（最短/最安/最速）  
* バリデーションロジックの差し替え

#### **トレードオフと注意点**

クライアントは適切な戦略を選択するために、各戦略クラスの挙動の違いや特性をあらかじめ理解しておかなければならないという負担が生じる。

#### **JavaScriptサンプルコード**

```javascript
// --- 戦略インタフェース（各支払い方法） ---
class CreditCardStrategy {
  pay(amount) { console.log(`💳 クレジットカード決済: ¥${amount}`); }
}
class PayPayStrategy {
  pay(amount) { console.log(`📱 PayPay決済: ¥${amount} (0.5%ポイント還元)`); }
}
class BankTransferStrategy {
  pay(amount) { console.log(`🏦 銀行振込: ¥${amount} (3営業日後反映)`); }
}

// --- Context：戦略を差し替え可能に保持する ---
class ShoppingCart {
  constructor() { this.items = []; this.strategy = null; }
  addItem(price)          { this.items.push(price); }
  setPaymentStrategy(s)   { this.strategy = s; }
  checkout() {
    const total = this.items.reduce((sum, p) => sum + p, 0);
    this.strategy.pay(total);
  }
}

// --- 利用側：実行時に戦略を切り替える ---
const cart = new ShoppingCart();
cart.addItem(1200);
cart.addItem(3800);

cart.setPaymentStrategy(new CreditCardStrategy());
cart.checkout();  // => 💳 クレジットカード決済: ¥5000

cart.setPaymentStrategy(new PayPayStrategy());
cart.checkout();  // => 📱 PayPay決済: ¥5000 (0.5%ポイント還元)
```

### **22. Template Method（テンプレートメソッド）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | クラスパターン |
| **アナロジー** | ケーキの基本レシピ（「混ぜる・焼く」の手順は同じだが、具材は派生レシピに委ねる） |

#### **どんなパターンか**

アルゴリズムの**骨格を基底クラスで定義**し、一部のステップをサブクラスに委ねる。アルゴリズム全体の構造を変えずに、特定のステップだけをサブクラスで変化させられるようにする。

#### **何が実現できるか**

* 共通のアルゴリズム構造を再利用しながら、可変部分だけを差し替えられる  
* コードの重複を排除できる  
* 「Hook（フック）」で任意のステップにデフォルト実装を持たせられる

#### **どういう場面で使うか**

* フレームワークで処理の流れを固定し、アプリ側が一部を実装する（IoC: 制御の反転）  
* データの読み込み→変換→保存というETLパイプラインの骨格  
* テストフレームワークのsetUp/tearDownの仕組み

#### **トレードオフと注意点**

継承による静的な結合であるため実行時の変更ができず、サブクラスの実装によっては親クラスが意図したアルゴリズムの全体構造を誤って破壊してしまうリスクがある。

#### **JavaScriptサンプルコード**

```javascript
// --- 抽象クラス：アルゴリズムの骨格を定義 ---
class DataExporter {
  // テンプレートメソッド：処理の順番を固定
  export(data) {
    const raw       = this.extract(data);   // 抽象ステップ
    const converted = this.transform(raw);  // 抽象ステップ
    this.load(converted);                   // 抽象ステップ
  }
  extract(data)       { throw new Error("extract() must be implemented"); }
  transform(data)     { throw new Error("transform() must be implemented"); }
  load(data)          { throw new Error("load() must be implemented"); }
}

// --- 具体的クラス①：CSV出力 ---
class CsvExporter extends DataExporter {
  extract(data)   { return data; }
  transform(data) { return data.map(row => Object.values(row).join(",")).join("\n"); }
  load(csv)       { console.log(`[CSV]\n${csv}`); }
}

// --- 具体的クラス②：JSON出力 ---
class JsonExporter extends DataExporter {
  extract(data)   { return data; }
  transform(data) { return JSON.stringify(data, null, 2); }
  load(json)      { console.log(`[JSON]\n${json}`); }
}

// --- 利用側：export() の呼び出し方は同じ、結果が変わる ---
const records = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];

new CsvExporter().export(records);
// => [CSV]
// => 1,Alice
// => 2,Bob

new JsonExporter().export(records);
// => [JSON]
// => [{ "id": 1, "name": "Alice" }, ...]
```

### **23. Visitor（ビジター）**

| 項目 | 内容 |
| :---- | :---- |
| **分類** | オブジェクトパターン |
| **アナロジー** | 動物園（動物の種類は固定で、専門家である「餌やり担当」や「獣医」が順次巡回して処理を行う） |

#### **どんなパターンか**

オブジェクト構造の要素に対して実行する**操作をVisitorとして分離**する。要素クラスを変更せずに、新しい操作を追加できるようにする。ダブルディスパッチを使って型に応じた処理を振り分ける。

#### **何が実現できるか**

* 既存のクラスを変更せずに新しい操作を追加できる  
* 型に応じた処理を型チェックなしに実現できる  
* 操作のロジックをオブジェクト構造から分離して集中管理できる

#### **どういう場面で使うか**

* ASTの各ノードに対するコード生成・型チェック・最適化などの複数操作  
* オブジェクト構造に対するシリアライズ、表示、集計などの異なる処理  
* 税計算など、同じオブジェクト群に対して頻繁に処理の種類が増えるケース

#### **トレードオフと注意点**

要素クラスの種類が増えると、すべてのVisitorに新しいvisitメソッドを追加する必要があり、オープン/クローズド原則の観点では逆に硬直しやすい（State/Strategyとトレードオフ関係にある）。また、新しい「操作」の追加は容易だが、新しい「要素の型」を追加しようとすると既存コードの修正が必要になるため、要素の種類が変動するシステムには絶対に適用してはならない。

#### **JavaScriptサンプルコード**

```javascript
// --- 要素クラス群（変更しない） ---
class Circle {
  constructor(radius) { this.radius = radius; }
  accept(visitor) { return visitor.visitCircle(this); } // ダブルディスパッチ
}
class Rectangle {
  constructor(width, height) { this.width = width; this.height = height; }
  accept(visitor) { return visitor.visitRectangle(this); }
}

// --- Visitor①：面積計算 ---
class AreaCalculator {
  visitCircle(circle)       { return Math.PI * circle.radius ** 2; }
  visitRectangle(rectangle) { return rectangle.width * rectangle.height; }
}

// --- Visitor②：JSON出力（要素クラスを変えずに新しい操作を追加） ---
class JsonSerializer {
  visitCircle(circle)       { return JSON.stringify({ type: "circle", radius: circle.radius }); }
  visitRectangle(rectangle) { return JSON.stringify({ type: "rect", w: rectangle.width, h: rectangle.height }); }
}

// --- 利用側 ---
const shapes = [new Circle(5), new Rectangle(4, 6)];
const areaCalc  = new AreaCalculator();
const serializer = new JsonSerializer();

shapes.forEach(s => {
  console.log(`面積: ${s.accept(areaCalc).toFixed(2)}`);
  console.log(`JSON: ${s.accept(serializer)}`);
});
// => 面積: 78.54
// => JSON: {"type":"circle","radius":5}
// => 面積: 24.00
// => JSON: {"type":"rect","w":4,"h":6}
```

## **全23パターン一覧表**

| # | パターン名 | 分類 | キーワード |
| :---- | :---- | :---- | :---- |
| 1 | Abstract Factory | 生成 | 製品ファミリー、ファクトリの抽象化 |
| 2 | Builder | 生成 | 段階的生成、表現の分離 |
| 3 | Factory Method | 生成 | サブクラスによる生成の決定 |
| 4 | Prototype | 生成 | クローン、インスタンスのコピー |
| 5 | Singleton | 生成 | 唯一のインスタンス、グローバルアクセス |
| 6 | Adapter | 構造 | インタフェース変換、Wrapper |
| 7 | Bridge | 構造 | 抽象と実装の分離、独立した拡張 |
| 8 | Composite | 構造 | ツリー構造、部分-全体の統一扱い |
| 9 | Decorator | 構造 | 動的な機能追加、透過的なラッピング |
| 10 | Facade | 構造 | 複雑なサブシステムの簡易窓口 |
| 11 | Flyweight | 構造 | 共有によるメモリ節約 |
| 12 | Proxy | 構造 | アクセス制御・遅延初期化の代理 |
| 13 | Chain of Responsibility | 振る舞い | 連鎖的リクエスト処理 |
| 14 | Command | 振る舞い | 操作のオブジェクト化、Undo/Redo |
| 15 | Interpreter | 振る舞い | 言語・文法のクラス表現 |
| 16 | Iterator | 振る舞い | 内部構造を隠した走査 |
| 17 | Mediator | 振る舞い | 相互作用の集中管理 |
| 18 | Memento | 振る舞い | 状態のスナップショットと復元 |
| 19 | Observer | 振る舞い | 一対多の変更通知 |
| 20 | State | 振る舞い | 状態に応じた振る舞いの変化 |
| 21 | Strategy | 振る舞い | アルゴリズムの交換可能な切り替え |
| 22 | Template Method | 振る舞い | アルゴリズムの骨格と可変ステップ |
| 23 | Visitor | 振る舞い | 操作の分離、ダブルディスパッチ |

## **パターン間の関係と使い分けのヒント**

### **よく混同されるパターンの比較**

| 比較対象 | 使い分けのポイント |
| :---- | :---- |
| **Strategy vs State** | Strategyはクライアントが意図的にアルゴリズムを選ぶ。Stateはオブジェクト自身が状態に応じて振る舞いを変える |
| **Decorator vs Proxy** | Decoratorは機能追加が目的。Proxyはアクセス制御・代替が目的。外見は似ているが意図が異なる |
| **Factory Method vs Abstract Factory** | Factory MethodはひとつのProduct生成をサブクラスに委ねる。Abstract Factoryは関連するProductのファミリー全体を生成する |
| **Composite vs Decorator** | どちらも再帰的構造を持つが、Compositeは「部分-全体」の構造表現、Decoratorは機能の付加が目的 |
| **Iterator vs Visitor** | Iteratorは「どう走査するか」、Visitorは「走査しながら何をするか」を分離する。組み合わせて使うことも多い |
| **Command vs Strategy** | Commandは操作の履歴管理・遅延実行が目的。Strategyはアルゴリズムの実行時切り替えが目的 |

### **よく組み合わせて使われるパターン**

* **Composite + Iterator**: ツリー構造を統一的に走査する  
* **Composite + Visitor**: ツリー構造の各ノードに複数の操作を追加する  
* **Factory Method + Template Method**: フレームワーク設計の基本コンビ  
* **Command + Memento**: Undo/Redo機能の完全実装  
* **Observer + Mediator**: 複雑なイベント連鎖を仲介者を通して整理する  
* **Decorator + Strategy**: 機能追加と動的なアルゴリズム切り替えを両立する  
* **MVCアーキテクチャ (Observer + Composite + Strategy)**: GUIフレームワークの基本。ModelとViewの同期をObserverで、Viewのツリー構造をCompositeで、ユーザー入力の処理(Controller)をStrategyで実現する古典的かつ強力な組み合わせ。

## **本書が述べる「良い設計」の評価軸**

本書では、設計品質を測る観点として以下が繰り返し登場する。これらはデザインパターンを評価・選択する際の軸にもなっている。

| 評価軸 | 内容 |
| :---- | :---- |
| **変更への柔軟性** | どこに変更が生じても影響が局所化されているか |
| **再利用性** | クラス・モジュールが文脈を超えて再利用可能か |
| **結合度（Coupling）** | クラス間の依存関係が最小限に抑えられているか |
| **凝集度（Cohesion）** | 1つのクラスが1つの明確な責任だけを持っているか |
| **拡張性** | 既存コードを変更せずに新しい機能を追加できるか |

## **補足：パターンを使う上での注意点と文脈**

GoF本自身が認めているように、デザインパターンには以下の注意が必要。

* **パターンの乱用（Over-engineering）**: シンプルな問題にパターンを当てはめると、不必要な複雑さを生む  
* **パターンはゴールではなく手段**: 「パターンを使うこと」が目的になってはいけない  
* **言語・実行環境による適用可否とビルトインの代替**: GoFパターンは当時のC++/Smalltalkを念頭に置いており、現代言語（Python/JavaScriptなど）ではビルトインの機能がパターンを完全に代替することも多い（例：Iteratorは各言語のforeach構文などに吸収され、Observerはリアクティブプログラミングの基盤に組み込まれている）。  
* **関数型パラダイムとの関わり**: 関数型プログラミングの台頭により、特定のオブジェクト指向パターン（特に可変状態の変更を前提とするもの）は、現代的なアーキテクチャでは別の手法で解決されることも多い。  
* **パターン名は共通語彙**: 現代において最大の価値は、手段としてのコード実装よりも「アーキテクチャの設計意図を他者に瞬時に伝えるための共通言語」としての側面にある。

*本ドキュメントは GoF 本（Design Patterns: Elements of Reusable Object-Oriented Software, 1994）の内容をもとに作成しました。*