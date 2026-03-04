# GA4 Clean Analysis

GA4 のアクセスデータから開発環境・ボット・内部アクセスを自動除外し、外部ユーザーのみのクリーンなデータセットを生成する Claude Code スキル。

## なぜ必要か

GA4 のデータにはノイズが大量に混ざっています:

- **開発環境**: localhost、Vercel プレビュー、ステージング
- **ボット**: ヘッドレスブラウザ（800x600 等の異常な解像度）
- **データセンター**: Boardman、Ashburn 等からの自動アクセス
- **社内アクセス**: Direct チャネルに混在する開発者のブラウジング

このスキルは、それらを対話的に識別・除外し、実際の外部ユーザーデータだけで分析できる状態を作ります。

## 前提条件

### 1. Google Analytics MCP サーバー

[google-analytics-mcp](https://github.com/googleanalytics/google-analytics-mcp) のセットアップが必要です。

```bash
# pipx をインストール（未導入の場合）
brew install pipx
pipx ensurepath

# analytics-mcp をインストール
pipx install analytics-mcp
```

### 2. Google Cloud の設定

1. [Google Cloud Console](https://console.cloud.google.com/) でプロジェクトを選択
2. 以下の API を有効化:
   - Google Analytics Admin API
   - Google Analytics Data API
3. OAuth クライアント ID を作成（デスクトップ アプリ）→ JSON をダウンロード
4. gcloud で認証:

```bash
gcloud auth application-default login \
  --scopes="https://www.googleapis.com/auth/analytics.readonly,https://www.googleapis.com/auth/cloud-platform" \
  --client-id-file=ダウンロードしたJSON
```

認証情報の保存先（例: `~/.config/gcloud/application_default_credentials.json`）をメモ。

### 3. MCP サーバーの接続

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

`command` のパスは `which analytics-mcp` で確認してください。

## スキルのインストール

```bash
npx skills add daito-dot/ga4-clean-analysis@ga4-clean-analysis -g -y
```

## 使い方

Claude Code で以下のように呼び出します:

```
/ga4-clean-analysis
```

または自然言語で:

```
GA4を分析して
アクセス解析をクリーンデータで見せて
```

### スキルの流れ

1. **プロパティ特定** - GA4 アカウント一覧からプロパティを選択
2. **ノイズ識別** - 5つのディメンションで除外候補を自動検出
3. **除外判定** - 自動除外 + ユーザー確認で除外対象を確定
4. **フィルタ構築** - GA4 Data API の `dimension_filter` を自動生成
5. **フィルタ確認** - 除外前後のセッション数を比較して承認

## ノイズ判定の基準

### 自動除外

| 種別 | 判定ルール |
|---|---|
| 非本番ホスト | localhost, *.vercel.app, staging 等 |
| ボット解像度 | 800x600, 1024x1024 等の異常値 |
| データセンター | Boardman, Ashburn, Council Bluffs 等 |

### ユーザー確認して除外

| 種別 | 判定ヒント |
|---|---|
| Direct チャネル | 社内アクセスの混在率を確認 |
| 開発系 referral | vercel.com, netlify.com 等 |
| 管理パス | /admin, /d/, /dashboard 等 |
| 社内拠点 | 特定都市への集中パターン |

### 除外しない

| 種別 | 理由 |
|---|---|
| 地方都市 | PR施策の効果測定に必要 |
| SNS 流入 | 認知拡大の効果測定に必要 |
| ブログ記事 | コンテンツマーケの効果測定に必要 |

## ライセンス

MIT
