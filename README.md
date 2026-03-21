# ⚾ Baseball SSW Simulator

> **Seam-Shifted Wake（SSW）をリアルタイムで可視化・体感できるインタラクティブシミュレーター**

MLB硬式球の3D回転モデルに、マグヌス力とシーム効果（SSW）の物理エンジンを統合。  
ドラッグ操作でボールを回転させ、縫い目の配置がピッチの変化量にどう影響するかを直感的に理解できます。

![React](https://img.shields.io/badge/React_18-61DAFB?style=flat-square&logo=react&logoColor=black)
![Canvas 2D](https://img.shields.io/badge/Canvas_2D-FF6600?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Zero Dependencies](https://img.shields.io/badge/Dependencies-0_(runtime)-blue?style=flat-square)

---

## 特徴

### 3D ボールレンダリング
- Thompson (1998) の球面パラメトリック方程式に基づく**精密なシームカーブ**
- 108本のステッチを個別にモデリング（交差縫い2列 × 4アーク × 27ステッチ/アーク）
- 革テクスチャ（プロシージャルノイズ生成）、球面シェーディング、スペキュラハイライト

### SSW（Seam-Shifted Wake）物理エンジン
- UMBA 2.0 準拠の**アクティブゾーン判定**（ヘミスフィア平面 ±60° 帯）
- 投手側前面の重み付け：Smith & Smith (2021) の知見を反映
- 回転に応じてリアルタイムで SSW 力の方向と大きさを計算

### RK4 弾道積分
- Nathan (2012) の揚力係数モデル：`C_L = 0.40√S / (1 + 0.80√S)`
- 4次ルンゲ＝クッタ法による投球軌道のフル積分（dt = 4ms、約112ステップ）
- マグヌス変化量、SSW 変化量、合計変化量を分離表示

### インタラクティブ操作
- **ドラッグ回転**：マウスまたはタッチでボールを任意方向にスピン
- **2本指ツイスト**：ジャイロ成分を直感的に操作
- **ピッチャー/キャッチャー視点切替**
- **リアルタイムメトリクス**：RPM、Tilt（時計表記）、Efficiency、True Spin / Gyro Spin

---

## デモ

### ブラウザで即座に試す（HTMLファイル1枚）

`baseball-ssw-simulator-v8.html` をダウンロードしてダブルクリックするだけ。  
サーバー不要、インストール不要です。

### 開発サーバーで起動

```bash
git clone https://github.com/YOUR_USERNAME/baseball-ssw-simulator.git
cd baseball-ssw-simulator
npm install
npm run dev
```

→ `http://localhost:5173` で起動します。

---

## プロジェクト構成

```
baseball-ssw-simulator/
├── index.html                          ← エントリHTML
├── package.json
├── vite.config.js                      ← Vite設定（相対パス出力対応）
├── README.md
├── src/
│   ├── main.jsx                        ← React マウント
│   └── BaseballSSW.jsx                 ← シミュレーター本体（単一ファイル）
└── baseball-ssw-simulator-v8.html      ← スタンドアロン版（HTML1枚で動作）
```

## 技術スタック

| 項目 | 詳細 |
|------|------|
| UI | React 18（Hooks のみ） |
| 描画 | Canvas 2D API |
| ビルド | Vite |
| ランタイム依存 | **なし**（React 本体のみ） |
| 物理モデル | RK4積分 + Nathan (2012) C_L + UMBA 2.0 SSW |

---

## 操作方法

| 操作 | 効果 |
|------|------|
| ドラッグ（1本指 / マウス） | ボールを回転させる。速度と方向がスピン軸になる |
| 2本指ツイスト | ジャイロ成分を調整（0°= ピュアバックスピン、90°= ピュアジャイロ） |
| View ボタン | ピッチャー視点 ↔ キャッチャー視点 |
| Speed スライダー | 投球速度（60–105 mph） |
| RPM× スライダー | 回転数の倍率（1.0–10.0×） |
| Decay スライダー | スピン減衰率（0 = 無摩擦、0.020 = 高摩擦） |
| 数値タップ | inch ↔ cm / mph ↔ km/h 単位切替 |

---

## 表示メトリクス

### 上段パネル
- **RPM** — 現在の総回転数（= 表示rpm × RPM倍率）
- **Efficiency** — スピン効率 = cos(gyro°) × 100%
- **Gyro Deg** — ジャイロ角度（0°= 純回転、90°= 純ジャイロ）
- **True Spin / Gyro Spin** — 有効回転と非有効回転の内訳
- **Tilt** — Rapsodo 定義の時計表記（マグヌス力の方向）

### 変化量パネル（Break）
- **V Break** — 垂直変化量（▲ ride / ▼ drop）
- **H Break** — 水平変化量（→ arm side / ← glove side）
- **Magnus** — マグヌス力による変化量の内訳
- **SSW** — シーム効果による変化量の内訳 + Active zone 比率
- **Total Σ** — 合計変化量 + 方向矢印

---

## 物理モデルの詳細

### シームカーブ（Thompson 1998）

MLB 公式球のパラメータ（周囲長 C₀ = 9.25 in、シーム幅 S = 1.25 in）から球面上の縫い目曲線を解析的に構成。4 つのアークに分割し、各アーク上に等弧長で 27 本のステッチを配置します。

### SSW 力の計算（UMBA 2.0 準拠）

1. 全 216 ステッチの代表点を現在の回転行列で変換
2. ヘミスフィア平面（速度ベクトルに垂直）から ±60° の帯域内にある点を「アクティブ」と判定
3. 投手側（front）は最大 1.6 倍、捕手側（back）は 0.6 倍の重み付け
4. 重み付き重心ベクトルの方向と大きさから SSW 力を導出

### 弾道積分（RK4）

以下の 3 軌道を並列で積分し、差分から各変化量を算出します：

| 軌道 | 含む力 | 用途 |
|------|--------|------|
| 基準軌道 | 抗力 + 重力 | ゼロ点（ナックルボール相当） |
| Magnus 軌道 | 抗力 + 重力 + マグヌス揚力 | Magnus 変化量 |
| Total 軌道 | 抗力 + 重力 + マグヌス揚力 + SSW | Total 変化量 |

物理定数は MLB 標準環境（海抜 0m、気温 20°C、ρ_air = 1.204 kg/m³）を使用。

### 揚力係数（Nathan 2012）

```
C_L = 0.40 √S / (1 + 0.80 √S)
S   = r · ω / v   （スピンパラメータ）
```

90 mph / 2200 rpm で Magnus ≈ +16.4 in（実測 14–16 in に合致）。

---

## ビルドと配布

### 静的HTMLとしてビルド

```bash
npm run build
```

`dist/` フォルダに静的ファイルが生成されます。任意の Web サーバーに配置、または `dist/index.html` を直接開けます。

### スタンドアロン HTML の生成

React と Babel を CDN から読み込む単一 HTML ファイルも同梱しています。  
サーバー不要で `file://` プロトコルでも動作します。

---

## 参考文献

- Thompson, J.C. (1998). *Designing a Baseball Cover.* The College Mathematics Journal, 29(1), 48–61.
- Nathan, A.M. (2012). *Determining Pitch Movement from PITCHf/x Data.*
- Smith, A.J. & Smith, J. (2021). *Seam Shifted Wake and Its Impact on Pitch Movement.*
- UMBA 2.0 — *Understanding Modern Baseball Analytics: SSW Framework.*
- Box, G.E.P. — *"All models are wrong, but some are useful."*

---

## ライセンス

MIT License

---

> *"All models are wrong, but some are useful."* — G.E.P. Box
