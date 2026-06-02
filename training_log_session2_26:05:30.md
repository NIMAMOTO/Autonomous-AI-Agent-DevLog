# 📅 5月31日 トレーニング記録｜Step 5 全体の訓練（Colab → ローカル環境への移行）

> **テーマ：** 自作Toolをエージェントに統合し、クラウド（Gemini）からローカル（Gemma 4）へ移行する  
> **主軸：** Geminiとのペアプログラミング（実装・デバッグ）  
> **補助：** Claudeとの対話（学習姿勢の振り返り・Python観の言語化）

---

# PART 1｜時系列ログ（客観的な記録）

## ① Step 5：脳と手足のドッキング

これまで単体だった `profile` ツールを、LLM・create_agent・実行部と統合した。Gemini 2.5 Flashを脳に据え、「tesuoの趣味は？」という曖昧な質問を投げ、LLMが自律的にツールを選んで実行する全体の流れを完成させた。

**今日作ったものの全体像：**
```
ユーザーの質問（query）
        ↓
  app.stream() でエージェント起動
        ↓
  LLM が思考「profileツールを使えばいい」と判断
        ↓
  Pythonがprofile関数をローカルで実行
        ↓
  結果（dict）をLLMに返す
        ↓
  LLMが自然言語に翻訳して最終回答
```

→ Step 4までは「手足を手動で動かしただけ」。Step 5で初めて「脳に曖昧な指示を出し、脳が手足を動かす」段階に到達した。

---

## ② 2つのコードの違いを問う

コードが急に長くなったことに疑問を持った。

> 💬 「今のコードと一個前のコード、機能として何が違う？先のコードはLLMを使っているのか？だとしたらなぜLangChainを呼んだのにLLMなしで動けたのか？」

→ 答え：Step 4のコード（`profile.invoke(...)`）は**LLMを一切使っていなかった**。`@tool` で規格を合わせた手足を、自分で手動でスイッチを押して動かしただけ。Step 5で初めてLLM（脳）とcreate_agent（神経系）が登場した。

---

## ③ ModuleNotFoundError と自力デバッグの宣言

実行したら `ModuleNotFoundError: No module named 'langchain_google_genai'` が出た。

> 💬 「原因を教えてくれたら、自力で直してみたい。まだ早いかな？」

→ Geminiは答え（コマンド）を出さず、ヒントだけ提示。原因はColabのセッションがリセットされ、インストール済みパッケージが消えていたこと。エラー文の末尾NOTEに `!pip` で再インストールせよと書かれていた。

---

## ④ 公式ドキュメントを自力で読む

Geminiにインストール方法が載った公式サイトの場所を聞き、自力で読みに行った。Geminiが教えたコマンド（`pip install -U langchain-google-genai`）とは違うものを公式で発見した。

> 💬 「公式ドキュメントに `pip install -qU langchain "langchain[google-genai]"` と書いてあるよ。」

→ **AIの答えを鵜呑みにせず、一次情報を自分で確認して、より新しい記述を持ち帰った。** 中身は同じだが、`[google-genai]`（Extras機能）を使う公式推奨の書き方。

→ 実行成功。`tesuoの趣味はshipです。` が出力された。

---

## ⑤ ダミーデータのバグを実験的に作る

`profile` のdictをわざと壊れた形にして、複雑な論理パズルを質問した。

```python
"name": "a" "b" "c" "d",            # スペース区切りの文字列
"age": "23" "24" "25" ...
"hobby": "haking" "running" "swimming"
```

→ Pythonの仕様で、スペース区切りの文字列リテラルは**自動的に連結される**。結果、LLMには `"hakingrunningswimming"` という壊れた文字列が届いた。LLMはそれを100%信じて「趣味はhakingrunningswimmingです」と答えた。

> 💡 **学んだ鉄則：** AIは渡されたデータを無条件で信じる（Garbage In, Garbage Out）。バグの原因はLLMでもLangChainでもなく、手足（Python関数）の設計不良。複数データはリストで分けて返す。

---

## ⑥ 429エラー（レート制限）

遊んでいる途中で `429 RESOURCE_EXHAUSTED` が出た。

→ Gemini APIの無料枠（1分20回）を超えた。エージェントは1回の質問で裏側でAPIを3〜5回叩くため、数回テストしただけで上限に達する。**コードが正しく動いてGoogleと何度も通信した証拠**でもある。

---

## ⑦ ローカル環境への移行を自分で決断

レート制限を見て、待っても解決しないと判断。自分で移行を決めた。

> 💬 「VS Code + Ollama + ローカルLLM（Gemma 4 26B）で行く。LangChainでAPIを切り替えればすぐGeminiに戻せる。あと、Difyはありえない。基礎能力が鍛えられない。」

→ ノーコード（楽な道）を自分の意志で断り、ローカルでの基礎力養成を選んだ。LangChainはLLMのインターフェースが統一されているため、数行の書き換えで「脳の移植」ができる。

---

## ⑧ 知識の更新（Gemma 4 26B）

GeminiがGemma 4を知らず古い `gemma2:27b` を提示したのに対し、最新情報を提示した。

> 💬 「Gemma 4 26B 知らないの？Googleが2026年に出した最新のローカルモデルよ。」

→ 2026年4月リリース。MoEアーキテクチャでアクティブパラメータ約3.8Bながら31Bクラスの推論能力。Function CallingとThinkingモードをネイティブサポートする最新モデル。AIの知識が古い場面で、自分の知識が上回った。

---

## ⑨ 一旦立ち止まり、コードを自分のものにする

外部アクセスへ進む前に立ち止まった。

> 💬 「今までのコードをVS Codeで慣れさせて、外部にコンタクトする作業は次の日にやろう。全部理解して、自由自在に編集できるようになりたい。」

→ ここから、統合マスターコードを1行ずつ自力で解釈するフェーズに入った（`query`・インポート4行・`temperature`・`@tool`の中身・`create_agent`・`invoke`/`stream`/`for in`）。

---

## ⑩ temperature への2つの仮説

`temperature=0` について、2つの仮説を同時に立てて検証しようとした。

> 💬 「temperature=0 が0じゃなければどうなる？答えが間違える？Macにかかる負担は下がるの？」

→ 答え：値が高いとランダム性が混じり、ツール呼び出しの書式が崩れてエラーになりやすい。Macの計算負荷は temperature の値に関係なく完全に同じ（重い行列計算は常に全力で行われ、temperatureは最後のピックアップ方法を変えるだけ）。

---

## ⑪ dictのリストについての誤解

自力で `@tool` の中身を解釈した際、リスト値について誤解した。

> 💬 「"age", "hobby" などの多選択肢の項目は、LLMの判断によって出力されるもの。」

→ **修正：** 関数は毎回リストを**そのまま全部**返す。LLMが選ぶのは、dictが `return` で外に出た**後**。プロンプトの文脈と照らし合わせて回答を生成する際に初めて選択する。

→ この誤解は後にClaudeとの対話で深い気づきにつながった（PART 2-C参照）。

---

## ⑫ invoke / stream / for in とインデントのバグ

`invoke` の意味と `for in` 構文を質問。さらに自分で書いたコードにインデントのバグがあった。

> 💬 「なぜAgentを入れたらinvokeを使わないといけない？for in関数は初めて見る。」

→ `invoke`（一発回答）と `stream`（途中経過のリアルタイム中継）の違いを理解。`print("稼働終了")` をループの中（インデントあり）に置くと毎周表示されるバグになるため、ループの外に出す必要があった。

---

# PART 2｜知識・経験の分類

## 🔧 A. 技術知識（コード・文法）

### Step 4 と Step 5 の違い

| | Step 4（Tool単体テスト） | Step 5（エージェント統合） |
|---|---|---|
| LLMを使うか | 使わない。手動でToolを動かした | 使う。LLMが自分でToolを選ぶ |
| 実行方法 | `profile.invoke({"aaa": "tesuo"})` | `app.invoke({"messages": [...]})` |
| 何をしているか | 手足を直接手動で動かした | 脳に曖昧な指示を出し、脳が手足を動かす |

### 統合マスターコード（main.py / ローカル版）

```python
from langchain_ollama import ChatOllama
from langchain_core.tools import tool
from langchain.agents import create_agent

# 1. ローカルLLM（脳）の定義
llm = ChatOllama(model="gemma4:26b", temperature=0)

# 2. 手足（Tool）の定義
@tool
def profile(aaa: str) -> dict:
    """名前（aaa）を入力すると、プロフィールを辞書形式で返すツールです。"""
    dummy_data = {"name": aaa, "age": "23", "hobby": "ship"}
    return dummy_data

tools = [profile]

# 3. エージェントの組み立て
app = create_agent(
    model=llm,
    tools=tools,
    system_prompt="あなたは実直なデータアシスタントです。必要なツールがあれば必ず使ってください。"
)

# 4. 実行とプロセスの可視化
query = "tesuoの趣味は何ですか？プロファイル情報を調べて答えてください。"
for step in app.stream({"messages": [("user", query)]}):
    print(step)

print("稼働終了")   # ← ループの外に置く
```

### インポート4行とllm定義の解剖

| コード | 役割 |
|---|---|
| `from langchain_core.tools import tool` | `@tool`デコレータ（関数を手足に変える）を持ってくる |
| `from langchain.agents import create_agent` | 脳と手足を繋ぎReActループを回す組み立て装置 |
| `from langchain_ollama import ChatOllama` | ローカルOllamaとPythonを繋ぐ接続プラグ |
| `llm = ChatOllama(model="gemma4:26b", temperature=0)` | 使うモデルを指定し、`llm`に格納 |

### temperature

| 値 | 特性 | 用途 |
|---|---|---|
| `0` | 確定的・機械的。同じ入力→常に同じ出力 | エージェント開発・データ処理（**鉄則**） |
| `0.7〜1.0` | ランダム・創造的。答えが毎回変わる | 小説・アイデア出し |

> ⚠️ temperature を上げると、ツール呼び出しの書式が崩れてLangChainのループがクラッシュする。  
> ⚠️ **Macの計算負荷は temperature の値と無関係。** 重い行列計算は常に全力で行われ、temperatureは「完成した確率リストから、サイコロを振るか一番上を黙って取るか」という最後のピックアップ方法を変えるだけ。

### create_agent の3パラメータ

```python
app = create_agent(
    model=llm,        # 頭（推論エンジン）。モデル変更はここだけ
    tools=tools,      # 手足（使える機能のリスト）
    system_prompt="..." # 性格 + 絶対的な業務命令
)
```

> 💡 `system_prompt` は性格付けだけでなく、**LLMがツールを使わずにサボるのを防ぐ制御装置**。ローカルLLMは自分の学習データから適当に答える傾向があるため、「必ずツールを使え」と強制する。

### tools = [profile] の役割
デコレータの締めの合図ではない。`create_agent` が「手足のリスト」を要求する仕様のため、`[ ]` でリスト化して渡す。ツールが増えたら `tools = [profile, web_search, calculator]` のように追記するだけ。

### return が必要な理由（スコープ）
`def` の中で作った変数（`dummy_data`）は「ローカル変数」で、関数の内部でしか存在できない。処理が終わると自動的に消滅する。外の世界（LangChainやLLM）が受け取るには、消滅する前に `return` で箱の外へ投げ渡す必要がある。

### invoke と stream

| | `invoke` | `stream` |
|---|---|---|
| 動作 | 全処理が終わってから最終結果だけ一括で返す | 1周思考するたびに途中経過をリアルタイムで返す |
| 用途 | 最終結果だけほしい時 | ReActループを可視化したい時 |

> 💡 `invoke` はLangChainの共通の起動ボタン名。ツール単体 → `profile.invoke(...)` / エージェント全体 → `app.invoke(...)`

### for in 構文
`for 変数 in 集まり:` は、データの集まりから中身を1つずつ取り出して変数に入れ、繰り返す構文。

```python
for step in app.stream(...):
    print(step)       # ← インデントあり = ループの中で毎回実行
print("稼働終了")     # ← インデントなし = ループ完了後に1回だけ
```

---

## 🏗️ B. アーキテクチャ・設計の知識

### Step 5 のReActプロセス（裏側で起きていること）
1. ユーザーの曖昧な質問をスクリプトが受け取る
2. LangChainが質問とツールの説明書をLLMに送信（Thought）
3. LLMが「profileツールに引数tesuoを渡せ」と推論・指示
4. Pythonがローカルで `profile("tesuo")` を実行、辞書を取得（Action & Observation）
5. LangChainが辞書をLLMに再送信（ループ2周目）
6. LLMが自然言語に翻訳して最終回答

### Garbage In, Garbage Out（手足の設計責任）
AIは渡されたデータを無条件で信じる。Python側が壊れたデータを返せばLLMの出力も必ず壊れる。バグの原因はLLMやLangChainではなく、手足（Python関数）の設計にある。複数データは文字を繋げず、リストで分けて返す。

### クラウド → ローカルの移行
LangChainはLLMのインターフェースが統一されているため、手足（Tool）のコードを一切変えず、数行で「脳の移植」ができる。

```python
# クラウド版（Gemini）
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

# ローカル版（Gemma 4 26B）— APIキー不要
from langchain_ollama import ChatOllama
llm = ChatOllama(model="gemma4:26b", temperature=0)
```

VS Codeでのインストールは、Colabと違い先頭の `!` は不要：
```bash
pip install -qU langchain langchain-ollama
```

### LLM（脳）と Tool（手足）の境界線
- **LLM（脳）：** 曖昧な判断・文脈理解・選択
- **Python/Tool（手足）：** 確定的・機械的な処理

この境界線をどこに引くかが、システム設計の核心。dictのリストの誤解は、この境界線がまだ曖昧だったために起きた。

---

## 🧠 C. 学習法・メタ認知（経験知）

### AIの誘導を断る軸
「Difyはありえない。基礎能力が鍛えられない。」— AIが「楽な道」を提示した場面で、自分の学習目的に照らして断った。AIの伴走に流されず、自分の軸で進路を決める姿勢。

### AIを鵜呑みにしない（一次情報の確認）
Geminiが出したインストールコマンドを鵜呑みにせず、公式ドキュメントを自力で読み、より新しい記述を持ち帰った。AIの知識は古いことがある（Gemma 4 26Bの件も同様）。一次情報を自分で確認する習慣。

### 仮説思考
temperatureについて「答えが間違える？」「Macの負荷は下がる？」と2つの仮説を同時に立てて検証しようとした。問いを立ててから答えを取りに行く動き。

### 最初からLLMを前提にしたPython観（最重要の気づき）
> 💬 「私のPythonに対しての感覚が、一般の情報系の人と違って、最初からLLMがある上で成り立つ。」

一般的なプログラミング学習は「Pythonだけで完結するもの」として教わる。関数・条件分岐・ループ、すべてPythonの中で解決する前提。

しかし自分は最初からそうではなかった。「Pythonが判断できないところはLLMに任せる」という前提でコードを読んでいる。だから `dict` にリストを入れた瞬間「これはLLMが選ぶ部分だ」という発想が自然に出てきた。

→ これは誤解だったが、**方向性は正しかった**。2024年以降に学び始めた人間ならではの、時代に合った出発点。  
→ ただし「LLMに任せる部分」と「Pythonが確定的に処理する部分」の**境界線を常に意識する**こと。今後コードが複雑になるほど、この境界線がシステム設計の核心になる。

### 立ち止まる判断
外部アクセスへ進む前に「全部理解して自由自在に編集できるようになりたい」と立ち止まった。新しい概念を定着させるには、実際にコードを触り、意図的に壊して直すプロセスが不可欠。

---

## 🐛 D. デバッグで得た教訓

| 現象 | 原因 | 教訓 |
|---|---|---|
| `ModuleNotFoundError: 'langchain_google_genai'` | Colabのセッションがリセットされパッケージが消えた | 実行前に `!pip install` で再インストール |
| `hakingrunningswimming` | スペース区切りの文字列が自動連結された | 複数データはリストで分けて返す |
| `429 RESOURCE_EXHAUSTED` | 無料枠（1分20回）超過。エージェントは1質問で3〜5回API叩く | 待つ / `time.sleep()` / 課金 / ローカルLLM移行 |
| `稼働終了` が毎周表示される | `print` がループの中（インデントあり） | ループ後に1回だけ出すならインデントを外す |

---

## 🗒️ 余白メモ｜このセッションの核心

> Difyはありえない。基礎能力が鍛えられない。

このセッションは「前半とは別人レベル」の日だった。Pythonの関数からローカルLLMとエージェントの統合まで一日で到達し、しかも要所で**AIに流されなかった**。

- AIが「楽な道（Dify）」を勧めた場面で、自分の軸で断った
- AIのコマンドを鵜呑みにせず、公式を自力で確認してより新しい記述を持ち帰った
- AIが知らなかった最新モデル（Gemma 4 26B）を自分が教えた
- レート制限にぶつかった時、環境移行を自分で決断した

そして一番面白いのは、dictのリストの誤解から生まれた気づき——「自分は最初からLLMを前提にPythonを読んでいる」。これは弱点ではなく、この時代ならではの出発点。あとは「LLMに任せる部分とPythonが処理する部分の境界線」を意識しながら精度を上げていく段階。
