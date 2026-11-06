# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
  - 第1回の `app.ask`（素のLLM）では、AIE2026固有の「宿題の3レベル」を知らないため、一般的な講座の例として「Level 1（基礎）・Level 2（応用）・Level 3（発展）」といった推測の回答をしていた。
  - 一方、今回のエージェント（`app.agent`）は回答前に `search_glossary` で用語集を検索するため、「基礎（講義内容の確認）・実装（自分でコードを書く）・発展（調査や試行）の3段階があり、どれか1つに取り組めば完了（原則採点なし）」という講座の正確な定義に基づいて回答できた。また、`INSTRUCTIONS` の指示通り末尾に「（関連する回：第1回）」も表示された。
- `--trace` で見えたツール呼び出し（ツール名と引数）：
  - 「AIE2026の講座の宿題にある『3レベル』とは何ですか？」のとき：
    - `search_glossary({'query': '3レベル'})`
  - 「LoRAとQLoRAの違いは？」のとき：
    - `search_glossary({'query': 'LoRA'})`
    - `search_glossary({'query': 'QLoRA'})`
    - （`search_glossary` の docstring に「query には調べたい用語を1つだけ入れる」とあるため、1つずつ計2回に分けて呼ばれた）
  - 「第2回の内容から確認問題を1問出して」のとき：
    - `make_quiz({'lecture': 2})`
- 1つの質問での LLM 呼び出し回数（`requests`）：
  - `requests=2`（1回目でどのツールをどの引数で呼ぶかを判断し、ツールの実行結果を受け取った2回目で最終的な回答文を生成しているため）
- 追加したツールと、docstring に書いた説明：
  - 追加したツール：`make_quiz(lecture: int) -> str`（指定した回の用語集からランダムに1つ用語を選び、問題と定義のJSON文字列を返す。該当する回の用語がない場合はメッセージを返す）
  - docstring に書いた説明：
    ```text
    指定した回（1始まりの整数）の用語集から確認問題を1問作って返す。

    「第2回の確認問題を出して」「クイズを出して」のように、確認問題やクイズの
    出題を求められたときに使う。
    ```

## 宿題

- 取り組んだレベル：発展
- やったこと：
  - 発展(a)として、演習で作ったエージェントと同じ `INSTRUCTIONS` と3つのツール（`search_glossary`, `get_lecture_summary`, `make_quiz`）をそのまま使い、`adk_agent/glossary_agent/agent.py`（および `__init__.py`）を作成して Google の ADK（`google-adk`）でエージェントを組み立て直し、`PYTHONPATH=. uv run --with google-adk adk run adk_agent/glossary_agent` で演習と同じ3つの質問を実行して比較した。
- 結果：
  - **書き方の違い**：
    - Pydantic AI では `Agent(instructions=INSTRUCTIONS)` を作ったあと `agent.tool_plain(...)` でツールを登録し、実行時に `GoogleModel` を渡して `agent.run_sync()` を呼ぶ構成になっている（テスト時に `TestModel` に差し替えやすい）。
    - ADK では `root_agent = Agent(model="gemini-3.5-flash-lite", name="glossary_agent", instruction=INSTRUCTIONS, tools=[tools.search_glossary, tools.get_lecture_summary, tools.make_quiz])` のように、モデル名・指示（単数形の `instruction`）・Python関数のリストを1つのコンストラクタにまとめて渡し、`root_agent` という変数名で定義するだけで `adk run` から対話型のCLIとしてそのまま動かせた。
  - **回答の違い**：
    - 同じモデル（`gemini-3.5-flash-lite`）・同じ `INSTRUCTIONS`・同じツール関数を使っているため、「宿題の3レベル」「LoRAとQLoRAの違い」「第2回の確認問題」のいずれも、用語集の定義に基づいた正確な回答・末尾の「関連する回：第N回」の付与・確認問題で答えを直接言わずに出題する挙動まで Pydantic AI 版とほぼ同じ結果になった。違いとして、`adk run` は対話セッションが維持されるため、複数回の質問を連続してやり取りできる形になっていた。
- 気づき・疑問：
  - `app/tools.py` に書いた関数は普通の Python 関数（型ヒント＋docstring）なので、フレームワークを Pydantic AI から Google ADK に変えても、ツール側のコードや `INSTRUCTIONS` を1行も書き換えずにそのまま流用できるのがとても便利だと感じた。フレームワークが裏で関数の型と docstring を読み取って LLM のツール仕様に変換している仕組みがよく理解できた。
