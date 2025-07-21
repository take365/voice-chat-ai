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

## 画面の使い方
- **ダッシュボード**: このモードでは利用できません。フル版のUIをご覧ください。
- **エンハンス**: OpenAI TTSモデルを使った対話画面。APIキーは利用者が入力する形式です。
- **リアルタイム**: OpenAI Realtime APIを利用したWebRTC対話画面。APIキーを入力して使用します。
- **スピーチテスト**: ブラウザの Web Speech API を試すための簡易ページです。

## ファイル構成
- `app/minimal_app.py` : APIとHTMLページを提供するFastAPIアプリ本体
- `app/minimal_shared.py` : STT/LLM/TTS処理をまとめた共通ロジック
- `app/templates/` : 最小限のHTMLテンプレート類

## API一覧
`minimal_app.py` では以下のエンドポイントが定義されています。
- `POST /api/transcribe`
- `POST /api/chat`
- `POST /api/synthesize`
- `GET  /characters`
- `POST /set_character`
- `GET  /kokoro_voices`
- `GET  /enhanced_defaults`
- `POST /clear_history`
- `GET  /openai_ephemeral_key`
- `POST /openai_realtime_proxy`
- `GET  /api/character/{name}`

## キャラクター解説 (日本語版)
- **ja_announcer**: 日本のニュースアナウンサー風のキャラクター。落ち着いた声で丁寧に解説します。
- **ja_butler**: ご主人様と呼びかける執事キャラクター。常に敬語で簡潔に応対します。
- **ja_kamoku**: 非常に寡黙で無愛想。20文字以内の断定口調で回答します。

その他のキャラクターは `characters/` フォルダを参照してください。
