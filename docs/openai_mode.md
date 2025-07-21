# OpenAI専用モード (Minimal App)

## 概要
OpenAI専用モードは、`minimal_app.py` と `minimal_shared.py`、および最小限の静的ファイルを用いて動作する軽量版です。他のプロバイダーやローカルのSTT/TTSは利用できず、OpenAI API前提でWeb公開を想定した構成になっています。

## 起動方法
```bash
uvicorn app.minimal_app:app --reload --host 0.0.0.0 --port 8000
```
Docker 利用時は以下の通りです。
```bash
docker build -t voice-chat-ai:latest -f Dockerfile.minimal .
docker run -d --env-file .env --name voice-chat-ai -p 8000:8000 voice-chat-ai:latest
```

## 処理の流れ
1. 音声のアップロードを `/api/transcribe` で受け取り [`transcribe_audio_bytes`](../app/minimal_shared.py#L143-L155) に渡して文字起こしを行います。
2. 得られたテキストを [`generate_response_text`](../app/minimal_shared.py#L157-L178) でLLMに送り会話文を生成します。
3. 返信文は [`synthesize_text`](../app/minimal_shared.py#L180-L194) で音声化され `/api/synthesize` から返却されます。

* **ダッシュボード**
  このモードでは利用できません。フル版のUIをご覧ください。

* **エンハンス (Enhanced)**
  **説明**: OpenAI TTSモデルを用いた対話画面。ユーザーがAPIキーを入力して利用します。

  **項目 & 選択肢**:

  1. **APIキー入力**

     * タイプ：`password`入力欄。
     * 説明：OpenAI APIキーを入力（任意）。
  2. **キャラクター** (`characterSelect`)

     * タイプ：`select`ドロップダウン。
     * ソース：`/characters` エンドポイントから取得されるキャラクターリスト。
     * 説明：対話中のキャラクターを選択。
  3. **音声認識（STT）** (`transcriptionModelSelect`)

     * タイプ：`select`ドロップダウン。
     * 選択肢：

       * `whisper-1`: OpenAI Whisper-1
       * `web-speech`: Web Speech API (クライアント)。ブラウザ依存で無料で使えるAPI
     * 説明：録音した音声を文字起こしする方法を選択。
  4. **チャットモデル (LLM)** (`modelSelect`)

     * タイプ：`select`ドロップダウン。
     * 選択肢：

       * `gpt-4o`: GPT-4o
       * `gpt-4o-mini`: GPT-4o Mini
       * `gpt-4`: GPT-4
     * 説明：対話に使用するLLMモデルを選択。
  5. **音声合成（TTS）** (`ttsModelSelect`)

     * タイプ：`select`ドロップダウン。
     * 選択肢：

       * `gpt-4o-mini-tts`: GPT-4o Mini TTS
       * `tts-1`: TTS-1
       * `tts-1-hd`: TTS-1 HD
       * `web-speech`: Web Speech API (クライアント)。ブラウザ依存で無料で使えるAPI
     * 説明：応答文を音声に合成する方法を選択。
  6. **ボイス** (`voiceSelect`)

     * タイプ：`select`ドロップダウン。
     * 選択肢：

       * `alloy`: Alloy - female
       * `echo`: Echo - male
       * `fable`: Fable - male
       * `onyx`: Onyx - male
       * `nova`: Nova - female
       * `shimmer`: Shimmer - female
       * `sage`: Sage - female
       * `coral`: Coral - female
       * `ash`: Ash - male
       * `ballad`: Ballad - male (no TTS-1)
       * `verse`: Verse - male (no TTS-1)
     * 説明：音声合成時の声のキャラクターを選択。

* **リアルタイム (Realtime)**
  **説明**: OpenAI Realtime APIを用いてWebRTCで双方向ストリーミング対話を行う画面。
  **項目**:

  * **APIキー入力**: OpenAI APIキーを入力（任意）。
  * **Session Status**: セッションの接続状態を表示するバッジ。
  * **Character**: 対話キャラクター選択。
  * **Voice**: 音声合成用の声を選択。
    **ボタン**:
  * **Start Session**: WebRTCセッションを開始。
  * **Stop Session**: セッションを停止。
  * **Clear Transcript**: トランスクリプトをクリア。
  * **Test Microphone**: マイクテストを実行。
    **表示要素**:
  * **Voice Visualization**: ユーザーとAIの音声レベルをバーグラフで表示。
  * **Transcript**: 発話の文字起こしを色分け表示。

* **スピーチテスト (Speech Test)**
  **説明**: ブラウザ標準の Web Speech API を利用し、音声認識と合成を簡易テストする画面。
  **項目**:

  * **Recognition Language**: 音声認識の対象言語を選択するドロップダウン。
  * **Synthesis Voice**: 合成音声に使用する声を選択するドロップダウン。
  * **Text Input**: 読み上げるテキストを入力するテキストエリア。
    **ボタン**:
  * **Start Recognition**: 音声認識を開始。
  * **Stop Recognition**: 音声認識を停止。
  * **Speak Text**: 入力テキストを音声で読み上げ。
  

## ファイル構成

以下のようなディレクトリ構成になっており、各ファイルはそれぞれの役割を担います。

```
Dockerfile.minimal
requirements-minimal.txt
.env
app
│
├─static
│  │  favicon.ico
│  ├─css
│  │    styles.css
│  ├─images
│  │    games.png
│  │    oregon.jpg
│  └─js
│       debug_panel.js
│       enhanced.js
│       scripts.js
│       speech_test.js
│       webrtc_realtime.js
│
└─templates
    │
    └─ja
         enhanced.html
         index.html
         speech_test.html
         webrtc_realtime.html
```

* **Dockerfile.minimal**: 最小限のコンテナイメージをビルドするためのDocker設定
* **requirements-minimal.txt**: `minimal_app.py` の実行に必要な最小限のPythonパッケージ一覧
* **.env**: 環境変数（OpenAI APIキーなど）を管理するファイル
* **app/static**: CSS、画像、JavaScriptなどの静的アセットを格納するフォルダ

  * **css/styles.css**: 画面の共通スタイル定義
  * **images/**: ダッシュボード用のサンプル画像
  * **js/**: 各画面の動作を担うスクリプト（Enhanced, Realtime, Speech Test）
* **app/templates/ja**: 日本語化用HTMLテンプレートを格納するフォルダ

  * **enhanced.html**: Enhanced画面の日本語版テンプレート
  * **index.html**: ダッシュボード日本語版テンプレート
  * **speech\_test.html**: スピーチテスト画面日本語版テンプレート
  * **webrtc\_realtime.html**: リアルタイム画面日本語版テンプレート



## API一覧

`minimal_app.py` では以下のエンドポイントが定義されています。

* `POST /api/transcribe`     : 音声バイト列を受け取り、文字起こしテキストを返します。
* `POST /api/chat`           : 文字起こしされたテキストをLLMに送り、チャット応答テキストを返します。
* `POST /api/synthesize`     : 応答テキストを受け取り、音声WAVデータを生成して返却します。
* `GET  /characters`         : 利用可能なキャラクター一覧をJSON形式で取得します。
* `POST /set_character`      : リクエストで指定したキャラクターをセッションに設定します。
* `GET  /enhanced_defaults`  : Enhanced画面の初期設定値（モデル・声・トランスクリプト設定など）を取得します。
* `POST /clear_history`      : 会話履歴をサーバー・クライアント両方でリセットします。
* `GET  /openai_ephemeral_key`: Realtime API用のエフェメラルキーを取得します。
* `POST /openai_realtime_proxy`: クライアントからのRealtime APIリクエストを安全にプロキシします。
* `GET  /api/character/{name}`: 指定キャラクターの詳細設定情報を取得します。


## キャラクター解説 (日本語版)
- **ja_announcer**: 日本のニュースアナウンサー風のキャラクター。落ち着いた声で丁寧に解説します。
- **ja_butler**: ご主人様と呼びかける執事キャラクター。常に敬語で簡潔に応対します。
- **ja_kamoku**: 非常に寡黙で無愛想。20文字以内の断定口調で回答します。

その他のキャラクターは `characters/` フォルダを参照してください。
