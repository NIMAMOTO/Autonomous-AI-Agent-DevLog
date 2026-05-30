# 🐍 Python × LangChain × LLM 学習ノート

> **目的：** AIエージェントの「手足（Tool）」を自分で作れるようになる  
> **対象コード：** 企業情報を返す関数 → LangChain Toolへのアップグレード

---

## 📦 Step 0｜このコードは何のため？

| 質問 | 答え |
|---|---|
| 最終目標 | AIエージェントに「就活サイトから企業情報を取ってこい」と命令できる「手足（Tool）」を作る |
| 今やっていること | その基礎練習。外部サイトの代わりに「名前を入れたらダミー情報を返す関数」を作る |

---

## 🔑 概念ノート

### 1｜関数（Function）とは？

**自動販売機のイメージ**

```
入れるもの（引数） → [　自動販売機の中身（処理）　] → 出てくるもの（戻り値）
company_name      →  情報をdictにまとめる        →  完成したdict
```

- **作る** → `def` キーワードを使う
- **動かす（呼び出す）** → 関数名の後ろに `()` をつける
- `()` をつけないと、機械そのもの（設計図）が返ってくる ⚠️

---

### 2｜`def` の行を解剖する

```python
def get_company_info(company_name: str) -> dict:
```

| パーツ | 役割 | 例 |
|---|---|---|
| `def` | 「今から関数を作るぞ」という宣言 | definitionの略 |
| `get_company_info` | 関数の**名前**（自由に決めていい） | `make_profile`, `calc_salary` など |
| `(company_name: str)` | **入り口のルール**：`company_name`という名前で文字列(str)を受け取る | |
| `-> dict:` | **出口のルール**：辞書(dict)の形で返す | |
| `:` | 絶対に忘れてはいけない！忘れるとエラー ⚠️ | |

> 💡 **大事な理解：`def` の行は「外枠」を作るだけ**  
> なぜ文字列が辞書に変換できるかは、`def` は知らない。  
> 変換するのは、箱の「中」に書かれたコード（処理部分）の仕事。  
> → これを「インターフェース（外枠）」と「実装（中身）」の分離という。

---

### 3｜辞書（dict）の作り方

```python
dummy_data = {
    "name": "syokoen",   # "見出し（キー）": "中身（値）"
    "age": "23",
    "hobby": "ship"
}
```

| ルール | 内容 |
|---|---|
| `{ }` で囲む | 「これはdictです」という目印 |
| `"キー": "値"` | ラベル付きでデータを整理する |
| `,` で区切る | 項目と項目の間にカンマ。最後はなくてもOK |

> 💡 **なぜdictを使うのか？**  
> LangChainのエージェントが一番好きなデータ形式だから。  
> 複数の情報をラベル付きでひとまとめにできる。

---

### 4｜`return` の役割

```python
return dummy_data
```

- 「この結果を外の世界に届けて！」という**配達命令**
- `return` がないと、データは関数の中に閉じ込められたまま**消滅**する ⚠️

---

### 5｜インデント（字下げ）のルール

```python
def profile(aaa: str) -> dict:
    dummy_data = { ... }  # ← スペース4つ分、右にズレている
    return dummy_data     # ← これも同じズレ幅
```

- `def` の下の行は必ず**スペース4つ分**右にズラす
- Pythonは「このズレている部分が、関数のグループ」と判断する
- ズレが狂うとエラー ⚠️

---

## 🔗 Python × LangChain × LLM の関係

### 3者の役割

| 役割 | 例え | 何をするか |
|---|---|---|
| **Python Script** | マザーボード・OS | 全体の土台。「いつ何を動かすか」という流れを制御する |
| **LLM（GPT, Geminiなど）** | クラウド上の超高性能CPU | テキストを読んで「次にどのツールを使うべきか」を推論する。クラウド上に存在し、Pythonの中に入ってくるわけではない |
| **LangChain** | 翻訳機・配線 | ローカルのPythonとクラウドのLLMを繋ぐ |

### LangChainがない世界 vs ある世界

**LangChainがない場合（手動でやること）：**
1. ツールの説明書をLLMが理解できるJSON形式に手動変換
2. APIサーバーにHTTPリクエストを飛ばす
3. LLMが返した文字列を解析して対応する関数を探し出す
4. 実行する

**LangChainがある場合：**
→ 上記をすべて**全自動**でやってくれる

> 💡 **「AIがPython Scriptの判断の一部になっている」という理解は正しい**  
> LLMは「Pythonスクリプトに組み込まれた、高度な条件分岐（if文）の代わり」として機能する。  
> Pythonが処理できない「曖昧な判断」が必要な時だけ、LangChain経由でLLMに投げる。

---

### ReActループとは何か？

**Q：LangChainはただの配線なのに、なぜ「思考のループ」が生まれる？**

→ ループの正体は、**LangChainの内部にあらかじめ書かれたPythonのループ構文（`for`文）**。  
　 蔣さんが `app.invoke()` を呼んだ瞬間、このループのスイッチが入る。

**ReActの流れ：**

```
思考（LLM）→ 行動（どのToolを使うか決定）→ ローカル実行（PythonがToolを動かす）
    ↑                                                         |
    └─────────── 観察結果をhistoryに繋いでLLMに再送 ←──────────┘
                        ↓
                  「最終回答」が出たらループ終了
```

**LangChain内部の擬似コード（イメージ）：**

```python
# ⚠️ これはLangChainの内部で動いているイメージコード（実際のコードではない）
def create_agent_内部動作(llm, tools, user_input):
    history = [user_input]  # 会話履歴を鎖のように繋いでいく

    for i in range(max_iterations):  # ← ループの主導権は100%Python側にある
        llm_response = llm.invoke(history)  # LLMに「次どうする？」と聞く

        if "最終回答" in llm_response:
            return llm_response  # 完了したらループを抜ける

        # LLMが「このToolを使え」と言ってきたら
        observation = execute_tool(llm_response.tool_to_use, llm_response.tool_input)

        history.append(observation)  # 結果をhistoryに追加して次のループへ
```

**重要な理解：**

| 問い | 答え |
|---|---|
| ReActループはどこにある？ | LangChainというPython Scriptの内部に内蔵されている |
| ループの主導権はAI側？ | **No。100%ローカルで動くLangChain（Python）側** |
| LLMはなぜ記憶できないのにループできる？ | LangChainが毎回`history`（鎖）を繋いでLLMに送り直しているから |
| なぜ「Chain（鎖）」という名前？ | 思考と観察結果をテキストとして鎖のように繋いでいくから |

> 💡 **ループが「見えなかった」理由**  
> 簡単な問題（文字数を数えるなど）はループが2周で終わるため一瞬で完了する。  
> 複雑な条件分岐が必要な問題では、このループが何度も回り続ける。

---

## 🛠️ コードノート

### 基本の関数（LangChainなし）

```python
def profile(aaa: str) -> dict:
    dummy_data = {
        "name": aaa,      # 引数を動的に使う
        "age": "23",
        "hobby": "ship"
    }
    return dummy_data

print("---kekka---")
print(profile("tesuo"))   # () をつけて呼び出す
```

**出力：**
```
---kekka---
{'name': 'tesuo', 'age': '23', 'hobby': 'ship'}
```

---

### LangChain Tool へのアップグレード

```python
from langchain_core.tools import tool  # LangChainを呼び込む

@tool  # ← デコレータ：「この関数をAIの手足にする」という宣言
def profile(aaa: str) -> dict:
    """名前（aaa）を入力すると、プロフィールを辞書形式で返すツールです。"""
    # ↑ ドックストリング：AIへのトリセツ。これがないとエラーになる！
    dummy_data = {
        "name": aaa,
        "age": "23",
        "hobby": "ship"
    }
    return dummy_data

print("---kekka---")
print(profile.invoke({"aaa": "tesuo"}))  # Tool化したら .invoke() で呼び出す
```

**普通の関数との違い：**

| | 普通の関数 | LangChain Tool |
|---|---|---|
| 宣言 | `def` だけ | `@tool` + `def` |
| 説明文 | 任意 | **必須**（AIが読むため） |
| 呼び出し方 | `profile("tesuo")` | `profile.invoke({"aaa": "tesuo"})` |
| 誰が使うか | Pythonのコード | AIエージェント |

---

## ⚠️ よくあるエラーと対処法

| エラー | 原因 | 対処 |
|---|---|---|
| `<function profile at 0x...>` が表示される | `()` をつけずに関数名だけ書いた | `profile("tesuo")` のように `()` をつける |
| `ValueError: Function must have a docstring` | `@tool` を使ったのにdocstringがない | `def` の直下に `"""説明文"""` を追加する |
| `IndentationError` | インデントがズレている | スペース4つ分に統一する |

---

## 📝 自分の言葉でのまとめ

- `def` の行は「箱の外枠（入り口と出口）」を作るだけ。中身の処理は関係ない。
- `dict` はラベル付きで複数のデータをまとめる形式。LangChainと相性が良い。
- `return` がないと結果が消える。
- LLMはPythonの「中」に入ってくるわけではなく、クラウド上に独立して存在する。
- LangChainは「Pythonとクラウド上のLLMを繋ぐ配線」。
- `@tool` + `"""docstring"""` で、普通のPython関数がAIの手足になる。
- ReActのループはLangChain（Python）の内部コード。主導権はAI側ではなくPython側にある。
- LLMは本来記憶を持てない（ステートレス）。LangChainが`history`を鎖のように繋いで毎回送り直すことで、ループが成立している。
