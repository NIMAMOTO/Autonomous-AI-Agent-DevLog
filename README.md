# 自走型AI Agent作成記録

情報収集なんて、人でやるもんか！！

契機：

1. 就活の情報収集がしんどい！！pythonを学ぶ時間が取れない！！バランスを取り戻したい。
2. これを機にpythonで繋ぐAIのagentの稼働の仕組みを勉強する
3. open clawを諦めた悔しさまだ残っている

解決策：「じゃあ、その就活の情報収集自体を、AIエージェントに丸投げして自動化すればいいじゃん！」という天才的な発想がありました。

目標：APIを利用し、安全なシステム環境で情報収集を行うこと。

新たな発想：もしこのプロジェクトが成功したら、前に練習したPython とSQLのスキルと接続して、Agentによる情報収集、洗い出し、Python Script によるMySQLデータベースへの保存まで一貫した業務プロセスができる

##### 単語帳：

Python： スクレイピング：ウェブサイトから特定の情報を自動で抽出・収集する技術のこと（网络爬虫，网页抓取）

モジュール：（模块）
ステートレス：サーバーがクライアントの過去のやり取りや状態（セッション）を保持しないシステム設計（无状态）　対義語：ステートフル
HTTPSプロトコル：Webブラウザとサーバー間の通信を暗号化して安全に行うためのプロトコル
API（Application应用 Programming程序 Interface端口）通信とは、異なるソフトウェアやシステム同士がインターネットなどを介して機能やデータを共有・連携する仕組みです



## ロード

* まずフレームを知りたい、Agent稼働の仕組み。

  * どういう媒体で、どこからのスペック（token）
  * どこで作業を発生している→ローカルデバイス？
    * なぜクラウドでのAIがローカルで作業できる？
  * セキュリティ面の潜在的なリスク→Dockerを使う理由
  * lang chainフレームワークとしての仕組みと実現方法。その媒体（pythonスクリプト（脚本））の書き方

* AI回答

  * 稼働ロジック
    1. LLMクラウドからのスペック→API通信を経由しローカルデバイスに届く
    2. Python Script（Lang Chain）を通じてローカルデバイスで作業する
    3. 作業：就活サイト、企業採用ページにアクセスし、見つけたものをscriptを経由し、クラウドLLMに考えさせる。
    4. クラウドLLMLがフィードバックを出しscriptを経由で次の指示を下す
  * 媒体とTokenの出所
    * API通信とTokenの従量課金
  * 環境：コンテナ（docker）
    * Dockerとは：Macの中に「完全に隔離された、使い捨てのコンテナ
    * 絶対に使う理由（リスク）：AIの暴走・バグを防ぎ、Macの環境が守る。
  * ローカル作業ができる理由：LLM自体が作業できないがPython Scriptを指示して代行できる

  * 懸念点：

    * 就活サイトの「ログイン壁」
      * 予めPython Scriptにアカウントを保持させ、AIにブラウザ操作の具体的な手順えさせる
    * AIの「無限思考ループ」
      * Python Scriptに最大5回ループしたら強制終了や、「安全弁（リミッター）」のコードを絶対に1行入れておく
    * Dockerコンテナ内での「永続化（データの保存）」
      * Dockerコンテナの中のフォルダをMacのローカルフォルダに同期
    * APIキー（認証情報）の漏洩リスク
      * キー情報はコードに書かず、`.env` という外部の秘匿ファイルに保存し、Dockerの起動時に「環境変数（Environment Variable）」としてコンテナ内に安全に注入する設計（ベストプラクティス）を徹底します。

  * 実装するのに二つのルート

    1. Google Colabで、Pythonコードの裏側（ReActロジック）を覗き見しながらハックする
       特徴：ルールブックを**手書きコード**で実装Pythonで1行ずつガチ記述裏側の挙動を100%完全支配できる
    2. クラウド版Difyを使って、最速で「就活情報自動収集システム」の設計図を形にする
       特徴：ルールブックを**視覚的なブロック**で実装マウス操作 ＋ プロンプト調整爆速・安全。エラーで挫折しにくい

    * LangChainとの関係：１も２も稼働の根本にあるロジックはLangChain

## 知識の補足

### Lang Chain

* 稼働ロジック（ReActロジック）：
  思考→行動→ローカル実行→観察→思考→完了
* 正体：Python に基づたPython Script（コード）のこと。
* 本質：AIをうまく動かすコードを書くロジック、一つの考え方のこと。
  →従来の「一本道」ではなく「ループ（自律思考）」という考え方
* Chain（チェーン）とは：
  * クラウドLLM→Prompt→Tools
  * LLMとPrompt の関係：
    * LLM：数千億の言葉の概念が、未整理のまま詰まったプール（神経細胞）
    * Prompt：電気信号そのもの。「お前は就活エージェントだ。ReActロジックに従って、Thought、Action、Observationのループを回せ」
      PromptTemplate：電気刺激のセット、ガイドブック
    * 思考回路：Prompt（電気信号）の刺激でLLM（神経細胞）が反応し、バラバラだった神経細胞が一瞬で「専用の回路」へと形を変える。この刺激入力、反応、輸出という流れがあった初めて思考回路を形成される
* PromptTemplate（Python Scriptの一部）から、Prompt①を出力
  →LLM反応→Python Scriptが指示され→ローカルAction→新事実（Observation）を発見！→結果をPromptTemplateに持ち込み→Prompt②を出力
  →LLM反応（集めたものを観察、Prompt②に基づいて思考）→LLM次の行動を下す→Python Scriptが指示され→ローカルAction......![img](https://www.ibm.com/adobe/dynamicmedia/deliver/dm-aid--2b2dd7fd-2f04-44fa-8082-a876210a3fde/prompt-chaining-langchain.png?preferwebp=true)

### Deploy、デプロイ、構築、部署

APIの選択　by Gemini3.5 Flash

| **企業名（モデル名）**            | **主な特徴（強み）**                          | **メリット（エージェント開発視点）**                         | **デメリット（リスク）**                                     |
| --------------------------------- | --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Google** (Gemini 1.5 Pro/Flash) | **超巨大な記憶容量** (コンテキストウィンドウ) | * **圧倒的な無料枠**（AI Studioで即日ノーリスクで試せる） * 大量の就活サイトのデータを丸ごと脳みそにブチ込める。 | * OpenAIに比べると、LangChainの最新機能の対応が数日遅れることがある。 |
| **OpenAI** (GPT-4o / GPT-4o-mini) | **世界標準・最強の論理思考**                  | * エージェントの自律思考（ReAct）の成功率が最も高い。 * 世界中の教材やコードがGPT前提なのでエラーの解決策が見つかりやすい。 | * **完全従量課金**（クレカ登録が必要）。無料枠がほぼなく、暴走時のコストリスクがある。 |
| **Anthropic** (Claude 3.5 Sonnet) | **超高精度なコード記述・指示理解**            | * プログラミングのコード生成や、複雑なプロンプト（金型）の指示を1ミリも外さず理解する。 | * OpenAI同様、完全に有料（従量課金）。無料枠がない。         |



開発環境（Development Environment）の選択　by Gemini3.5 Flash

３大スタイル

* 実験・学習スタイル：
  Google Colab / Jupyter Notebook
  * 使用する人物像：学習者、データサイエンティスト、AIの研究者。
  * メリット：ブラウザを開くだけで、環境構築が不要で100%安全
  * デメリット：ブラウザを閉じると一定時間で接続が切れ、プログラムが止まる（24時間運用は無理）。
* 本格開発スタイル：
  VS Code＋ ローカルPC
  * 使用する人物像：全世界9割のエンジニア、プロのシステム開発者。
  * メリット：自分のPC内で複数のファイルを連携させ、Git（GitHub）でコードの歴史を管理しながらガチガチにシステムを組める。
  * デメリット：ローカル環境にPythonを直接入れるため、環境が汚れるリスクがある（だからDockerと組み合わせる）。
* 本番運用スタイル：
  クラウドサーバー（AWS / GCP）＋ Docker
  * 使用する人物像：完成したシステムを24時間自動で動かしたい人。
  * メリット：ローカル環境の電源を切っても、AmazonやGoogleから24時間眠らないサーバーの中でエージェントが24時間稼働できる。
  * デメリット：月数千円のサーバー代がかかる。



開発環境と本番環境とDeployの違い

* Development Environment（開発環境）：コードを書いたり実験したりの場所。略称：dev environment
* Production Environment（本番環境）：完成したシステムを世の中に放り出して動かす場所。略称：prod
* Deploy：完成したシステムを開発環境から本番環境の配置するときのAction



## 開発ステップ

### 私の開発スタイル（助言by Gemini3.5 Flash）

1. 開発環境：Google Colab
   API：完全無料のGeminiのAPI
   →LangChainのロジックが動いてみる
2. Deploy環境：VS Code＋ Docker
   API：任意（APIを変えるのにコードを1行書き換えて切り替える）
   →デバイスの中で安全な自動化システムを組み、Dockerの練習をする
3. （オプション）
   Deploy環境：クラウド
   API：任意
   →24時間運用させたいときのみ

### 開発の流れ

1. APIキーの取得
2. 環境の準備とライブラリ導入
3. 環境変数（セーフティ）の設定
4. Pyhton Scriptの執筆と実行

#### ステップ１：Get API key

> https://aistudio.google.com/

API key：

#### ステップ２：

Colabの画面に最初からある、プログラムをセルにコマンドを打ち、LangChainのインストールをする

> !pip install -qU langchain-google-genai langchain



#### ステップ４：（以下のコードは全部AIにより作成したもの）

テストコード　

~~~python
import os
from google.colab import userdata
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.agents import AgentExecutor, create_react_agent
from langchain_core.tools import tool
from langchain import hub

# 1. 鍵マーク（シークレット）から安全にAPIキーを読み込む
os.environ["GOOGLE_API_KEY"] = userdata.get('GOOGLE_API_KEY')

# 2. クラウドにあるGeminiの「神経細胞（モデル）」を定義
# 思考のブレをなくすために temperature=0 (一番冷静なモード) に設定
llm = ChatGoogleGenerativeAI(model="gemini-1.5-flash", temperature=0)

# 3. エージェントに持たせる「手足（Tool）」を定義する
# ここでは練習として、文字数を数える簡単な関数を「道具」として登録
@tool
def count_letters(text: str) -> int:
    """文字数をカウントするツール。テキストを入力すると、その文字数を整数で返します。"""
    return len(text)

# 道具箱のリストを作成
tools = [count_letters]

# 4. LangChainの公式リポジトリから、世界標準の「ReActの金型（PromptTemplate）」をダウンロード
prompt = hub.pull("hwchase17/react")

# 5. 「脳みそ」＋「手足」＋「金型」をガッチャンコして、思考のベースを組み立てる
agent = create_react_agent(llm, tools, prompt)

# 6. エージェントを実行する司令塔を定義
# 「暴走リミッター(max_iterations)」と、裏側を覗き見する「verbose=True」をセット！
agent_executor = AgentExecutor(
    agent=agent, 
    tools=tools, 
    verbose=True, 
    max_iterations=3  # 万が一の無限ループを防ぐ安全弁
)

# 7. 実行命令を下す
print("--- エージェントの自律思考スタート ---")
agent_executor.invoke({"input": "『関西大学経済学部』という文字列の文字数を数えてください。"})
~~~



エラーメッセージ

```python
---------------------------------------------------------------------------
ImportError                               Traceback (most recent call last)
/tmp/ipykernel_5595/2144685422.py in <cell line: 0>()
      2 from google.colab import userdata
      3 from langchain_google_genai import ChatGoogleGenerativeAI
----> 4 from langchain.agents import AgentExecutor, create_react_agent
      5 from langchain_core.tools import tool
      6 from langchain import hub

ImportError: cannot import name 'AgentExecutor' from 'langchain.agents' (/usr/local/lib/python3.12/dist-packages/langchain/agents/__init__.py)

---------------------------------------------------------------------------
NOTE: If your import is failing due to a missing package, you can
manually install dependencies using either !pip or !apt.

To view examples of installing some common dependencies, click the
"Open Examples" button below.
---------------------------------------------------------------------------
```

数回のエラーの積み重ね遂に成功した
成功したコード

 ```python
 import os
 from google.colab import userdata
 from langchain_google_genai import ChatGoogleGenerativeAI
 from langchain_core.tools import tool
 from langchain.agents import create_agent
 
 # 1. APIキーの設定
 os.environ["GOOGLE_API_KEY"] = userdata.get('GOOGLE_API_KEY')
 
 # 2. 【ここを修正！】現在稼働している最新モデル「gemini-2.5-flash」を呼び出す
 llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)
 
 # 3. 手足（Tool）の定義
 @tool
 def count_letters(text: str) -> int:
     """文字数をカウントするツール。テキストを入力すると、その文字数を整数で返します。"""
     return len(text)
 
 tools = [count_letters]
 
 # 4. 脳みそと手足をガッチャンコ（2026年最新のLangChainアーキテクチャ）
 app = create_agent(
     model=llm, 
     tools=tools,
     system_prompt="あなたは実直なアシスタントです。質問に答えるために必要なツールがあれば、必ず使ってください。"
 )
 
 # 5. 実行
 print("--- 2026最新エージェント自律思考スタート ---")
 response = app.invoke({
     "messages": [("user", "『関西学院大学経済学部』という文字列の文字数を数えてください。")]
 })
 
 # 6. 結果出力
 print("\n【最終結果】")
 print(response["messages"][-1].content)
 ```



### エラー原因まとめ　by Gemini3.1 Pro

#### ① `AgentExecutor` が見つからないエラー（ImportError）

- **エラーの正体：** LangChainの「部品の置き場所（フォルダ構造）」の変更。
- **背景：** 以前は `langchain.agents` という一番目立つ場所に置いてあった実行部隊（AgentExecutor）が、アップデートによって別の深い階層に引っ越しさせられたため、「そこにはもう無いよ！」とPythonが怒りました。

#### ② `langchain.agents.agent` が無いエラー（ModuleNotFoundError）

- **エラーの正体：** 引っ越し先すらも破壊されるという、大規模なアーキテクチャ変更。
- **背景：** LangChain開発チームが「古いエージェントの仕組みはもう捨てて、LangGraphという新しい仕組みに完全移行しよう」と決断したため、古いモジュールが跡形もなく消し飛んでいました。

#### ③ `create_react_agent` の引数エラー（TypeError）

- **エラーの正体：** 過渡期における関数の仕様（ルール）変更。
- **背景：** 最新の枠組みを使おうとしたところ、今度は「引数（プロンプトを渡すための変数名）」のルールが `state_modifier` から `system_prompt` などへコロコロ変わっており、「そんな名前のデータは受け取れない！」と弾かれました。

#### ④ モデルが見つからないエラー（404 NOT_FOUND）

- **エラーの正体：** Google側のインフラ（APIサーバー）での旧モデルの完全廃止。
- **背景：** `gemini-1.0-pro` や `gemini-1.5-flash` といった、数ヶ月前まで最新だったモデルが、Googleのサーバーから物理的に削除（またはアクセス制限）されていました。

まとめ：コード自体に問題はなかったが、１−３番の問題はLangChain公式から仕様変更の罠に陥っている。４番はGoogle側の変更と最新に採用しているモデルを使っていなかったことが原因でした

振り返り：ごろごろ変わる今の時代では、自ら公式に最新の情報を探して、常に最新の知識を持たないとすぐ時代遅れになる。AIにはそうなるから、人も然り。

* Python Scriptを独自で書ける能力
* 独自で公式ドキュメントを読め、最新の変更に常についていけるように



## トレーニング計画

### フェーズ1：エージェント開発に「絶対必要な」基礎だけを鍛える

Pythonには膨大な機能がありますが、AIエージェントを作るために必要なのは実はごく一部です。まずは以下の**4つの武器**だけを完璧に使いこなせるようにトレーニングしてください。

1. **データ構造（`dict` と `list`）**
   - **なぜ必要？:** AIとの通信（APIリクエスト）や、LangChainの裏側でやり取りされるデータは、100%「JSON（Pythonでいう辞書型 `dict` と配列 `list` の組み合わせ）」で動いているからです。ここが読めないと公式ドキュメントの引数も理解できません。
2. **関数の定義（`def` と 型ヒント）**
   - **なぜ必要？:** エージェントに持たせる「手足（Tool）」はすべて関数で作ります。引数に何を受け取り、何を `return` するのかを自分で設計できるようになる必要があります。
3. **外部との通信（`requests` ライブラリ）**
   - **なぜ必要？:** AIの脳みそ（LLM）と通信する「API」の基礎構造を知るためです。
4. **エラー処理（`try / except`）**
   - **なぜ必要？:** スクレイピングやAPI通信は必ずどこかでコケます。エラーが起きた時にプログラムを止めず、AIに「失敗したから別の方法を考えて」と促すために必須の構文です。

### フェーズ2：「公式ドキュメント」の読み方と破壊（ハック）

基礎が少し身についたら、次は「公式ドキュメント（一次情報）から知識を吸収する」トレーニングです。英語のドキュメントに慣れることが最強の近道です。

1. **「Quickstart」と「Concepts」だけを読む**
   - ドキュメントの最初から最後まで読む必要はありません。まずは動かすための「Quickstart」のコードをColabにコピペします。
   - そして、そのコードが「なぜ動いているのか」の概念（Concepts/Architecture）のページだけを熟読します。
2. **コピペしたコードを「わざと壊す」**
   - 動いたコードの引数（パラメータ）を1つだけ消してみたり、変数名を変えてみたりして、**わざとエラーを出します**。
   - どんなエラーメッセージが出るかを観察することで、「あ、この引数がないとこの機能は死ぬんだな」という**システムの骨格**が手にとるように分かるようになります。

### フェーズ3：LangChainを「使わずに」ミニアプリを作る（車輪の再発明）

これが一番実力が跳ね上がるトレーニングです。 LangChainなどの便利なフレームワークは、裏側で複雑な処理を隠してくれている「ブラックボックス」です。仕様変更についていけなくなる原因は、このブラックボックスの中身を知らないからです。

- **課題:** LangChainを一切使わず、Pythonの標準機能と `requests` （または公式SDK）だけで、「ユーザーが文字を入力したら、Gemini APIを叩いて、返答をprintするだけの永遠に続くループ（チャットアプリ）」を作ってみてください。

これを自力で書くと、「あ、ループ（while）の中で過去の会話履歴（list）を毎回APIに送り直さないと、AIは文脈を忘れちゃうんだな」というLLMのステートレスな性質（仕組みの根本）が痛いほど分かります。 その後にLangChainを使うと、「うわ、この面倒な処理を1行でやってくれてたのか！」と感動し、公式ドキュメントの意図が手に取るように読めるようになります。