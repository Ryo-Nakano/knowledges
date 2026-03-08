# LLM の Function Calling（関数呼び出し）

## 概要

Function Calling は、LLM が**外部のツールや関数を呼び出す能力**を持つ仕組み。
LLM 単体では「テキスト生成」しかできないが、Function Calling を使うと現実世界のデータや処理と連携できる。

---

## 登場人物

```
ユーザー  ←→  アプリケーション  ←→  LLM API
                    ↓
              外部ツール・関数
              （天気API、DBなど）
```

| 役割 | 説明 |
|---|---|
| ユーザー | 質問・指示を出す人 |
| アプリケーション | ユーザーと LLM の中間層。ツールの定義・実行を担当 |
| LLM API | 推論・テキスト生成を担当。ツールの実行はしない |

---

## 構造の理解：ユーザーは LLM と直接会話していない

ユーザーは LLM と**直接**やりとりしているのではなく、**アプリケーションを介して**やりとりしている。

OS の構造に例えると：

```
OS:  ユーザー ←→ シェル ←→ カーネル
LLM: ユーザー ←→ アプリケーション ←→ LLM
```

ただし OS と異なる点として、OS ではカーネルが実処理をすべて担うが、
LLM の場合は**推論は LLM**、**ツール実行はアプリ**と役割が両側に分かれている。

---

## なぜ Function Calling が必要か

LLM 単体の限界：
- 学習データのカットオフ以降の情報は知らない
- リアルタイムデータ（天気・株価など）を取得できない
- 外部システム（DB・API）を操作できない

Function Calling でこれらを解決する。

---

## 仕組み（処理フロー）

```
アプリ「ツール一覧と一緒にユーザーの質問を渡す」
  ↓
LLM「渡されたツール一覧を見て、必要なものを選んで JSON を返す」
  ↓
アプリ「ツールを実行して結果を LLM に返す」
  ↓
LLM「結果を使って最終回答を生成」
  ↓
アプリ「ユーザーに届ける」
```

### 重要なポイント

- **ツール一覧は LLM が持っているのではなく、アプリがリクエスト時に毎回渡す**
- **LLM はツールの実行をしない**。「どの関数を・どの引数で呼ぶか」を JSON で返すだけ
- **ツールの実際の実行はアプリケーション側が行う**

---

## 具体例

### ツール定義（アプリ → LLM に渡す）

```json
{
  "name": "get_weather",
  "description": "指定した都市の現在の天気を取得する",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {
        "type": "string",
        "description": "都市名（例: Tokyo）"
      }
    },
    "required": ["city"]
  }
}
```

### LLM の応答（テキストではなく関数呼び出し指示）

ユーザーが「東京の天気は？」と聞くと：

```json
{
  "function_call": {
    "name": "get_weather",
    "arguments": { "city": "Tokyo" }
  }
}
```

### コードで見るアプリ側の処理

```python
# 1. ユーザーの入力をツール定義と一緒に LLM に送る
response = llm.chat("東京の天気は？", tools=[get_weather_tool])

# 2. LLM が「関数を呼んで」と返してきた場合
if response.function_call:
    func_name = response.function_call.name       # "get_weather"
    args = response.function_call.arguments       # {"city": "Tokyo"}

    # 3. アプリ自身が関数を実行する
    result = get_weather(city=args["city"])

    # 4. 結果を LLM に送り返す
    final_response = llm.chat(tool_result=result)

# 5. ユーザーに表示
print(final_response.text)
```

---

## 身近な例：Antigravity 自体もこの構造

```
あなた（ユーザー）
  ↓
Antigravity アプリケーション
  ├── Gemini LLM（推論・回答生成）
  └── ツール実行（view_file, run_command, browser_subagent...）
```

「ファイルを見て」と言ったとき：
1. LLM が「`view_file` を呼べ」と JSON で返す
2. アプリが実際にファイルを読む
3. 結果を LLM に渡す
4. LLM が回答を生成してユーザーへ届ける

---

## 主要な LLM での呼称

| プロバイダー | 呼び方 |
|---|---|
| OpenAI | Function Calling / Tool Calling |
| Google Gemini | Function Calling |
| Anthropic Claude | Tool Use |
