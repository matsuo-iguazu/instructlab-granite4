# 学び 02: ハードウェアプロファイルと設定のポータビリティ

## 状況
- `ilab config init` にて、ThinkPad (Intel CPU) に最適化されたプロファイルを選択。
- 設定ファイルがホームディレクトリ配下の隠しフォルダに生成された。

## 選択したプロファイル
- **Vendor:** [3] INTEL
- **Configuration:** [1] INTEL CPU
- **結果:** `intel/cpu.yaml` がベースとして適用された。これにより、Intel CPU上での推論効率が向上する設定が読み込まれる。

## 工夫した点
- **設定のプロジェクト化:** デフォルトの `~/.config/instructlab/config.yaml` をリポジトリ `~/instructlab-granite4/config/` 内に移動。
- **理由:** 1. 学習パラメータやAPI設定の変更履歴をGitで管理するため。
  2. 他の環境に移行した際、同じ設定を即座に再現するため。

## 次の課題
- `config.yaml` 内のモデルパスを、リポジトリ内の `models/` フォルダを指すように調整する。
- 外部API（Teacher Model）を利用するための endpoint 設定。
