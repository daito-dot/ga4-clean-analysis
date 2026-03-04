# GA4 Marketing Toolkit

GA4 クリーンデータ生成 + CRO・SEO・GEO 分析をまとめたマーケティング Cowork プラグイン。

## 含まれるスキル

| コマンド | スキル | 概要 |
|----------|--------|------|
| `/ga4:clean-data` | ga4-clean-analysis | GA4データから開発ノイズを自動除外 |
| `/ga4:cro-audit` | marketing-cro | コンバージョン率最適化の分析・監査 |
| `/ga4:cro-funnel` | cro-funnel | フルファネルCRO（フォーム〜ペイウォール） |
| `/ga4:pricing-optimize` | pricing-page-optimizer | 料金ページのコンバージョン最適化 |
| `/ga4:seo-report` | seo-analytics | PageSpeed InsightsベースのSEOレポート |
| `/ga4:seo-geo` | seo-geo | SEO + AI検索エンジン最適化 |
| `/ga4:geo-audit` | geo-optimizer | AI検索可視性の監査 |

## 前提条件

### Google Analytics MCP サーバー（clean-data 用）

[google-analytics-mcp](https://github.com/googleanalytics/google-analytics-mcp) のセットアップが必要です。

```bash
brew install pipx
pipx ensurepath
pipx install analytics-mcp
```

### Google Cloud の設定

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

### MCP サーバーの接続

プロジェクトの `.mcp.json` に analytics-mcp を設定:

```json
{
  "mcpServers": {
    "analytics-mcp": {
      "command": "/path/to/analytics-mcp",
      "args": [],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/application_default_credentials.json",
        "GOOGLE_PROJECT_ID": "your-project-id"
      }
    }
  }
}
```

`command` のパスは `which analytics-mcp` で確認してください。

## インストール

### Cowork プラグインとして

```
daito-dot/ga4-clean-analysis
```

### Claude Code スキルとして

```bash
npx skills add daito-dot/ga4-clean-analysis -g -y
```

## プラグイン構成

```
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── clean-data.md
│   ├── cro-audit.md
│   ├── cro-funnel.md
│   ├── pricing-optimize.md
│   ├── seo-report.md
│   ├── seo-geo.md
│   └── geo-audit.md
└── skills/
    ├── ga4-clean-analysis/   # オリジナル
    ├── marketing-cro/
    ├── cro-funnel/
    ├── pricing-page-optimizer/
    ├── seo-analytics/
    ├── seo-geo/
    └── geo-optimizer/
```

## クレジット

このプラグインには以下のオープンソーススキルが含まれています:

| スキル | 作者 | ライセンス | ソース |
|--------|------|-----------|--------|
| cro-funnel | AITYTech | MIT | [agentkits-marketing](https://github.com/aitytech/agentkits-marketing) |
| seo-geo | ReScienceLab | MIT | [opc-skills](https://github.com/ReScienceLab/opc-skills) |
| pricing-page-optimizer | ID8Labs | - | - |
| marketing-cro | - | - | - |
| seo-analytics | - | - | - |
| geo-optimizer | - | - | - |

## ライセンス

MIT（ga4-clean-analysis 部分）。バンドルされた各スキルは元の作者のライセンスに従います。
