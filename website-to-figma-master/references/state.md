# 作業状態・更新・再実行

## 保存する状態

`website-master.config.json`は対象と方針、`website-master.state.json`は実際に確認・作成した結果を持つ。会話の記憶だけに依存しない。

stateには次を含める。

- schemaVersion、projectId、runId、status、実際のFigma file key。
- sources：URL、取得日時、viewport、状態、source ID、証拠の保存先、未取得理由。
- foundations：値、型、出典、observed/estimated/derived/unknown、Figma ID。
- components：sourceのrole、component/variant/propertyの対応、Figma ID。
- screens：URL・幅・状態に対応するFrame IDと検証状態。
- operations：処理キー、予定対象、開始／終了、作成・変更したID、エラー。
- runs：変更概要、before/after、検証結果、残る差。
- snapshots：対象年月、実取得日時、元masterの更新日時、scope、frame ID、画像と値の保存先、検証状態、未保存理由。
- scheduler：定期実行の実際のtask ID、設定したタイムゾーンと時刻、最終成功日時。未設定・失敗を成功扱いしない。

値を知らない項目はnullまたはunknownとする。偽のIDや仮URLを実値として保存しない。

## 再実行

ファイル＋ページ＋source＋viewport＋state＋部品の役割を基に安定した処理キーを作る。IDを再取得して存在と型を確認し、完了済み処理を飛ばす。Plugin APIを使える場合、自分が作ったノードにproject IDと処理キーをplugin dataとして持たせる。

API応答が途切れた場合は、先にノードと記録を読み、書込みが済んだか確認する。名前が似ているだけのノードを更新しない。所有の判別ができない既存ノードは触らず、紐付けが必要な箇所だけ確認する。

## 更新前の固定

更新の比較条件（URL・幅・状態・言語等）を前回に揃える。記事内容、広告、日時、個人化の変化と、UIそのものの変化を分ける。

1. 変更される部品と、その部品・変数・スタイルを使う画面を調べる。tokenを変える場合は利用先全体を対象に含める。
2. 影響する画面・部品を更新履歴へ複製する。ネストしたInstanceを含めて独立させ、実効値を保存してvariablesのbindingを外し、共通styleの参照も切る。利用可能なAPIで固定し切れないなら画像の書出しをbeforeの証拠として残し、その制約を記す。
3. 画像の書出し、スタイル値、プロパティ値、元IDも保存する。画像は履歴の証拠であり、編集可能なScreensの代用ではない。
4. 更新後にbefore画像と保存値を照合し、過去が動いていないことを確認する。

初回はbeforeが無いので「初回作成」と記録する。過去回の記録は上書きしない。同じrunの再開は同じ枠に追記する。

## 既存物の扱い

ユーザーが配置したものを勝手に移動・改名しない。空きを確認して追加し、必要な範囲だけセクションを広げる。新しく作ったノードが失敗した場合、自分のrunで作成したと確認できるものだけ片付ける。

役割が同じ部品の観察済みの修正は、beforeを固定した後で更新できる。併存する形や挙動の違いはvariant追加または別部品にする。旧版を残す必要があるものは`old_`等の設定接頭辞を付け、いつ・なぜ旧版になったかを書く。
