# no01 — 入力を振り分けて表に残す

フォームで受けた相談を「不具合かどうか」で2つの対応レーンに分け、n8n 内蔵の Data Table に1行ずつ残すワークフローです。
外部サービスの認証情報（Credential）は使いません。

- 確認した版: self-hosted n8n 2.41.6（Node.js 24.21.0）・2026-10-03〜04
- ファイル: [`n8n-basics-form-routing.n8n.json`](n8n-basics-form-routing.n8n.json)

## 構成（5つの箱）

| 箱 | 種類 | 役割 |
|---|---|---|
| 相談を受け取る | Form Trigger | `件名`（テキスト）・`種別`（`質問` / `不具合` の選択）・`本文`（複数行）。3項目とも必須 |
| 不具合か？ | If | `種別` が `不具合` なら true、それ以外は false |
| 不具合確認レーン | Edit Fields | true 側。`対応レーン=不具合確認` を足す（元の項目は残す） |
| 通常受付レーン | Edit Fields | false 側。`対応レーン=通常受付` を足す（元の項目は残す） |
| 受付表に追加 | Data Table | Row / Insert / By Name で `n8n_basics_20261003_intake` に1行追加 |

## 手順

前提: n8n のエディタを開ける状態であること。始め方（Cloud か self-host か）は公式の案内を見てください: https://docs.n8n.io/get-started/choose-how-to-use-n8n

1. **表を先に作る。** Data tables の画面で、名前 `n8n_basics_20261003_intake` の表を作り、String 型の列 `subject`・`category`・`body`・`lane` の4つを足す（JSON を読み込んでも表は作られません。列名は英字のまま）。
2. **ワークフローを読み込む。** New workflow を開き、ワークフローのメニューの Import from File → `n8n-basics-form-routing.n8n.json` を選ぶ。
3. **待ち受けを始める。** エディタ下部の `Execute workflow` を押す（フォーム箱の `Execute step` ではありません）。
4. **1通送る。** テスト用フォームが開くので、種別を `質問` にして3項目を入れて送信する。
5. **結果を見る。** Data tables の表に、件名・種別・本文と `lane=通常受付` の1行が入っていれば成功。種別を `不具合` にして送ると `lane=不具合確認` の行が増えます。
6. **記録を見る。** 画面上部の Executions から、それぞれの実行で通った線と、箱ごとの入力・出力を確かめられます。

## 補足

- Publish / Activate や本番用 URL（Production URL）は使っていません。テスト用 URL だけで動きます。
- 外部サービスへつなぐ場合や Cloud を使う場合の料金・制限は、n8n 公式の最新の案内を確認してください。
- 言い回しの違う文章を意味で振り分けたい場合は、If の位置に AI 判断の箱を置き換えます。その実演は同チャンネルの JEV 第3回で扱っています。
