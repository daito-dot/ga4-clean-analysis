# セットアップガイド

GA4 クリーンデータ生成（`/ga4:clean-data`）を使うための設定手順です。CRO/SEO/GEO スキルは追加設定不要で使えます。

## 1. analytics-mcp のインストール

```bash
# pipx をインストール（未導入の場合）
brew install pipx
pipx ensurepath

# analytics-mcp をインストール
pipx install analytics-mcp
```

インストール先を確認:

```bash
which analytics-mcp
# 例: /Users/あなた/.local/bin/analytics-mcp
```

## 2. Google Cloud の設定

### API の有効化

[Google Cloud Console](https://console.cloud.google.com/) で以下を有効化:

- **Google Analytics Admin API**
- **Google Analytics Data API**

### OAuth クライアント ID の作成

1. 「API とサービス」>「認証情報」>「認証情報を作成」
2. 「OAuth クライアント ID」を選択
3. アプリケーションの種類: **デスクトップ アプリ**
4. 作成後、JSON をダウンロード

### gcloud で認証

```bash
gcloud auth application-default login \
  --scopes="https://www.googleapis.com/auth/analytics.readonly,https://www.googleapis.com/auth/cloud-platform" \
  --client-id-file=ダウンロードしたJSON
```

認証情報の保存先をメモ（通常 `~/.config/gcloud/application_default_credentials.json`）。

## 3. MCP サーバーの接続

プロジェクトルートに `.mcp.json` を作成:

```json
{
  "mcpServers": {
    "analytics-mcp": {
      "command": "/Users/あなた/.local/bin/analytics-mcp",
      "args": [],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/Users/あなた/.config/gcloud/application_default_credentials.json",
        "GOOGLE_PROJECT_ID": "your-project-id"
      }
    }
  }
}
```

各パスを自分の環境に合わせて変更してください。

## 4. 動作確認

Claude Code で以下を実行:

```
/ga4:clean-data
```

GA4 アカウント一覧が表示されれば成功です。

## トラブルシューティング

### MCP サーバーが認識されない

- `.mcp.json` がプロジェクトルートにあるか確認
- VSCode の場合、プロジェクトレベルの `.mcp.json` が必要（`~/.claude/mcp.json` では認識されないことがある）
- `which analytics-mcp` でパスを再確認

### 認証エラー

- `gcloud auth application-default login` を再実行
- `--scopes` に `cloud-platform` が含まれているか確認
- OAuth クライアント ID が「デスクトップ アプリ」タイプか確認（「ウェブ アプリケーション」では動かない）

### GA4 データが取得できない

- Google Cloud Console で Analytics Admin API と Data API が有効か確認
- GA4 プロパティへのアクセス権限があるか確認
