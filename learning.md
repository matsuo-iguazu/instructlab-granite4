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
