# リーダブルコード 完全観点まとめ

Dustin Boswell / Trevor Foucher 著『The Art of Readable Code』の観点を網羅的に整理し、さらに設計思想レベルのアプローチを加筆したドキュメント。

---

## 前提：リーダブルコードの根本原則

> **「コードは他の人が最短時間で理解できるように書かなければならない」**

「他の人」とは未来の自分も含む。理解しやすいコードは、バグが少なく、変更しやすく、チームの生産性を高める。コードは書かれる回数より読まれる回数のほうが圧倒的に多いため、読む側のコストを最小化することが最優先となる。

**⚠️ 「行数の少なさ＝可読性の高さ」ではない**

しばしば「コードを短くすれば読みやすくなる」と誤解されるが、物理的な行数の短さよりも、解読時間（認知負荷）の短さが優先される。ネストした三項演算子やメソッドチェーンの乱用などで過度に圧縮された1行のコードは、論理展開された数行のコードよりも理解に時間がかかることが多い。

---

## Part 1：表面上の改善（命名・書式・コメント）

### 1. 名前に情報を詰め込む

#### なぜ大切か

名前はコードの中で最も頻繁に目にするドキュメントである。曖昧な名前はコードを読むたびに「これは何をするものか」という解釈コストを生む。

#### どうするべきか

- **具体的な単語を選ぶ**：`get` より `fetch`・`download`、`size` より `height`・`numNodes` のように意図が伝わる単語を使う。
- **汎用的な名前を避ける**：`tmp`・`retval`・`foo` は理由がない限り使わない。ループ変数も `i` より意味のある名前にする。
- **単位・属性を名前に付加する**：`delay` より `delayMs`、`password` より `plaintextPassword` のように状態や単位を明示する。
- **スコープに応じた長さにする**：短いスコープでは短い名前、広いスコープでは長く説明的な名前を使う。
- **大文字・アンダースコアなどのフォーマットで情報を伝える**：定数は `UPPER_SNAKE`、クラスは `PascalCase`、変数・関数は `camelCase` など、プロジェクト内で一貫したルールを持つ。

#### 良いコードの例

```javascript
// NG：何のサイズか、何の遅延か、どんなパスワードかわからない
let size = 0;
const delay = 30;
const password = getPassword();

// OK：単位・属性・対象が名前に込められている
let numActiveUsers = 0;
const downloadTimeoutMs = 30000;
const plaintextPassword = getPassword();
```

```javascript
// NG：汎用的すぎる関数名
function getData(input) { ... }

// OK：何のデータを、どこから取得するか明確
async function fetchUserProfileFromDb(userId) { ... }
```

```javascript
// NG：ループ変数が意味を持たない
for (let i = 0; i < users.length; i++) {
  console.log(users[i].name);
}

// OK：何をループしているかが伝わる
for (const user of users) {
  console.log(user.name);
}
```

```javascript
// NG：定数なのに let で宣言、名前も役割不明
let x = 0.1;

// OK：const で不変性を示し、名前に意味を持たせる
const TAX_RATE = 0.1;
```

---

### 2. 誤解されない名前にする

#### なぜ大切か

名前が誤解を招くと、コードを読んだ人が間違った仮定のもとでコードを変更し、バグを生む。

#### どうするべきか

- **`filter` は曖昧**：「取り出す」なら `select`、「除外する」なら `exclude` を使う。
- **`clip` は曖昧**：「切り取る」なら `truncate`、「切り抜く」なら `remove` などより明確な動詞を使う。
- **限界値には `min` / `max` を使う**：`CART_TOO_BIG_LIMIT` より `MAX_ITEMS_IN_CART` のほうが境界が明確。
- **範囲には `first`/`last` または `begin`/`end` を使う**：`end` は「含まない終端」の慣習があるため一貫させる。
- **bool 変数には `is`・`has`・`can`・`should` を付ける**：`readPassword` は「読む動作」か「読んだ状態」か曖昧なので `needsPassword` や `isAuthenticated` にする。
- **否定形の名前を避ける**：`disableSsl = false` より `useSSL = true` のほうが読みやすい。

#### 良いコードの例

```javascript
// NG：LIMIT が上限か下限か、以上か超過かが曖昧
const CART_TOO_BIG_LIMIT = 10;
if (cartItems > CART_TOO_BIG_LIMIT) showError();

// OK：MAX_ で上限であることが明確、比較方向が自然になる
const MAX_ITEMS_IN_CART = 10;
if (cartItems > MAX_ITEMS_IN_CART) showError();
```

```javascript
// NG：readPassword は「読む動作」か「読んだフラグ」かわからない
let readPassword = true;

// OK：is / has / needs で boolean であることが明確
let needsPassword = true;
let isAuthenticated = false;
```

```javascript
// NG：否定形のフラグは二重否定が生まれやすい
const disableSsl = false;
if (!disableSsl) connectSecurely(); // 「disable しない → SSL 有効」と頭で変換が必要

// OK：肯定形にすると直読みできる
const useSSL = true;
if (useSSL) connectSecurely();
```

```javascript
// NG：filter が「含める」か「除く」か不明
const results = filterUsers(users, "inactive");

// OK：動作が一目瞭然
const results = excludeInactiveUsers(users);
```

---

### 3. 美しく整形する

#### なぜ大切か

コードのレイアウトは「読む流れ」を視覚的に支援する。整形が乱れていると脳のリソースが構文解析に取られ、ロジックの理解が遅れる。

#### どうするべきか

- **一貫したスタイルを使う**：インデント・括弧・スペースの使い方をプロジェクト全体で統一する。既存のスタイルに合わせることを優先する（Prettier 等のフォーマッターを活用する）。
- **縦の整列で対応関係を示す**：変数の代入やオブジェクトのプロパティを縦に揃えることで、差分が目立ちやすくなる（ただしやりすぎると変更コストが増えるため適度に）。
- **意味のある順序で並べる**：重要なものを先に、対応するファイルと同じ順序で書くなど、恣意的でない順序を持たせる。
- **コードを「段落」に分ける**：空行を使って処理の塊（初期化・データ取得・変換・出力など）を視覚的に分離する。
- **1行に詰め込みすぎない**：1行1処理を基本とし、ネストも浅く保つ。
- **視覚的な「不規則性」を排除するためのヘルパー関数を導入する**：引数が長すぎるなど、コードの見た目（シルエット）が不規則になる場合は、論理の共通化のためだけでなく「視覚的なノイズを消して見た目を揃えるためだけ」にヘルパー関数でラップする。

#### 良いコードの例

```javascript
// NG：処理の区切りがなく、何の固まりかわからない
async function processOrder(order) {
  const user = await getUser(order.userId);
  const product = await getProduct(order.productId);
  if (!user) throw new Error("User not found");
  if (!product) throw new Error("Product not found");
  const total = product.price * order.quantity;
  const tax = total * TAX_RATE;
  const finalPrice = total + tax;
  await saveOrder(order, finalPrice);
  await sendConfirmationEmail(user, order);
}

// OK：空行で「段落」を分け、処理のフェーズが目で追える
async function processOrder(order) {
  // データ取得
  const user = await getUser(order.userId);
  const product = await getProduct(order.productId);

  // バリデーション
  if (!user) throw new Error("User not found");
  if (!product) throw new Error("Product not found");

  // 金額計算
  const total = product.price * order.quantity;
  const tax = total * TAX_RATE;
  const finalPrice = total + tax;

  // 保存・通知
  await saveOrder(order, finalPrice);
  await sendConfirmationEmail(user, order);
}
```

```javascript
// NG：縦の対応関係がバラバラで差分が見えにくい
const userConfig  = { name: "Alice", role: "admin", active: true };
const guestConfig = { name: "Guest", role: "viewer", active: false };

// OK：縦に揃えると項目の対応関係がひと目でわかる
const userConfig  = { name: "Alice", role: "admin",  active: true  };
const guestConfig = { name: "Guest", role: "viewer", active: false };
```

```javascript
// NG：引数が長すぎてフォーマットが崩れ、視覚的なノイズになっている
assert.equal(user.getAttributes().subscription.status, "active");
assert.equal(guest.getAttributes().subscription.status, "inactive");

// OK：視覚的な不規則性を消すためだけにヘルパー関数を導入する
function assertStatus(userObj, expected) {
  assert.equal(userObj.getAttributes().subscription.status, expected);
}
assertStatus(user,  "active");
assertStatus(guest, "inactive");
```

---

### 4. コメントには何を書き、何を書かないか

#### なぜ大切か

コメントは「コードで表現しきれない情報」を補う手段であり、適切なコメントは理解速度を大幅に上げる。一方、コードをそのまま言い換えるだけのコメントは読む手間を増やすだけで価値がない。

#### どうするべきか

**書くべきでないコメント：**

- コードを見ればわかることを繰り返す（`i++; // i に 1 を足す`）
- 「ひどいコード」を説明するだけで、コードを直さない（コードを直す）

**書くべきコメント：**

- **「なぜ」を説明する**：コードが「何をしているか」はコード本体から読める。「なぜその実装を選んだか」「なぜこの順序か」をコメントに書く。
- **欠陥・TODO を明示する**：未完成・問題点をコメントとして残す。
- **定数の意味を説明する**：数値の根拠・由来をコメントに書く。
- **読者が驚く箇所を説明する**：「なぜこんな書き方をしているのか」と疑問を持ちそうな箇所には先手を打ってコメントを入れる。
- **関数・クラスに「全体像」を書く**：引数の意味、戻り値、副作用、利用上の注意を簡潔にまとめる（JSDoc が有効）。
- **入出力の「具体例（Corner Cases）」を視覚的に提示する**：自然言語で複雑な挙動を説明するより、実際の引数と戻り値のサンプルを書くことで、境界値の挙動などが一瞬で伝わる。

**コメントは正確に・簡潔に：**

- 代名詞は使わない（「それ」が何を指すか曖昧になる）
- 動作を「精密に」説明する（「これは〜をします」よりも「最大 5 個の〜を返します」のように具体的に）
- **情報密度の高い専門用語（標準語）を活用する**：「キャッシュ」「ヒューリスティック」といったソフトウェア工学の標準的なパターン名を使うことで、長い説明文を圧縮しコメントの画面占有率を下げる。

#### 良いコードの例

```javascript
// NG：コードを日本語に訳しただけで価値がない
i++;            // i に 1 を加算する
const list = []; // list を空の配列で初期化する
```

```javascript
// OK：「なぜ」を説明し、コードから読めない意図を補足する

// 外部 API のレート制限（100 req/min）に引っかからないよう 1 秒待機
await sleep(1000);

// キャッシュが古い場合でも安全側に倒すため、有効期限は短めに設定
// 60 秒 = 実測レスポンス遅延 ~200ms を考慮した値
const CACHE_TTL_MS = 60_000;

// TODO: 現在は線形探索（O(n)）。ユーザー数が 10 万超えたら Map に切り替える
function findUserByEmail(email, users) {
  return users.find((u) => u.email === email) ?? null;
}
```

```javascript
/**
 * 2 つのユーザー統計オブジェクトをマージして返す。
 * 同じキーが存在する場合は newStats の値で上書きする。
 * どちらの引数も変更せず、新しいオブジェクトを返す（副作用なし）。
 *
 * @param {Object} baseStats
 * @param {Object} newStats
 * @returns {Object}
 */
function mergeUserStats(baseStats, newStats) {
  return { ...baseStats, ...newStats };
}
```

```javascript
// OK：直感に反する挙動に先手を打つコメント
// Array.sort() は stable sort のため、同スコアのユーザーは元の順序を保持する
// （表示上の一貫性のために意図的にこの性質を利用している）
users.sort((a, b) => b.score - a.score);
```

```javascript
// OK：具体例（Corner Cases）を視覚的に提示するコメント
/**
 * 文字列の両端から特定の文字を削除する
 * 例: Strip(" a b ", " ") -> "a b"
 * 例: Strip("///test/", "/") -> "test"
 */
function strip(str, charToRemove) { ... }
```

---

## Part 2：ループとロジックの単純化

### 5. 制御フローを読みやすくする

#### なぜ大切か

複雑な条件分岐やループは認知負荷を高め、バグの温床になる。制御フローを整理することで「コードが何をしているか」を追いやすくなる。

#### どうするべきか

- **条件式の引数の順序を整える**：「変化するもの（変数）を左、安定したもの（定数・比較対象）を右」に書く（`if (length >= 10)` は自然、`if (10 <= length)` は読みにくい）。
- **if/else のポジティブケースを先に書く**：否定条件より肯定条件を先に書くと読みやすい。ただし注目すべき条件や短い処理が先という優先ルールもある。
- **三項演算子は単純な場合のみ**：複雑な条件や複数の分岐を三項演算子に詰め込むと読みにくくなる。
- **`do-while` は避ける**：条件がループの末尾にあると読みにくい。`while` に書き換えを検討する。
- **深いネストを避ける**：早期 `return`（ガード節）や `continue` を使って、成功ケースのネストを浅くする。

#### 良いコードの例

```javascript
// NG：深いネストで「正常系」がどこにあるか見えにくい
function processPayment(user, order) {
  if (user) {
    if (user.isActive) {
      if (order) {
        if (order.total > 0) {
          chargeCreditCard(user, order.total);
          return "success";
        } else {
          return "invalid order total";
        }
      } else {
        return "order not found";
      }
    } else {
      return "inactive user";
    }
  } else {
    return "user not found";
  }
}

// OK：ガード節（早期 return）でエラーケースを先に弾き、正常系がフラットになる
function processPayment(user, order) {
  if (!user)          return "user not found";
  if (!user.isActive) return "inactive user";
  if (!order)         return "order not found";
  if (order.total <= 0) return "invalid order total";

  chargeCreditCard(user, order.total);
  return "success";
}
```

```javascript
// NG：条件の向きが直感に反する（定数を左に置く Yoda 記法）
if (10 <= length) { ... }

// OK：変数を左、定数を右に置くと自然な英語に近い読み方ができる
if (length >= 10) { ... }
```

```javascript
// NG：三項演算子に複雑な条件を詰め込む
const label = user.score >= 100 && user.isActive && !user.isBanned
  ? "premium"
  : user.score >= 50 ? "standard" : "basic";

// OK：if/else に展開して意図を明確に
let label;
if (user.score >= 100 && user.isActive && !user.isBanned) {
  label = "premium";
} else if (user.score >= 50) {
  label = "standard";
} else {
  label = "basic";
}
```

---

### 6. 巨大な式を分割する

#### なぜ大切か

一行に詰め込まれた長い式は、一度に理解しなければならない情報量が多く、脳に負荷をかける。また、デバッグも難しくなる。

#### どうするべきか

- **説明変数を使う**：複雑な式の一部を意味のある名前の変数に抽出する。
- **要約変数を使う**：`if (request.user.id === document.ownerId)` は `const isOwner = ...` として意図を名前で表す。
- **ド・モルガンの法則を活用する**：`!(a || b)` は `!a && !b` に変換すると読みやすい場合がある。
- **短絡評価（`&&`・`||`）を乱用しない**：複雑なロジックを1行に書くより複数のステートメントに分けるほうがデバッグしやすい。

#### 良いコードの例

```javascript
// NG：1行に複数の操作が詰まっていてパース負荷が高い
if (line.split(":")[0].trim().toLowerCase() in activeUsers.map((u) => u.username.toLowerCase())) { ... }

// OK：説明変数で各ステップに名前をつける
const inputUsername   = line.split(":")[0].trim().toLowerCase();
const activeUsernames = activeUsers.map((u) => u.username.toLowerCase());
if (activeUsernames.includes(inputUsername)) { ... }
```

```javascript
// NG：条件式が長く、何を確認しているのか一読してわからない
if (request.user.id === document.ownerId || request.user.role === "admin") {
  allowEdit();
}

// OK：要約変数で「何を判定しているか」を名前で表す
const isOwner = request.user.id === document.ownerId;
const isAdmin = request.user.role === "admin";
if (isOwner || isAdmin) {
  allowEdit();
}
```

```javascript
// NG：ド・モルガン適用前（否定が入ると頭の中で変換が必要）
if (!(fileExists || hasPermission)) throw new Error("Access denied");

// OK：ド・モルガン適用後（各条件を個別に読める）
if (!fileExists && !hasPermission) throw new Error("Access denied");
```

---

### 7. 変数と読みやすさ

#### なぜ大切か

変数が多すぎると追いかける状態が増え、どこで何が変わるのかわかりにくくなる。変数のスコープと寿命を最小化することでコードの複雑度が下がる。

#### どうするべきか

- **不要な変数を削除する**：中間結果を保存するだけで後で使わない変数、コードを複雑にするだけの変数は削除する。
- **変数のスコープを縮小する**：変数は使う場所の直前で宣言し、参照できる範囲を最小限にする。`var` を避け `const` / `let` を使う。
- **変数への書き込みを減らす**：`const` を使うことで「この値は変わらない」と読み手に保証を与える。
- **グローバル変数を避ける**：どこからでも変更できる変数は追いにくくバグの原因になる。モジュールスコープを活用する。

#### 良いコードの例

```javascript
// NG：result という中間変数が不要（すぐ return するだけ）
function getUserLabel(score) {
  const result = score >= 100 ? "premium" : "standard";
  return result;
}

// OK：直接 return する
function getUserLabel(score) {
  return score >= 100 ? "premium" : "standard";
}
```

```javascript
// NG：var でスコープが関数全体に広がり、宣言と使用箇所が離れる
function processUsers(users) {
  var errorMessages = []; // ← 関数の先頭で宣言
  var validUsers = [];
  users.forEach((user) => {
    validUsers.push(user);
    if (!user.isActive) {
      errorMessages.push(`${user.name} is inactive`); // ← ずっと下で初めて使う
    }
  });
  return { validUsers, errorMessages };
}

// OK：const で宣言し、使う直前にスコープを絞る
function processUsers(users) {
  const validUsers    = users.filter((u) => u.isActive);
  const errorMessages = users
    .filter((u) => !u.isActive)
    .map((u) => `${u.name} is inactive`);
  return { validUsers, errorMessages };
}
```

```javascript
// NG：ループ内で再代入が繰り返される変数は状態を追いにくい
let total = 0;
for (const item of orderItems) {
  total = total + item.price; // total が何度も書き換えられる
}

// OK：reduce() を使って書き換えをなくす（変数が事実上イミュータブルになる）
const total = orderItems.reduce((sum, item) => sum + item.price, 0);
```

---

## Part 3：コードの再構成

### 8. 無関係の下位問題を抽出する

#### なぜ大切か

一つの関数が本来の目的とは無関係な細部処理（文字列フォーマット、バリデーション、データ変換など）を抱えていると、主要なロジックが見えにくくなる。

#### どうするべきか

- **「この関数の高レベルな目標は何か？」を問い続ける**：各コードブロックが目標と直接関係しているか確認する。
- **無関係な処理を関数に抽出する**：たとえば「ユーザーを保存する」関数の中でメールアドレスのバリデーションをしているなら、`validateEmail()` として分離する。
- **汎用コードを作る**：特定のプロジェクトに依存しない処理（配列操作、フォーマット処理など）は汎用ユーティリティ関数として切り出す。
- **複雑なAPIを隠蔽する「ラッパー関数」を作る**：外部ライブラリなどの扱いにくいインターフェースが散在すると見通しが悪くなる。自プロジェクト用に使いやすく包み直したラッパー関数（Facadeパターン）を作成し、外部依存の複雑さを隔離する。

#### 良いコードの例

```javascript
// NG：「ユーザーを登録する」関数が、バリデーション・ハッシュ化・保存を全部抱えている
async function registerUser(email, password) {
  // バリデーション（下位問題）
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) throw new Error("Invalid email");
  if (password.length < 8) throw new Error("Password too short");

  // パスワードハッシュ化（下位問題）
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hashBuffer = await crypto.subtle.digest("SHA-256", data);
  const passwordHash = Array.from(new Uint8Array(hashBuffer))
    .map((b) => b.toString(16).padStart(2, "0"))
    .join("");

  // 保存（本来の目的）
  await db.save({ email, passwordHash });
}
```

```javascript
// OK：下位問題をそれぞれ専用関数に切り出し、本筋が明確になる
function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

async function hashPassword(plaintext) {
  const data = new TextEncoder().encode(plaintext);
  const buffer = await crypto.subtle.digest("SHA-256", data);
  return Array.from(new Uint8Array(buffer))
    .map((b) => b.toString(16).padStart(2, "0"))
    .join("");
}

async function registerUser(email, password) {
  if (!isValidEmail(email)) throw new Error("Invalid email");
  if (password.length < 8) throw new Error("Password too short");

  const passwordHash = await hashPassword(password);
  await db.save({ email, passwordHash });
  // → 関数の本筋（登録フロー）だけが残り、何をしているか一読できる
}
```

---

### 9. 一度に一つのことをする

#### なぜ大切か

複数のタスクを同時に行う関数やコードブロックは理解しにくく、個別にテストできない。タスクを分離することで各ブロックの責務が明確になる。

#### どうするべきか

- **関数が行うタスクを列挙する**：複数のタスクが混在している場合、それぞれを別の関数・コードブロックに分ける。
- **処理の「フェーズ」を分ける**：読み込み → 変換 → 出力のように処理段階を明確に分離する。
- **「データの抽出」と「計算・処理」のフェーズを分離する**：複雑なオブジェクトから値を取り出すフェーズと、その値を使ってロジックを回すフェーズを分ける。DBからの取得と条件判定が複雑に絡み合うような状態を避ける。
- **1関数1目的を守る**：ユーザー検索と同時にキャッシュ更新も行うような「副作用の多い関数」を避ける。

#### 良いコードの例

```javascript
// NG：1つの関数が「集計」「勝者判定」「表示」を同時に行っている
function tallyAndDisplayVotes(votes) {
  const counts = {};
  for (const vote of votes) {
    counts[vote] = (counts[vote] ?? 0) + 1;
  }
  const winner = Object.keys(counts).reduce((a, b) => (counts[a] > counts[b] ? a : b));
  console.log(`Winner: ${winner} with ${counts[winner]} votes`);
  console.log("All results:", counts);
}
```

```javascript
// OK：各タスクを独立した関数に分割する

/** 票を集計してカウントオブジェクトを返す */
function countVotes(votes) {
  return votes.reduce((counts, vote) => {
    counts[vote] = (counts[vote] ?? 0) + 1;
    return counts;
  }, {});
}

/** 最多得票者を返す */
function findWinner(counts) {
  return Object.keys(counts).reduce((a, b) => (counts[a] > counts[b] ? a : b));
}

/** 集計結果を表示する */
function displayResults(counts, winner) {
  console.log(`Winner: ${winner} with ${counts[winner]} votes`);
  console.log("All results:", counts);
}

// 呼び出し側：各フェーズが一行で読める
const counts = countVotes(votes);
const winner = findWinner(counts);
displayResults(counts, winner);
```

```javascript
// NG：データの抽出（検索）と処理（更新）が混在している
function updateActiveUserScores(users) {
  for (const user of users) {
    if (user.isActive && user.lastLogin > getOneMonthAgo()) {
      user.score += 10;
      saveToDb(user);
    }
  }
}

// OK：フェーズを分離し、抽出と処理を1つのタスクずつ行う
function updateActiveUserScores(users) {
  // タスク1: 処理対象のデータを抽出する
  const activeUsers = users.filter(u => u.isActive && u.lastLogin > getOneMonthAgo());

  // タスク2: 抽出したデータに対して計算・処理を行う
  for (const user of activeUsers) {
    user.score += 10;
    saveToDb(user);
  }
}
```

---

### 10. 思いを言葉にする（コードを説明してから書く）

#### なぜ大切か

「このコードは何をしているか」をシンプルな日本語（自然言語）で説明できないなら、コード自体も整理されていない可能性が高い。説明できる状態になって初めて、シンプルな実装が見えてくる。

#### どうするべきか

- **コードを書く前に処理を言葉で説明する**：ゴム製のアヒル（ラバーダック・デバッグ）に向かって説明するように、処理ステップを言語化する。
- **言葉の中のキーワードをコードに反映する**：説明の中で使った動詞や名詞をそのまま関数名・変数名に使う。
- **複雑に感じたらシンプルな言葉で言い直す**：言い直した説明がシンプルなら、コードもシンプルに書けるはず。
- **言葉の構造をそのままコードに落とす**：言い直した自然言語の構造がシンプルなら、そのまま `if` 文の構成や関数の分割単位として採用する。

#### 良いコードの例

```javascript
// 「管理者でなく、かつ自分のドキュメントでもなければ閲覧を拒否する」と言葉にする
// → 言葉の構造をそのままコードに落とすと、ガード節が自然に生まれる

// NG：ネストと否定が重なって読みにくい
function canViewDocument(user, document) {
  if (user.role === "admin" || user.id === document.ownerId) {
    if (!document.isDeleted) {
      return true;
    }
  }
  return false;
}

// OK：言葉にした説明をそのままコードの構造に写す
// 「削除済みなら拒否」「管理者または所有者なら許可、それ以外は拒否」
function canViewDocument(user, document) {
  if (document.isDeleted) return false;

  const isAdmin = user.role === "admin";
  const isOwner = user.id === document.ownerId;
  return isAdmin || isOwner;
}
```

```javascript
// 言語化 → 命名に反映する例
// 「アクティブなユーザーのうち、先月ログインしたユーザーを返す」と説明できるなら
// 関数名は getRecentlyActiveUsers、変数名は lastMonthThreshold が自然に決まる

function getRecentlyActiveUsers(users) {
  const lastMonthThreshold = Date.now() - 30 * 24 * 60 * 60 * 1000;
  return users.filter(
    (u) => u.isActive && u.lastLoginAt >= lastMonthThreshold
  );
}
```

---

### 11. 少ないコードを書く

#### なぜ大切か

コードは書いた量だけ読まれ、維持されなければならない。不要なコードは複雑さを増やし、バグの可能性を高める。最も読みやすいコードは「存在しないコード」である。

#### どうするべきか

- **要件を疑う**：「本当にこの機能が必要か」を問い、不要な機能は実装しない（YAGNI 原則）。
- **標準ライブラリを活用する**：車輪の再発明をしない。言語が提供している機能（`Array.prototype` メソッドなど）を積極的に使う。
- **コードを定期的に削除する**：使われていないコード（デッドコード）は無慈悲に削除する。コメントアウトで残すと混乱の元になる。
- **問題を分割して小さく解く**：大きな問題を小さな問題に分解し、各部分を最小のコードで解く。

#### 良いコードの例

```javascript
// NG：標準 API で済む処理を自前実装している
function findMax(numbers) {
  let maxVal = numbers[0];
  for (const n of numbers) {
    if (n > maxVal) maxVal = n;
  }
  return maxVal;
}

function flatten(nestedArray) {
  const result = [];
  for (const subArray of nestedArray) {
    for (const item of subArray) {
      result.push(item);
    }
  }
  return result;
}

// OK：組み込みメソッドを使う（可読性も上がる）
const maxVal   = Math.max(...numbers);
const flatList = nestedArray.flat();
```

```javascript
// NG：コメントアウトされたデッドコードが残っている
function calculateDiscount(price, user) {
  // if (user.tier === "gold") return price * 0.8;
  // else if (user.tier === "silver") return price * 0.9;
  if (user.isPremium) return price * 0.85;
  return price;
}

// OK：使われていないコードは削除する（Git 履歴に残るので消して問題ない）
function calculateDiscount(price, user) {
  if (user.isPremium) return price * 0.85;
  return price;
}
```

```javascript
// NG：不要な抽象化（YAGNI 違反）
function add(a, b) {
  return a + b;
}
const total = add(price, tax); // 単純な足し算に関数は不要

// OK：シンプルに書く
const total = price + tax;
```

---

## Part 4：テストと可読性

### 12. テストと読みやすさ

#### なぜ大切か

テストコードはプロダクションコードと同様に読まれる「最初のユーザー」であり、生きたドキュメントである。読みにくいテストはメンテナンスされなくなり、テストの信頼性が低下する。また、**「テストが書きにくいコード」は、本番コードの設計（強すぎる結合など）が悪いというアーキテクチャの明確なサイン（評価指標）** でもある。

#### どうするべきか

- **テストを簡潔・明確にする**：テストコードも同じ読みやすさの原則を適用する。
- **テスト入力は最小限にする**：テストしたいことに関係のない入力値は省略し、テストの意図を明確にする。
- **エラーメッセージを読みやすくする**：テスト失敗時のメッセージから何がどう失敗したのかが即座にわかるようにする。
- **テスト名に何をテストしているか書く**：名前自体がテストケースの仕様書となるようにする（`test login` より `ログイン失敗時に false を返す` のほうが内容が一目でわかる）。
- **実装とテストを分離する**：テストを書きやすくするため、コードをテスタブルに設計する（副作用の分離、依存の注入など）。
- **「カスタムアサーション（ミニ言語）」を作成する**：テストコード特有のボイラープレート（セットアップ処理などの重複）を減らすため、1行で「入力と期待値」の対応関係だけを表現できるヘルパー関数を作る。

#### 良いコードの例

```javascript
// NG：テスト名が曖昧で、失敗したときに何が壊れたか即座にわからない
//     テスト入力に関係のない属性が多く、意図が霞む
test("login", () => {
  const user = {
    id: 1, name: "Alice", email: "alice@example.com",
    role: "admin", createdAt: "2020-01-01", isActive: true,
    passwordHash: hashPassword("correct_password"),
  };
  expect(login(user, "wrongpassword")).toBe(false);
});

// OK：テスト名が「何を・どんな条件で・どうなるか」を表している
//     テスト入力もテストに必要な最小限の属性だけを持つ
test("誤ったパスワードでログインすると false を返す", () => {
  const user = createTestUser({ password: "correct_password" });
  expect(login(user, "wrong_password")).toBe(false);
});
```

```javascript
// NG：失敗したとき「何が」「どう違ったか」がわからない
test("label", () => {
  expect(getUserLabel(150)).toBe("premium");
});

// OK：テスト名と失敗メッセージから原因が即座にわかる
test("スコアが 100 以上のとき premium を返す", () => {
  const score    = 150;
  const expected = "premium";
  const actual   = getUserLabel(score);
  expect(actual).toBe(expected); // Jest は差分を自動表示するため詳細なメッセージが出る
});
```

```javascript
// NG：テスト対象が外部サービスに直接依存していてテストで実メール送信が走る
function sendWelcomeEmail(user) {
  nodemailer.createTransport({ host: "smtp.example.com" })
    .sendMail({ to: user.email, subject: "Welcome!" });
}

// OK：mailer を引数で受け取ることでテスト時にモックに差し替えられる
function sendWelcomeEmail(user, mailer = defaultMailer) {
  mailer.send({ to: user.email, subject: "Welcome!" });
}

test("正しいアドレスにメールを送る", () => {
  const mockMailer = { send: jest.fn() };
  const user = createTestUser({ email: "test@example.com" });

  sendWelcomeEmail(user, mockMailer);

  expect(mockMailer.send).toHaveBeenCalledWith(
    expect.objectContaining({ to: "test@example.com" })
  );
});
```

```javascript
// NG：セットアップやアサーションの記述が長く、何と何を検証しているか見えにくい
test("スコア計算のテスト", () => {
  const user1 = { id: 1, history: [10, 20] };
  expect(calculateScore(user1)).toBe(30);

  const user2 = { id: 2, history: [0, -5, 10] };
  expect(calculateScore(user2)).toBe(5);
});

// OK：カスタムアサーションを作り、検証ロジックを「ミニ言語」化する
function checkScore(historyArray, expectedScore) {
  const user = { id: 99, history: historyArray };
  expect(calculateScore(user)).toBe(expectedScore);
}

test("スコア計算の境界値テスト", () => {
  // 入力と期待値の対応関係だけが視覚的に浮き彫りになる
  checkScore([10, 20],    30);
  checkScore([0, -5, 10],  5);
  checkScore([],           0);
});
```

---

## Part 5：全体設計（アーキテクチャ）への昇華

### 13. データ構造とクラス設計の進化（ケーススタディ）

ミクロなコード行の改善を積み重ねた先には、クラスやシステム全体の「関心事の分離（Separation of Concerns）」というマクロなアーキテクチャの改善が存在する。

**例：「過去1分間 / 過去1時間のイベント数をカウントする機能」の設計プロセス**

1. **素朴な実装（アンチパターン）**
   イベントのタイムスタンプを1つのリストにすべて保存し、リクエストのたびにループして集計する。
   → *結果：すぐにメモリが溢れ、パフォーマンスが悪化する。*

2. **メモリ管理の混入（タスクの断片化）**
   リストに追記する際、古いタイムスタンプを削除するロジックを同じクラスに追加する。
   → *結果：「データの追記」「古いデータの破棄」「分と時間の集計計算」という複数のタスクが1つのクラス内に強く結合し、可読性と保守性が崩壊する。*

3. **汎用データ構造とビジネスロジックの分離（理想的な設計）**
   時間を区間（バケツ）で管理し、シフト操作とデータ保持のみを行う「純粋な汎用キュー（例: `ConveyorQueue`）」クラスを作成する（**無関係の下位問題の抽出**）。そのキューを内部に保持し、1分・1時間ごとの集計ロジックのみを担うビジネスクラス（例: `TrailingBucketCounter`）を別に作成する。
   → *結果：時間管理の複雑なアルゴリズムと、イベント集計という要件が完全に独立し、後任者が安全に理解し変更できる堅牢なアーキテクチャとなる。*

---

## 観点一覧まとめ

| カテゴリ | 観点 | 一言まとめ |
| :--- | :--- | :--- |
| 前提 | 行数 ≠ 可読性 | 物理的な短さより、認知負荷（解読時間）の短さを優先する |
| 命名 | 情報を詰め込む | 名前だけで意味・単位・状態を伝える |
| 命名 | 誤解されない | 境界・bool・方向性を正確に表現する |
| 書式 | 一貫した整形 | 視覚的なパターンでコード構造を伝える |
| 書式 | 視覚的なヘルパー関数 | 見た目の不規則性を排除するためだけにラップする |
| コメント | 書くべき内容 | 「なぜ」「驚き」「欠陥」を補足する |
| コメント | 書くべきでない内容 | コードの繰り返しにならない |
| コメント | 具体例の提示 | `Strip(" a b ") -> "a b"` のように入出力を視覚化する |
| コメント | 専門用語の活用 | 情報密度の高い単語（キャッシュ等）で説明を圧縮する |
| 制御フロー | 条件分岐の整理 | ガード節・順序・ネスト深度を最適化 |
| ロジック | 式の分割 | 説明変数・要約変数で複雑さを分解 |
| 変数 | スコープと寿命 | 変数は狭く・短く・変更少なく（状態の最小化） |
| 構造 | 下位問題の抽出 | 本筋と無関係な処理を別関数に切り出す |
| 構造 | ラッパー関数の作成 | 複雑な外部APIを隠蔽・隔離しインターフェースを成形する |
| 構造 | 1タスク1関数 | 責務を明確に分離する |
| 構造 | フェーズの分離 | オブジェクトの「抽出」と「計算・処理」を明確に分ける |
| 設計 | 言語化してから書く | シンプルに説明できるまで設計を練る |
| 量 | コードを減らす | 書かないコードが最も読みやすい（不要要件の削除） |
| テスト | テストの読みやすさ | テストコードにも同じ原則を適用する |
| テスト | カスタムアサーション | 入力と期待値を1行で表現するヘルパー言語を作る |
| テスト | アーキテクチャの指標 | テストのしにくさは結合度が強いサインと捉える |
| 全体設計 | 汎用構造とロジックの分離 | クラスやデータ構造を単一責任に分割しアーキテクチャへ昇華する |

---

*Based on: Dustin Boswell & Trevor Foucher, "The Art of Readable Code" (O'Reilly, 2011)*
