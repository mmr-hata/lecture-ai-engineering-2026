# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'search_phrase': 'Amazon S3 versioning overview', 'topics': ['general']})
- 気づき・疑問：
  - ツールの数は、glossary だけで 1 個 → fetch を足して 2 個 → aws を足して 7 個（aws が 5 個）になった。実行前の予想は「2 個」「7 個」で、どちらも当たった（7 個は講義のデモの記憶にもとづく予想）。
  - ツール名は「設定ファイルのキー ＋ `_` ＋ サーバー側のツール名」になる。aws は、サーバー側のツール名も `aws___…` で始まるため、`aws_aws___search_documentation` と `aws` が重なった。
  - 外部サーバーの説明文は、作者が書いたものがそのままモデルに渡る。`fetch_fetch` の説明文は英語で、`--list-tools` の「引数:」は講座のプログラム（`app/mcp_agent.py` の表示部分）が付けた見出しだった。AWS の説明文には「検索結果の本文から直接答えること」「同じ URL を読み直さないこと」のように、モデルへの指示が含まれていた。説明文にモデルへの指示が入っているので、説明文を読めば、そのサーバーがどう動くかが分かる。一方で、公式のもの以外の MCP サーバーを使うことを考えるときは、説明文の中身を注意して確認する必要があると感じた。
  - STEP 4 では、日本語の質問を、モデルが英語の検索語（`Amazon S3 versioning overview`）に直して AWS に送っていた。質問の内容は AWS に送られるが、送る文字列はモデルが決める。
  - STEP 3 のログで、fetch サーバーがページの前に `robots.txt` を取得し、転送（307）の先でも確認していた。講義資料の「robots.txt で禁止のページは取得しない」という既定の動作と合う（ログからの読み取りで、設定は確認していない）。
  - 外部サーバー（fetch は MCP のリポジトリ、AWS Knowledge は AWS）は、提供元と注意書きを確認してから足す。出所の分からないものは足さない（講義資料 p.16）。自分で読んだのは fetch の README だけで、AWS Knowledge の公式ページは読んでいない。
  - 503（モデルの混雑）が、STEP 2 で1回、STEP 4 で4回出た。どちらも再実行で成功した（STEP 2 は2回目、STEP 4 は5回目）。STEP 4 の AWS への接続自体は、503 の間も `[server] aws: URL に接続` まで進んでおり、503 の原因はモデル側だった。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎 / 実装 / 発展
- やったこと：
- 結果：
- 気づき・疑問：