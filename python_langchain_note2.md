# 🤖 Step 5｜全体の訓練ノート
## VS Code + Ollama + LangChain エージェント完全構築

> **前提：** Step 1〜4（関数・dict・Tool化・ReActループ）を理解済みの上で、  
> すべてを統合した「自律型エージェント」をローカル環境で動かす。

---

## 📦 今日作ったもの：全体像

```
ユーザーの質問（query）
        ↓
  app.stream() でエージェント起動
        ↓
  LLM（Gemma 4 26B）が思考
  「profileツールを使えばいい」と判断
        ↓
  Pythonがprofile関数をローカルで実行
        ↓
  結果（dict）をLLMに返す
        ↓
  LLMが自然言語に翻訳して最終回答
```

---

## 🗂️ マスターコード（main.py）

```python
from langchain_ollama import ChatOllama
from langchain_core.tools import tool
from langchain.agents import create_agent

# 1. ローカルLLM（脳）の定義
llm = ChatOllama(model="gemma4:26b", temperature=0)

# 2. 手足（Tool）の定義
@tool
def profile(aaa: str) -> dict:
    """名前（aaa）を入力すると、その人のプロフィール（年齢と趣味）を辞書形式で返すツールです。"""
    dummy_data = {
        "name": aaa,
        "age": "23",
        "hobby": "ship"
    }
    return dummy_data

tools = [profile]

# 3. エージェントの組み立て
app = create_agent(
    model=llm,
    tools=tools,
    system_prompt="あなたは実直なデータアシスタントです。ユーザーの質問に答えるために必要なツールがあれば、必ず使ってください。"
)

# 4. 実行とプロセスの可視化
query = "tesuoの趣味は何ですか？プロファイル情報を調べて答えてください。"

for step in app.stream({"messages": [("user", query)]}):
    print(step)
    print("--------------------------------------------------")

print("稼働終了\n--- Local Agent実行終了 ---")  # ← ループの外に置く
```

---

## 🔑 概念ノート

### 1｜Step 4のコードとStep 5のコードの違い

| | Step 4（Tool単体テスト） | Step 5（エージェント統合） |
|---|---|---|
| LLMを使うか | **使わない**。蔣さんが手動でToolを動かした | **使う**。LLMが自分でToolを選んで動かす |
| 実行方法 | `profile.invoke({"aaa": "tesuo"})` | `app.stream({"messages": [...]})` |
| 何をしているか | 手足を直接手動で動かしただけ | 脳に曖昧な指示を出して、脳が手足を動かす |
| コードの長さ | 短い | LLM定義・エージェント組み立てが加わり長くなる |

> 💡 **コードが長くなった理由**  
> Step 5で初めて「APIキー設定」「LLMの召喚」「create_agentによる脳と手足のドッキング」が加わったから。

---

### 2｜インポートの3行を解剖する

```python
from langchain_core.tools import tool       # ①
from langchain.agents import create_agent   # ②
from langchain_ollama import ChatOllama     # ③
```

| 行 | 何を取ってくるか | なぜ必要か |
|---|---|---|
| ① | `@tool` デコレータ | 普通の関数をLLMが使える手足に変えるため |
| ② | `create_agent` 関数 | 脳と手足を繋いでReActループを内蔵したエージェントを組み立てるため |
| ③ | `ChatOllama` クラス | ローカルのOllamaとPythonスクリプトを繋ぐプラグ |

---

### 3｜`llm = ChatOllama(model="gemma4:26b", temperature=0)` を解剖する

| パーツ | 役割 |
|---|---|
| `llm =` | 作ったAIの実体に名前をつけて変数に保存。後で `create_agent` に渡す |
| `model="gemma4:26b"` | Ollamaの中のどのモデルを使うかをピンポイントで指定 |
| `temperature=0` | AIの思考のランダム性をゼロに固定。常に論理的で確定的な答えを返す |

**temperatureについて：**

| 値 | 特性 | 用途 |
|---|---|---|
| `0` | 確定的・機械的。同じ入力→常に同じ出力 | エージェント開発、データ処理（**こちらを使う**） |
| `0.7〜1.0` | ランダム・創造的。答えが毎回変わる | 小説執筆、アイデア出し |

> ⚠️ **エージェント開発では `temperature=0` が絶対の鉄則**  
> 値が高いと、AIがツールの書式を崩したり存在しないツールを作り出したりして、LangChainのループがクラッシュする。  
> Macの計算負荷は temperature の値に関係なく変わらない（推論の計算量は同じ）。

---

### 4｜`app = create_agent(...)` を解剖する

```python
app = create_agent(
    model=llm,        # 頭（推論エンジン）
    tools=tools,      # 手足（使える機能のリスト）
    system_prompt="..." # 性格・絶対命令
)
```

| パラメータ | 役割 | 補足 |
|---|---|---|
| `model=llm` | 思考力の天井を決める。モデルの変更はここだけ差し替えればいい | |
| `tools=tools` | エージェントが「自分にはこの機能が使える」と認識する手足のリスト | ツールが増えたらリストに追記するだけ |
| `system_prompt` | 「性格」だけでなく「絶対的な業務命令」。ローカルLLMはツールを使わずにサボる傾向があるため、強制的にツールを使わせる制御装置として重要 | |

> 💡 **`app` という名前の意味**  
> 完成したエージェント（自律型ロボット）に付けた変数名。以降は `app.invoke()` や `app.stream()` で指示を出す。

---

### 5｜`tools = [profile]` の役割

デコレータの締めの合図ではない。

`create_agent` は「手足のリスト」を要求する仕様になっているため、`[ ]` でリスト化して渡す。ツールが複数になったときはリストに追加するだけ。

```python
# ツールが1つの場合
tools = [profile]

# ツールが複数になった場合（将来）
tools = [profile, web_search, calculator]
```

---

### 6｜`invoke` と `stream` の違い

どちらも「エージェントを起動するボタン」だが、データの返し方が違う。

| | `invoke` | `stream` |
|---|---|---|
| 動作 | 全処理が終わるまで待って、最終結果だけ一括で返す | 1周思考するたびに途中経過をリアルタイムで小出しに返す |
| 用途 | 最終結果だけほしい時 | 裏側のReActループ（思考・行動・観察）を可視化したい時 |

> 💡 **LangChainの統一ルール**  
> `invoke` はLangChainのすべての部品で使える共通の起動ボタン名。  
> ツール単体 → `profile.invoke(...)` / エージェント全体 → `app.invoke(...)`

---

### 7｜`for step in app.stream(...)` を解剖する

**`for 変数 in 集まり:` の基本ルール：**  
データの集まりから中身を1つずつ順番に取り出して、変数に入れながら繰り返す。

```python
for step in app.stream({"messages": [("user", query)]}):
    print(step)       # ← インデントあり = ループの中で毎回実行

print("稼働終了")     # ← インデントなし = ループが完全に終わった後に1回だけ実行
```

**流れのイメージ：**
```
app.stream() → コンベアベルトを生成
                    ↓
  エージェント1周目の思考データ → stepに入る → print(step)
  エージェント2周目の行動データ → stepに入る → print(step)
  エージェント3周目の最終回答  → stepに入る → print(step)
                    ↓
  ベルトが止まる（全処理完了）
                    ↓
  print("稼働終了") が1回だけ実行される
```

> ⚠️ **インデントのバグに注意**  
> `print("稼働終了")` をループの中（インデントあり）に置くと、  
> まだ処理が終わっていないのに毎周「稼働終了」と表示されるバグになる。

---

### 8｜`query` とは何か

```python
query = "tesuoの趣味は何ですか？プロファイル情報を調べて答えてください。"

for step in app.stream({"messages": [("user", query)]}):
```

`query` はユーザーの質問文を格納する変数。直接 `stream()` の中に書くこともできるが、括弧が何重にも重なる複雑な場所を毎回編集するとミスが増えるため、外に切り出して管理しやすくする設計。

---

## ⚠️ 重要な誤解の修正

### dict内のリストについて

```python
dummy_data = {
    "age": ["20", "24", "80"],      # リストがそのまま出る
    "hobby": ["hiking", "running", "swimming"],  # LLMが選ぶのではない
}
```

> ❌ **誤解：** `age` や `hobby` のリストはLLMが関数の内部で選んで出力する  
> ✅ **正解：** 関数は毎回リストをそのまま全部返す。LLMが選択するのは、このdictが `return` で外に出た**後**。LLMがプロンプトの文脈（「tesuoは年配で高いところが好き」）とdictの中身を照らし合わせて、回答を生成する際に初めて選択が行われる。

---

## 🐛 今日遭遇したエラーと対処法

### ① ModuleNotFoundError

```
ModuleNotFoundError: No module named 'langchain_google_genai'
```

**原因：** Colabのセッションがリセットされ、インストール済みのパッケージが消えた。  
**対処：** 実行前に新しいセルで再インストール。

```bash
!pip install -qU langchain "langchain[google-genai]"
```

> 💡 `"langchain[google-genai]"` の `[ ]` はPythonのExtras機能。  
> LangChain本体とGoogle専用拡張パーツを、最も相性の良いバージョンでセット買いする公式推奨の書き方。  
> `-q`（出力を静かに）、`-U`（最新版に更新）。

---

### ② 429 RESOURCE_EXHAUSTED

```
ClientError: 429 RESOURCE_EXHAUSTED
Quota exceeded for free_tier_requests, limit: 20
```

**原因：** Gemini APIの無料枠（1分20回）を超えた。LangChainのエージェントは1回の質問で裏側でAPIを3〜5回叩くため、数回テストするだけで上限に達する。

**対処法：**

| 方法 | 内容 |
|---|---|
| 待つ | 数秒〜1分でカウンターがリセットされる |
| `time.sleep(5)` | 連続実行する場合、コードに意図的な待機を入れる |
| ローカルLLMに切り替える | **根本解決。APIを叩かないので制限なし** |

---

## 🖥️ ローカル環境への移行（VS Code + Ollama）

### なぜローカルに切り替えたか

- クラウドAPIの無料枠制限を根本的に解決するため
- 基礎能力を鍛えるため（ノーコードツールに逃げない）
- LangChainはLLMのインターフェースが統一されているため、**数行書き換えるだけで脳の移植ができる**

### クラウド（Gemini）→ ローカル（Gemma 4）の切り替え箇所

```python
# ❌ クラウド版（Gemini）
from langchain_google_genai import ChatGoogleGenerativeAI
os.environ["GOOGLE_API_KEY"] = userdata.get('GOOGLE_API_KEY')
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

# ✅ ローカル版（Gemma 4 26B）
from langchain_ollama import ChatOllama
llm = ChatOllama(model="gemma4:26b", temperature=0)
# APIキー不要。手足のコード（Tool）は一切変更なし。
```

> 💡 **Gemma 4 26B について**  
> 2026年4月リリース。MoEアーキテクチャでアクティブパラメータ約3.8Bながら31Bクラスの推論能力。  
> ツール呼び出し（Function Calling）とThinkingモードをネイティブサポート。

### VS Codeでのインストール（Colabと違い `!` は不要）

```bash
pip install -qU langchain langchain-ollama
```

---

## 📝 自分の言葉でのまとめ

- Step 4は「手足を手動で動かしただけ」。Step 5は「脳に曖昧な指示を出して、脳が手足を動かした」。
- `temperature=0` はエージェント開発の鉄則。MacのCPU負荷とは無関係。
- `system_prompt` は性格付けだけでなく、LLMがツールをサボらずに使うよう強制する制御装置。
- `tools = [profile]` はデコレータの締めではなく、`create_agent` に渡すためのリスト作成。
- `invoke` は一発回答、`stream` はリアルタイム中継。どちらもLangChainの共通起動ボタン。
- `for step in app.stream(...)` のループで、ReActの各周回が可視化できる。
- `print("稼働終了")` はループの外（インデントなし）に置かないとバグになる。
- dictのリスト値はLLMが関数内部で選ぶのではなく、`return` で外に出た後にLLMが文脈から判断して選ぶ。
- クラウドAPIの制限にぶつかったら、ローカルLLMへの切り替えが根本解決。LangChainなら数行の変更で済む。

---

## 🗒️ 余白メモ｜面白いと思ったこと

一般的なプログラミング学習は「Pythonだけで完結するもの」として教わる。関数を作る、条件分岐を書く、ループを回す、すべてPythonの中で解決する前提。

自分は最初からそうではなかった。「Pythonが判断できないところはLLMに任せる」という前提でコードを読んでいる。だから `dict` の中にリストを入れた瞬間に「これはLLMが選ぶ部分だ」という発想が自然に出てきた。

これは誤解だったが、**誤解の方向性は正しかった**。「Pythonが確定的に処理する部分」と「LLMが判断する部分」の境界線をどこに引くか、という感覚は最初から持っていた。

2024年以降にプログラミングを学び始めた人間ならではの出発点かもしれない。
