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

- 取り組んだレベル：基礎
- やったこと：`app/mcp_server.py` の `# TODO(宿題・基礎)` の行の下に、`server.tool()(tools.get_lecture_summary)` を1行書き足した。エージェント側（`app/mcp_servers.json`、`app/mcp_agent.py`）は変えていない。最初は TODO の下の空行に書いたため、`git diff` に削除が1行出た。空行を戻して、追加1行・削除なしに直した。実行前の予想は、ツールが8個、名前が `glossary_get_lecture_summary`、質問で呼ばれる、の3つ。
- 結果：
  - `git status`：変更は `app/mcp_server.py` だけ。
  - `--list-tools`：ツールが7個から8個になり、`[glossary_get_lecture_summary]` が増えた。引数は `{"lecture": {"title": "Lecture", "type": "integer"}}`、説明文は `app/tools.py` の docstring がそのまま出た。
  - `--trace "第2回では何を学びますか？"`：`[tool call]   glossary_get_lecture_summary({'lecture': 2})`、`[usage] 入力 5,942 tokens / 出力 216 tokens / モデル応答 2 回`。回答は第2回「ファインチューニングとLoRA」の内容だった。回答にあった「ハイパーパラメータ（ランク r・学習率・エポック数）」「QLoRA との関係」「架空 FAQ のデータで LoRA 学習し、JSON 形式の遵守率を比べる演習」は、`grep` で `data/lectures/02.md` の13・14・19行目にあることを確認した。
  - `uv run pytest -q`：21 passed。
  - 503（モデルの混雑）：`--trace` が約18分の間に連続8回、503 で失敗した。Codespaces を停止して間をあけ、再開後の1回目で成功した。待つ間に、`get_lecture_summary(2)` を API を使わず直接呼び、`data/lectures/02.md` の内容が返ることを確認した。
- 気づき・疑問：
  - 予想は3つとも当たった。ツールは `server.tool()(...)` の1行で増え、エージェント側を変えなくても使えた。エージェントが起動のたびに、ツールの一覧をサーバーに問い合わせるため。
  - 入力は、STEP 2（ツール1個）の 715 tokens に対して 5,942 tokens だった。内訳を調べるため、ツールを呼ばない質問「こんにちは」（モデル応答1回）を1回測ると、入力は 2,689 tokens だった。質問の分を除いた、ツール8個の定義や指示文などだけで、1回の呼び出しに約 2,700 tokens かかる計算になる。今回は呼び出しが2回あったので、2,689 × 2 = 5,378 tokens（5,942 の約9割）がその分で、残りの約 560 tokens がツールの呼び出しと返り値の目安。実行前の予想は「ツール定義が5割くらい」で、外れた。質問の長さの違いなどを含む目安で、1回しか測っておらず、定義だけの正確な内訳は測っていない。ツールを足すほど、使わない質問でも毎回の入力が増える（講義資料 p.9 の「定義の分を毎回消費」と合う）。
  - `app/tools.py` の `get_lecture_summary` は、ファイルが無い回には、例外ではなく「第N回の要点はまだありません。用意されている回: …」という文字列を return する作りになっていた（80行目）。