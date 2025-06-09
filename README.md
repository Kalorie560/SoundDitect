# SoundDitect

リアルタイム音声異常検知AIアプリケーション / Real-time Sound Anomaly Detection AI Application

[🇯🇵 日本語](#日本語) | [🇺🇸 English](#english)

---

## 日本語

### 概要

SoundDitectは、PCのマイクからリアルタイムで入力される音声ストリームを分析し、衝撃音などの「異常音」を検知してユーザーに通知する高応答・高精度なWebアプリケーションです。

### 主な機能

- 🎤 **リアルタイム音声監視**: PCのマイクから1秒間隔で音声を分析
- 🧠 **AI異常検知**: 1D-CNN + Attention機構による高精度な異常音検知
- 💻 **Webベースインターフェース**: ブラウザで動作する直感的な日本語UI
- 📊 **リアルタイム可視化**: 音声波形とスペクトラムの実時間表示
- 📈 **実験管理**: ClearMLによる学習プロセスの記録・管理
- ⚡ **メモリ効率**: ストリーミング処理による大容量データ対応

### 技術選定理由

#### AI アーキテクチャ
- **1D-CNN with Attention**: 音声の時系列パターンと重要な特徴の両方を効果的に捉える
- **PyTorch**: 柔軟な モデル設計と実験に適している
- **ClearML**: 実験の再現性と結果管理に最適

#### バックエンド・フロントエンド
- **FastAPI**: 非同期処理とWebSocketに対応、高速なAPIサーバー
- **Web Audio API**: ブラウザから直接マイクアクセス、低遅延処理
- **WebSocket**: リアルタイム双方向通信で即座の結果通知

### セットアップと実行手順

#### 1. 環境準備

```bash
# リポジトリをクローン
git clone https://github.com/Kalorie560/SoundDitect.git
cd SoundDitect

# 仮想環境を作成・有効化
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# pipを最新バージョンにアップグレード（重要）
pip install --upgrade pip

# 依存関係をインストール
pip install -r requirements.txt
```

#### 2. ClearML設定（必須）

```bash
# ClearMLを設定（以下のいずれかの方法を選択）

# 方法1: 自動設定スクリプトを使用（推奨）
python scripts/setup_clearml.py

# 方法2: 手動設定
clearml-init

# 方法3: 環境変数で設定
export CLEARML_WEB_HOST=https://app.clear.ml
export CLEARML_API_HOST=https://api.clear.ml
export CLEARML_FILES_HOST=https://files.clear.ml
export CLEARML_API_ACCESS_KEY=your_access_key_here
export CLEARML_API_SECRET_KEY=your_secret_key_here
```

> 🌟 **ClearMLアカウント**: [https://app.clear.ml](https://app.clear.ml) でアカウント作成後、プロフィールページから認証情報を取得してください。

#### 3. サンプルデータ生成（オプション）

```bash
# サンプル学習データを生成（すべての設定はconfig.yamlから読み込まれます）
python scripts/generate_sample_data.py
```

#### 4. モデル学習（オプション）

```bash
# モデルを学習（すべての設定はconfig.yamlから読み込まれます）
python scripts/train_model.py
```

#### 5. サーバー起動

```bash
# サーバーを起動
python run_server.py
```

#### 6. アプリケーション使用

1. ブラウザで `http://localhost:8000` にアクセス
2. マイクアクセスを許可
3. 「録音開始」ボタンをクリック
4. リアルタイムで音声異常検知結果を確認

### プロジェクト構造

```
SoundDitect/
├── config.yaml              # 設定ファイル
├── requirements.txt          # Python依存関係
├── run_server.py            # サーバー起動スクリプト
├── backend/                 # バックエンドコード
│   ├── main.py             # FastAPIアプリケーション
│   ├── audio_processor.py  # 音声処理
│   ├── model_manager.py    # AIモデル管理
│   └── websocket_manager.py # WebSocket管理
├── frontend/                # フロントエンドコード
│   ├── index.html          # メインUI
│   ├── styles.css          # スタイルシート
│   ├── audio-processor.js  # 音声処理（クライアント）
│   ├── websocket-client.js # WebSocket通信
│   ├── ui-controller.js    # UI制御
│   └── app.js              # メインアプリケーション
├── scripts/                 # ユーティリティスクリプト
│   ├── train_model.py      # モデル学習
│   ├── generate_sample_data.py # サンプルデータ生成
│   └── setup_clearml.py    # ClearML設定スクリプト
├── models/                  # 学習済みモデル保存先
├── data/                    # 学習データ
├── logs/                    # ログファイル
└── outputs/                 # 実験結果
```

### データ仕様

学習データは以下のJSON形式で提供されます：

```json
[
  {
    "Waveform": [0.1, -0.2, 0.3, ...],  // 44100個の音声サンプル（1秒分）
    "Labels": 0  // 0: 正常, 1: 異常
  }
]
```

### アーキテクチャ解説

#### AIモデル
1. **前処理**: バンドパスフィルタ + 正規化
2. **1D-CNN**: 時系列パターンの特徴抽出
3. **Multi-Head Attention**: 重要な時間フレームに注目
4. **分類器**: 正常/異常の2クラス分類

#### リアルタイム処理
1. **音声キャプチャ**: Web Audio APIで44.1kHzサンプリング
2. **前処理**: ノイズ除去とフィルタリング
3. **WebSocket送信**: Base64エンコードで音声データ転送
4. **AI推論**: サーバーサイドでモデル推論
5. **結果表示**: 1秒以内にUI更新

### ClearML実験管理

```bash
# ClearML設定（初回のみ）
clearml-init

# 実験の可視化
clearml-serving --open
```

### カスタマイズ

`config.yaml`ファイルを編集して以下をカスタマイズできます：
- モデルアーキテクチャパラメータ
- 学習設定（エポック数、バッチサイズなど）
- 音声処理設定（サンプリング周波数、フィルタ設定）
- ClearMLプロジェクト設定

---

## English

### Overview

SoundDitect is a high-responsiveness, high-precision web application that analyzes real-time audio streams from PC microphones to detect "anomalous sounds" such as impact sounds and notify users.

### Key Features

- 🎤 **Real-time Audio Monitoring**: Analyzes audio from PC microphone at 1-second intervals
- 🧠 **AI Anomaly Detection**: High-precision anomalous sound detection using 1D-CNN + Attention mechanism
- 💻 **Web-based Interface**: Intuitive Japanese UI running in browsers
- 📊 **Real-time Visualization**: Real-time display of audio waveforms and spectrums
- 📈 **Experiment Management**: Recording and management of learning processes with ClearML
- ⚡ **Memory Efficiency**: Streaming processing for large-scale data handling

### Technical Design Decisions

#### AI Architecture
- **1D-CNN with Attention**: Effectively captures both temporal patterns and important features in audio
- **PyTorch**: Suitable for flexible model design and experimentation
- **ClearML**: Optimal for experiment reproducibility and result management

#### Backend/Frontend
- **FastAPI**: Supports asynchronous processing and WebSocket, high-speed API server
- **Web Audio API**: Direct microphone access from browser, low-latency processing
- **WebSocket**: Real-time bidirectional communication for immediate result notification

### Setup and Execution

#### 1. Environment Setup

```bash
# Clone repository
git clone https://github.com/Kalorie560/SoundDitect.git
cd SoundDitect

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Upgrade pip to latest version (important)
pip install --upgrade pip

# Install dependencies
pip install -r requirements.txt
```

#### 2. ClearML Setup (Required)

```bash
# Configure ClearML (choose one of the following methods)

# Method 1: Use automatic setup script (recommended)
python scripts/setup_clearml.py

# Method 2: Manual setup
clearml-init

# Method 3: Use environment variables
export CLEARML_WEB_HOST=https://app.clear.ml
export CLEARML_API_HOST=https://api.clear.ml
export CLEARML_FILES_HOST=https://files.clear.ml
export CLEARML_API_ACCESS_KEY=your_access_key_here
export CLEARML_API_SECRET_KEY=your_secret_key_here
```

> 🌟 **ClearML Account**: Create an account at [https://app.clear.ml](https://app.clear.ml) and obtain credentials from your profile page.

#### 3. Generate Sample Data (Optional)

```bash
# Generate sample training data (all settings are read from config.yaml)
python scripts/generate_sample_data.py
```

#### 4. Train Model (Optional)

```bash
# Train model (all settings are read from config.yaml)
python scripts/train_model.py
```

#### 5. Start Server

```bash
# Start server
python run_server.py
```

#### 6. Use Application

1. Access `http://localhost:8000` in browser
2. Allow microphone access
3. Click "録音開始" (Start Recording) button
4. Check real-time audio anomaly detection results

### Project Structure

```
SoundDitect/
├── config.yaml              # Configuration file
├── requirements.txt          # Python dependencies
├── run_server.py            # Server startup script
├── backend/                 # Backend code
│   ├── main.py             # FastAPI application
│   ├── audio_processor.py  # Audio processing
│   ├── model_manager.py    # AI model management
│   └── websocket_manager.py # WebSocket management
├── frontend/                # Frontend code
│   ├── index.html          # Main UI
│   ├── styles.css          # Stylesheets
│   ├── audio-processor.js  # Audio processing (client)
│   ├── websocket-client.js # WebSocket communication
│   ├── ui-controller.js    # UI control
│   └── app.js              # Main application
├── scripts/                 # Utility scripts
│   ├── train_model.py      # Model training
│   ├── generate_sample_data.py # Sample data generation
│   └── setup_clearml.py    # ClearML setup script
├── models/                  # Trained model storage
├── data/                    # Training data
├── logs/                    # Log files
└── outputs/                 # Experiment results
```

### Data Specification

Training data is provided in the following JSON format:

```json
[
  {
    "Waveform": [0.1, -0.2, 0.3, ...],  // 44100 audio samples (1 second)
    "Labels": 0  // 0: Normal, 1: Anomaly
  }
]
```

### Architecture Overview

#### AI Model
1. **Preprocessing**: Bandpass filter + normalization
2. **1D-CNN**: Feature extraction of temporal patterns
3. **Multi-Head Attention**: Focus on important time frames
4. **Classifier**: Binary classification (normal/anomaly)

#### Real-time Processing
1. **Audio Capture**: Web Audio API with 44.1kHz sampling
2. **Preprocessing**: Noise removal and filtering
3. **WebSocket Transmission**: Base64-encoded audio data transfer
4. **AI Inference**: Server-side model inference
5. **Result Display**: UI update within 1 second

### ClearML Experiment Management

```bash
# Configure ClearML (first time only)
clearml-init

# Visualize experiments
clearml-serving --open
```

### Customization

Edit the `config.yaml` file to customize:
- Model architecture parameters
- Training settings (epochs, batch size, etc.)
- Audio processing settings (sampling frequency, filter settings)
- ClearML project settings

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.