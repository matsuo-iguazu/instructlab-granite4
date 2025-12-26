# 学び 06: SDG（合成データ生成）に向けた Config の最適化

## 状況
- `ilab model chat` で Granite 4.0 Nano (1B) との対話に成功。
- 次のステップである `ilab generate` を実行した際、デフォルトの Teacher モデル（7B）を参照しようとしてエラーが発生。

## 課題と解決策
- **課題**: `generate` セクションの `teacher` 設定が外部 API や未ロードの 7B モデルを向いていた。
- **解決策**: `config.yaml` を「Chat 成功時の安定版」にロールバックし、その上で `teacher` の `model` パスと `backend` (llama-cpp) を手動で Nano に固定。

## 技術的知見
- `ilab` のエコシステムでは、生徒役 (Model) と先生役 (Teacher) が分かれている。
- 16GB RAM の制限下では、Nano に「一人二役（セルフ・インストラクション）」をさせる設定変更が不可欠。
- `teacher.backend` を `vllm` から `llama-cpp` に切り替えることで、GGUF モデルを Teacher として利用可能にした。

## 現在のステータス
- Taxonomy: 完全準拠（5x3 Q&A 構造）
- Config: Nano (1B) 特化型に最適化済み
- 次回：この安定した設定を基に、冬休み明けに大規模なデータ生成を実施する。
