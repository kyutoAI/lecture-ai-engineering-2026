# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'search_phrase': 'Amazon S3 バージョニング 概要'}) / aws_aws___search_documentation({'search_phrase': 'What is S3 Versioning Amazon Simple Storage Service'})
- 気づき・疑問：エージェント側のコードを変えずに設定ファイル（app/mcp_servers.json）へ stdio（command）と Streamable HTTP（url）のサーバーを足すだけでツールが増えることを確認した。一方、接続サーバーが増えるにつれて入力トークン数が STEP 2 の 715 tokens（1ツール・2応答）→ STEP 3 の 2,230 tokens（2ツール・2応答）→ STEP 4 の 11,349 tokens（7ツール・3応答）へ急増し、外部サーバーの説明文・スキーマの長さと再検索の往復がトークン消費に直接影響することが分かった。また、外部サーバーの説明文がそのままモデルに渡り、HTTP サーバーには検索クエリが送信されるため、提供元と既定の動作（robots.txt 遵守やプライベート IP 拒否など）の事前確認が必要だと確認した。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎
- やったこと：エージェント側（`app/mcp_servers.json`・`app/mcp_agent.py`）を変更せず、サーバー側の `app/mcp_server.py` に `server.tool()(tools.get_lecture_summary)` を1行追加した。追加後に `git status`、`--list-tools`、`--trace "第2回では何を学びますか？"`、`pytest -q` を順に実行して動作を確認した。
- 結果：
  - `git status` の変更対象は `app/mcp_server.py` のみ。
  - `--list-tools` でモデルに見えているツール数が 7 個から 8 個に増え、`glossary_get_lecture_summary`（引数: `lecture: integer`）が表示された。
  - `--trace "第2回では何を学びますか？"` で `[tool call] glossary_get_lecture_summary({'lecture': 2})` が呼ばれ、第2回の要点をもとに回答が返った（入力 5,942 tokens / 出力 218 tokens / モデル応答 2 回）。
  - `pytest -q` は 22 passed で全件通過した。
- 気づき・疑問：サーバー側に1行足すだけでエージェント側を触らずに新ツールが認識・呼び出しされ、起動のたびにツール一覧を取得する仕組みが腹落ちしました。一方、第2回の内容を聞くだけの質問でも、ほかの外部サーバーをつないだままだと用語集サーバーだけのときより入力トークンが約8倍（715から5,942）に増えており、使わないサーバーを常時つなぐと無関係な質問でも毎回ツール説明分のコストがかかると実感しました。外部サーバーは説明文がそのまま渡り検索内容も接続先に送られるため、手軽に追加できるぶん提供元や既定の動作を事前に確認する必要があると感じました。
