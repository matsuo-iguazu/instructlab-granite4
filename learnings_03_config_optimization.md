# 学び 03: config.yaml の最適化と「スリム化戦略」の実装

## 状況
- `ilab config init` で生成されたデフォルトの `config.yaml` を解析。
- デフォルト設定の多くが RAM 64GB 以上の強力なマシン（NVIDIA GPU等）を想定した `pipeline: full` になっていることを確認。
- このままでは 16GB RAM の ThinkPad ではデータ生成や学習時にメモリ不足（OOM）でハングアップするリスクが高い。

## 最適化のポイント（スリム化の実装）

1. **Teacherモデルの外部切り出し (Generate)**
   - `teacher` 設定をローカルの `llama-cpp` から API 連携を想定した構成へ変更。
   - 目的: 最も重い「合成データ生成（SDG）」の計算負荷を ThinkPad から解放する。

2. **パイプラインの軽量化 (Simple Mode)**
   - `generate` および `train` の `pipeline` を `full` から `simple` へ変更。
   - 理由: `full` パイプラインは高度だがメモリ消費が激しいため、16GB 環境では `simple` が現実的な選択となる。

3. **パスの局所化 (Locality)**
   - モデル (`models/`)、チェックポイント (`checkpoint/`)、データセット (`datasets/`) のすべてのパスをリポジトリ `~/instructlab-granite4/` 内に集約。
   - 理由: OSの隠しフォルダ (`~/.cache/`) ではなく、プロジェクトディレクトリ内で完結させることで、Gitによる構成管理とバックアップを容易にする。

4. **学習パラメータの抑制**
   - `effective_batch_size` を 4 程度に抑制。
   - `num_epochs` を 3 程度に設定し、まずは「CPUで学習が回るか」の検証を優先。

## 結論
低スペックPCでAIを育てるには、**「デフォルトの推奨（Full/Auto）」を疑い、ハードウェアの限界に合わせて設定を一段階「Simple」に落とす勇気**が必要である。
