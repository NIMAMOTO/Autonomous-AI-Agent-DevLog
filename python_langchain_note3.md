# Python訓練3

Agentの手足の拡張

目標：Agentに自力で外部サイトに情報収集してもらう

関数：requests, BeautifulSoup

なぜその関数がいるのか、他のもっと使いやすい関数はにないのか？
公式ドキュメントに本当に馴染んだか？

単語帳：
引数（argument）：「関数」と呼ばれる処理のまとまりに、外部から渡すデータ（入力値）のこと
HTTP POST：Webブラウザなどのクライアントからサーバーへデータを送信する際に使用されるHTTPリクエストメソッド
query：質問
encode:データ（文字、画像、動画など）を扱いやすい別の形式に変換する作業全般
Parametric：param：参数
Usage：使い方
characters：文字データ、字符

## 公式ドキュメント訓練法

1. **知る**
   AIに目的関数と公式ドキュメントアドレスを提示してもらう
2. **見つける**
   自力で公式ドキュメントでその答え（目的関数）を見つけ出す
3. **広げる**
   見つけたら、その周辺を読む。
   同じページにある他の情報を読む
   →質問：この公式はなぜこのページにある？
   →質問：なぜその答えのあるページが公式ドキュメントのこの位置にある？
4. **壊す**
   理解したらわざと壊す
   →「このコードが生きている理由を 死なせることで確認する」
   * 関数を消す→エラーを読む
     →エラーを理解する
   * 引数を変える→何が変わるか見る
     →引数と関数の関係を理解する
   * 順番を入れ替える→動くか確認する
     →関数の位置と構造を理解する
5. **作る**
   作ることで、自分のものにする
   * 関数を書いてみる（必ず自力で）
     関数を自分のものにする
   * パラメータがあれば、弄ってみる
     「なぜこの値にするか」を考える
   * 動いたらHithubに上がる
     自分の中に理解したものを他人に見えるようにする



1. https://requests.readthedocs.io/

2. requests.get('URL')
   requests.put
   requests.delete
   requests.head
   requests.options
   payload = {'key1': "value1", 'key2': "value2"}
   r= requests.get('https://www.notion.so/Portfolio-12359110f3c483299362811cc8da89af?source=copy_link', params=payload)

3. 質問：この公式はなぜこのページにある。他まだ何がある？

   * この公式はRequestsの基本的な使い方（外部へアクセス）として、最初に位置（make a Requests）置いられた。
   * 他には、URLの中でパラメータを渡す、回答の内容、バイナリ（Binary、2进制度）の回答の内容、JSONの回答の内容、生（Raw）の回答の内容、カスタマヘッダー、よく複雑なPOSTリクエスト、POST マルチパートエンコードされたファイル、レスポンスステータスコード、ヘッダー回答、Cookies、向け直すと歴史、タイムアウト、エラーと例外（Exceptions）

   質問：なぜその答えのあるページが公式ドキュメントのこの位置にある？他まだ何がある？

   * このページ（Quike start）は初心者のRequestsの使い方を教わるために、Useful Linksの一個目においられた。
   * クイックスタート、高度な使用、API参考、リリース履歴、貢献者ガイド、推奨パッケージと拡張機能、リクエスト @ GitHub、リクエスト @ PyPI、課題トラッカー

場所間違った、Developer Interfaceこそ正しいところだた。

params – (optional) Dictionary, list of tuples or bytes to send in the query string for the Request.
data – (optional) Dictionary, list of tuples, bytes, or file-like object to send in the body of the Request.



requests.**get**(*url: _t.UriType*, *params: _t.ParamsType = None*, ***kwargs:Unpack[_t.GetKwargs]*) → [Response](https://requests.readthedocs.io/en/latest/api/#requests.Response)[[source\]](https://requests.readthedocs.io/en/latest/_modules/requests/api/#get)

Sends a GET request.

- Parameters:

  **url** – URL for the new [`Request`](https://requests.readthedocs.io/en/latest/api/#requests.Request) object.**params** – (optional) Dictionary, list of tuples or bytes to send in the query string for the [`Request`](https://requests.readthedocs.io/en/latest/api/#requests.Request).***\*kwargs** – Optional arguments that `request` takes.

- Returns:

  [`Response`](https://requests.readthedocs.io/en/latest/api/#requests.Response) object

- Return type:

  [requests.Response](https://requests.readthedocs.io/en/latest/api/#requests.Response)

BeautifulSoup

```python
markup = '<a href="http://example.com/">\nI linked to <i>example.com</i>\n</a>'
soup = BeautifulSoup(markup, 'html.parser') 

soup.get_text() 
'\nI linked to example.com\n' 
soup.i.get_text() 
'example.com' 
```



0からコードを書いてみる

1. ツールを導入
   LangChain（toolsとcreate_agent）、LangChainをOllama(ChatOllama)と繋げる、OllamaでLocal LLMを呼び出す 
2. 手足を組む
3. Agentを作る
4. 結果をインポートする

