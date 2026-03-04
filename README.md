# GA4 Marketing Toolkit

GA4 クリーンデータ生成 + CRO・SEO・GEO 分析をまとめた Cowork / Claude Code プラグイン。

## コマンド一覧

| コマンド | 概要 |
|----------|------|
| `/ga4:clean-data` | GA4データから開発ノイズを自動除外 |
| `/ga4:cro-audit` | コンバージョン率最適化の分析・監査 |
| `/ga4:cro-funnel` | フルファネルCRO（フォーム〜ペイウォール） |
| `/ga4:pricing-optimize` | 料金ページのコンバージョン最適化 |
| `/ga4:seo-report` | PageSpeed InsightsベースのSEOレポート |
| `/ga4:seo-geo` | SEO + AI検索エンジン最適化 |
| `/ga4:geo-audit` | AI検索可視性の監査 |

## インストール

### Cowork

設定画面から `daito-dot/ga4-clean-analysis` を追加。

### Claude Code

```bash
npx skills add daito-dot/ga4-clean-analysis -g -y
```

## セットアップ

`/ga4:clean-data` を使うには Google Analytics MCP サーバーの設定が必要です。
詳細は [SETUP.md](SETUP.md) を参照してください。

CRO/SEO/GEO スキルは追加設定なしで使えます。

## ドキュメント

- [SETUP.md](SETUP.md) — GA4 MCP サーバーのセットアップ手順
- [CREDITS.md](CREDITS.md) — バンドルされたスキルの帰属情報
- [CHANGELOG.md](CHANGELOG.md) — 変更履歴

## ライセンス

MIT（ga4-clean-analysis 部分）。バンドルされた各スキルは元の作者のライセンスに従います。
