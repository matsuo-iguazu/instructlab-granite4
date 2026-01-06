# 学習メモ：InstructLab v0.25 (Skill v2) への回帰と検証方針

## 1. 調査結果：v0.25 採用の理由と懸念点
最新の v0.26.1 は依存ライブラリ（`docling` 等）のバージョン不整合により、2026年現在の環境では SDG プロセスが自爆することを確認。
「枯れた安定版」である v0.25 を採用し、確実に ThinkPad 上で 1b モデルを回す「実利」を優先する。

### 懸念される「落とし穴」と対策
- **モデルアーキテクチャの不整合**:
  - 0.25 世代の `llama-cpp-python` が最新の Granite 4.0 (NoPE等) を解釈できないリスク。
  - **対策**: インストール後、必要に応じて `llama-cpp-python` のみを強制アップグレードし、モデル認識を確認する。
- **リソース競合 (16GB RAM の攻防)**:
  - SDG（`ilab data generate`）時に教師モデルと生徒モデルがメモリを奪い合う。
  - **対策**: 教師モデルも軽量な Granite-4.0-1b の量子化版 (GGUF) を指定し、スワップ発生を抑止する。
- **Taxonomy 形式の厳守**:
  - v0.26 以降の Knowledge v3（フォルダ形式）は不可。
  - **対策**: `compositional_skills` 配下の `qna.yaml` (version: 2) による純粋な Skill v2 形式を貫く。

## 2. 環境再構築プロトコル (Python 3.11)
OS 標準の Python 3.11 を使用し、完全に独立した venv を構築する。

```bash
# 古い環境の完全排除
rm -rf venv
rm -rf ~/.cache/instructlab
rm -rf ~/.config/instructlab

# 再構築
python3.11 -m venv venv
source venv/bin/activate
pip install --upgrade pip setuptools wheel
pip install instructlab==0.25.0

## 4. GitHub 接続設定 (SSH)
HTTPS 認証の煩わしさを避けるため、リモートURLを SSH 形式へ変更。

```bash
git remote set-url origin git@github.com:matsuo-iguazu/instructlab-granite4.git

## 5. リソース管理 (.gitignore)
16GB RAM の ThinkPad での検証において、巨大なモデルファイルや学習中間データが GitHub 履歴を汚染するのを防ぐため、以下のディレクトリを Git 管理対象外に設定。

- `models/`: GGUFモデル等の格納先
- `generated/`: SDG (generate) プロセスの中間生成物
- `checkpoints/`: 学習時の重みデータ

## 5. リソース管理 (.gitignore)
16GB RAM の ThinkPad での検証において、巨大なモデルファイルや学習中間データが GitHub 履歴を汚染するのを防ぐため、以下のディレクトリを Git 管理対象外に設定。

- `models/`: GGUFモデル等の格納先
- `generated/`: SDG (generate) プロセスの中間生成物
- `checkpoints/`: 学習時の重みデータ

## 6. 環境構築完了報告 (2026-01-06)
- **Status**: Success
- **Version**: InstructLab v0.25.0
- **Python**: 3.11.x (venv)
- **Artifacts**: `ilab_version.txt`, `requirements_freeze.txt` をリポジトリに記録。

「更地」からの再構築により、v0.26 で発生していた依存関係の呪縛を完全に断ち切った。

## 7. モデルの再調達
環境の大掃除（rm -rf）に伴い、前回ダウンロード済みの GGUF ファイルも消失したため、再ダウンロードを実施。
- **Target**: Granite-4.0-1b-Chat (Q4_K_M)
- **Strategy**: 16GB RAM 環境での推論・学習の安定性を考慮し、4-bit 量子化版を選択。

## 8. モデル再取得と初期設定（手順修正）
`ilab` の仕様に従い、まず設定ファイルの初期化を行い、その後にモデルのダウンロードを実施。

1. `ilab config init`: 構成ファイルの生成。
2. `ilab model download`: Granite-4.0-1b-Q4_K_M.gguf の取得。

## 9. config init 時の設定値 (ThinkPad 最適化)
16GB RAM / Intel CPU 環境で完走させるための選択。

- **Model Path**: `/home/matsuo/.cache/instructlab/models/granite-4.0-1b-Q4_K_M.gguf` を明示。
- **Hardware Vendor**: `[3] INTEL`
- **Hardware Profile**: `[1] INTEL CPU` (非GPU環境での CPU 推論を確定)

## 10. WSL2 リソース制限設定（実設定値）
ThinkPadの安定稼働と、高負荷プロセス（SDG/Training）完走のため、以下の制限を適用中。

- **Config File**: `$env:USERPROFILE\.wslconfig`
- **設定値**:
  ```text
  [wsl2]
  processors=6
  memory=12GB
  swap=24GB

## 11. モデル配備完了
`ilab model download` が正常に終了。Granite-4.0-1b (Q4_K_M) が指定ディレクトリに配備された。

- **ファイルサイズ**: 976.2 MB
- **格納パス**: `/home/matsuo/.cache/instructlab/models/granite-4.0-1b-Q4_K_M.gguf`

1GB未満という軽量さは、WSL2に割り当てた 12GB RAM 環境において、推論・学習の両面で大きなアドバンテージとなる。
## 12. トラブルシューティング：ValueError (Failed to determine backend)
`ilab model serve` 実行時にバックエンド判定失敗のエラーが発生。

- **原因**: `config init` 時の入力ミスにより、`config.yaml` 内の `model_path` が不正な文字列（GGUFとして認識不可）になっていた。
- **対策**: `~/.config/instructlab/config.yaml` を手動修正し、正しい絶対パスを指定。

## 13. 初回推論成功 (ilab model chat)
Granite-4.0-1b-Q4_K_M の火入れに成功。

- **Response Time**: 4.938 seconds（想定以上に高速）
- **Output**: 
  > Hello! I am an AI language model developed by Red Hat and IBM Research. My name is Granite...
- **Observation**: 4.0-1b モデルだが、自己紹介では granite-3.0-8b と名乗る挙動を確認。推論自体は安定している。
## 14. フェーズ1（環境構築・動作確認）完了チェックリスト
- [x] v0.25.0 への完全移行とクリーンインストール
- [x] Intel CPU 向けハードウェアプロファイル適用
- [x] WSL2 リソース制限設定 (6CPU/12GB/24GB Swap)
- [x] Granite-4.0-1b-Q4_K_M での推論成功（レスポンス良好）

### 次フェーズの留意事項（Skills v2）
- `qna.yaml` は Skills v2 スキーマに従い作成すること。
- SDG (Data Generation) 時、メモリ溢れを防ぐため生成数を絞る設定を行う。


