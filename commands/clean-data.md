---
description: "GA4データから開発ノイズを自動除外し、クリーンなデータセットを生成する"
---

GA4 クリーンデータ生成スキルを実行してください。

`skills/ga4-clean-analysis/SKILL.md` のワークフローに従い、以下を順番に実行します:

1. `get_account_summaries` でプロパティ一覧を取得し、ユーザーに対象プロパティを確認
2. 5つのディメンション（hostName, screenResolution, city, sessionSourceMedium, testDataFilterName）で並行レポートを実行し、ノイズを自動検出
3. 自動除外（非本番ホスト、ボット解像度、データセンター都市）とユーザー確認除外を判定
4. `dimension_filter` の `and_group` を構築
5. 除外前後のセッション数を比較し、ユーザーの承認を得る
