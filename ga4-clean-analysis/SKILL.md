---
name: ga4-clean-analysis
description: "GA4 MCP接続を使い、開発環境・ボット・内部アクセスを自動除外したクリーンなデータセットを生成する。「GA4を分析して」「アクセス解析して」「クリーンデータで見て」で起動。"
---

# GA4 クリーンデータ生成

GA4 MCP サーバー（`analytics-mcp`）のデータから開発ノイズを自動除外し、外部ユーザーのみのクリーンなフィルタを構築する。

## 前提

- `analytics-mcp` MCP サーバーが接続済み

## ワークフロー

### Step 1: プロパティ特定

`get_account_summaries` で一覧取得。ユーザーにプロパティIDを確認。

### Step 2: ノイズ識別

以下5つのレポートを**並行実行**し、除外候補を自動検出する:

| # | dimension | 検出対象 |
|---|---|---|
| 1 | `hostName` | 開発環境（localhost, *.vercel.app, staging等） |
| 2 | `screenResolution` | ボット（800x600, 1024x1024 等の異常解像度） |
| 3 | `city` | データセンター（Boardman, Ashburn等） |
| 4 | `sessionSourceMedium` | 開発系 referral（vercel.com等） |
| 5 | `testDataFilterName` | 既存の内部トラフィックフィルタ状態 |

全て `metrics: ["sessions", "totalUsers"]` で、`order_bys` はセッション降順。

### Step 3: 除外判定

#### 自動除外（確認不要）

| 種別 | 判定ルール |
|---|---|
| **非本番ホスト** | hostname が本番ドメイン以外（localhost, preview, staging, *.vercel.app） |
| **ボット解像度** | 800x600, 1024x1024, 1600x1200 など正方形・低解像度で全員1セッション |
| **データセンター都市** | Boardman, Ashburn, Council Bluffs, Dublin(IE), Singapore(SG) で1PV/セッション |

#### ユーザー確認して除外

| 種別 | 判定ヒント | 確認の聞き方 |
|---|---|---|
| **Direct** | 社内ブックマーク、URL直打ちが混在しやすい | 「Directチャネルには社内アクセスが混ざりやすいですが、除外しますか？」 |
| **開発系 referral** | vercel.com, netlify.com 等、少数ユーザーで多セッション | 「vercel.com からの参照は開発ダッシュボード経由ですか？」 |
| **管理パス** | /admin, /d/, /dashboard 等 | 「/d/ のようなパスは管理画面ですか？」 |
| **社内拠点** | 特定都市に少数ユーザー・多セッションが集中 | 「会社の拠点はどちらですか？」 |

#### 除外しない

| 種別 | 理由 |
|---|---|
| 地方都市 | PR施策の効果測定に必要 |
| SNS流入 | 認知拡大の効果測定に必要 |
| ブログ記事 | コンテンツマーケの効果測定に必要 |

### Step 4: フィルタ構築

Step 2-3 の結果をもとに `dimension_filter` の `and_group` を構築。
テンプレート:

```json
{
  "and_group": {
    "expressions": [
      {"filter": {"field_name": "hostName", "string_filter": {"match_type": 1, "value": "本番ドメイン", "case_sensitive": false}}},
      {"not_expression": {"filter": {"field_name": "screenResolution", "in_list_filter": {"values": ["除外解像度リスト"]}}}},
      {"not_expression": {"filter": {"field_name": "city", "in_list_filter": {"values": ["除外都市リスト"]}}}},
      {"not_expression": {"filter": {"field_name": "sessionDefaultChannelGroup", "string_filter": {"match_type": 1, "value": "Direct", "case_sensitive": false}}}},
      {"not_expression": {"filter": {"field_name": "sessionSource", "string_filter": {"match_type": 2, "value": "vercel", "case_sensitive": false}}}}
    ]
  }
}
```

### Step 5: フィルタ確認

構築したフィルタで1つレポートを実行し、除外前後のセッション数を比較。
除外率をユーザーに提示:

```
除外前: ○○○ セッション
除外後: ○○○ セッション（-○○%）
除外内訳: 開発環境 ○○○、ボット ○○、Direct ○○○ ...
```

ユーザーの承認を得てから、このフィルタを以降の全レポートに適用する。

## 注意

- フィルタ内容はプロパティごとに異なる。テンプレートをそのまま使わず、必ず Step 2 の結果でカスタマイズする
- 「除外しすぎ」より「少し残る」方が安全。迷ったらユーザーに聞く
- 内部トラフィックフィルタがテストモードの場合、GA4管理画面での有効化を推奨する
