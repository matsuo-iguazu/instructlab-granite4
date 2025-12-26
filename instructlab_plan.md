# InstructLab on ThinkPad 実装プロジェクト・スナップショット

## 1. プロジェクトの目的
高価なGPUサーバーを使わず、手元のビジネスノートPC（ThinkPad）のリソースを最大限に活用し、InstructLabを用いて「Granite 4.0」モデルの微調整（Fine-tuning）を完走させる。

## 2. ターゲット環境（Hardware & OS）
- **Device:** Lenovo ThinkPad
- **CPU:** Intel Core i5-1335U (13th Gen, 1.30 GHz)
- **RAM:** 16.0 GB
- **OS:** Windows 11 Pro (WSL2 / Ubuntu 必須)

## 3. 「スリム化」戦略（成功のための3本柱）
低リソース環境を突破するための主要戦略：

### ① モデルの選定（Student Model）
- **Granite 4.0 Nano (1B前後)** または **Micro (3B)** を採用。
- 従来の7B/8Bモデルに比べ、メモリ消費量を劇的に抑え（VRAM/RAM数GB〜8GB程度）、CPUトレーニングの現実味を高める。

### ② 外部APIの活用（Teacher Model）
- 最も重い処理である「合成データ生成（SDG）」において、Teacherモデルをローカルで動かさず、外部API（OpenAI互換API等）にバイパスする。
- ThinkPadのリソースを「学習（Training）」だけに集中させる。

### ③ CPUトレーニングの実行
- GPUがないため、`ilab model train --device cpu` フラグを使用。
- メモリ不足対策として、WSL2側で十分な **Swap領域（20GB以上推奨）** を確保する。

## 4. 実装ロードマップ（冬休みの宿題）
1. **WSL2環境構築:** UbuntuのインストールとPython環境の整備。
2. **ilabインストール:** `pip install instructlab` と `ilab config init`。
3. **タクソノミー作成:** `taxonomy/` フォルダに独自の知識やスキルを記述（YAML形式）。
4. **データ生成 (SDG):** API経由で少量のシードから大量の学習データを生成。
5. **トレーニング:** CPUを回してモデルを微調整。
6. **テスト:** 自分の教えた知識をモデルが答えるか確認。

## 5. 期待される成果
- 低リソース（i5-Uシリーズ/16GB RAM）環境におけるGranite 4.0学習のベンチマーク。
- 「個人PCでAIは育てられる」という実証。
