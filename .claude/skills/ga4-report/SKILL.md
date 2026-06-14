---
name: ga4-report
description: "fi-labo.com（GA4プロパティ properties/538409062）のアクセス解析データをanalytics-mcp経由で取得し、直近30日間のHTMLレポートを生成してこのリポジトリのindex.htmlとして保存・GitHub Pagesで公開するワークフロー。「GA4レポート作って」「アクセス解析レポート更新」「fi-laboのレポート生成」「GA4レポートを更新して」などのリクエストに使う。"
---

# GA4 分析レポート生成（fi-labo.com）

`mcp__analytics-mcp__run_report` を使い、fi-labo.com の直近30日間のアクセス解析データを取得して、
このリポジトリの `index.html` をレポートとして更新する。

**対象プロパティ:** `properties/538409062`（fi-labo.com, account 395358275）

---

## ステップ1: 対象期間の決定

- 当期間: `30daysAgo` 〜 `yesterday`（直近30日間）
- 比較期間: `60daysAgo` 〜 `31daysAgo`（その直前の30日間、同じ長さ）

レポート表示用の実際の日付（YYYY年MM月DD日）は、システムの現在日時（`currentDate`）から逆算して算出する。

---

## ステップ2: analytics-mcpでデータ取得

`property_id: "properties/538409062"` で以下を取得する。当期間・比較期間の両方が必要なものは2回呼び出す。
（並列実行可能なものは並列で呼び出してよい）

### 2-1. KPIサマリー（当期間・比較期間それぞれ）
- dimensions: `[]`
- metrics: `["sessions", "activeUsers", "screenPageViews", "conversions", "engagementRate", "averageSessionDuration"]`

### 2-2. 日次トレンド（当期間のみ）
- dimensions: `["date"]`
- metrics: `["sessions", "activeUsers"]`
- order_bys: `[{"dimension": {"dimension_name": "date"}}]`（昇順）

### 2-3. チャネル別（当期間のみ）
- dimensions: `["sessionDefaultChannelGroup"]`
- metrics: `["sessions", "activeUsers", "conversions"]`
- order_bys: `[{"metric": {"metric_name": "sessions"}, "desc": true}]`

### 2-4. 人気ページ TOP10（当期間のみ）
- dimensions: `["pagePath", "pageTitle"]`
- metrics: `["screenPageViews", "sessions"]`
- order_bys: `[{"metric": {"metric_name": "screenPageViews"}, "desc": true}]`, limit: 10

### 2-5. デバイス別（当期間のみ）
- dimensions: `["deviceCategory"]`
- metrics: `["sessions", "activeUsers"]`
- order_bys: `[{"metric": {"metric_name": "sessions"}, "desc": true}]`

### 2-6. 新規 vs リピーター（当期間のみ）
- dimensions: `["newVsReturning"]`
- metrics: `["sessions", "activeUsers"]`

### 2-7. イベント別 TOP10（当期間のみ）
- dimensions: `["eventName"]`
- metrics: `["eventCount"]`
- order_bys: `[{"metric": {"metric_name": "eventCount"}, "desc": true}]`, limit: 10

### 2-8. ページ別ビジネスイベント数（当期間のみ）
申込・フォーム関連のイベントがどのページで発生しているかを把握するための内訳。
- dimensions: `["pagePath", "eventName"]`
- metrics: `["eventCount"]`
- dimension_filter: `{"filter": {"field_name": "eventName", "in_list_filter": {"values": ["online-seminar-apply", "real-seminar-apply", "form_submit", "form_start"]}}}`
  - サイトで使われている実際のビジネスイベント名は2-7の結果から判断し、上記リストを適宜更新する
- order_bys: `[{"metric": {"metric_name": "eventCount"}, "desc": true}]`, limit: 20

---

## ステップ3: 指標の計算

KPIサマリーは当期間と比較期間の差分から増減率を計算する：

```
増減率(%) = (当期間 - 比較期間) / 比較期間 * 100
```

- 比較期間の値が0の場合は「比較対象なし」と表示（ゼロ除算回避）
- 増減率の色分け: 増加 = 緑 (#34A853) / 減少 = 赤 (#EA4335) / 変化なし・比較不可 = グレー (#999)
- `averageSessionDuration`（秒）は「Xm Ys」形式に変換
- `engagementRate`（0〜1）は `xx.x%` 形式に変換

---

## ステップ4: 改善提案の生成

取得した数値（コード分析は行わない）をもとに、以下の観点で該当する項目を評価し、改善提案を作成する。
該当しない項目は無理に含めず、**3〜5件程度**に絞る。各提案は「① 観察された事実（具体的な数値）→ ② 提案」の2点構成で1〜2文にまとめる。複数の観点が同時に強く該当する場合は、最も影響が大きそうなものを優先度「高」とする。

| 観点 | 判定基準 | 提案の方向性 |
|---|---|---|
| ページ別の申込率の差 | 2-8のデータと2-4の人気ページのsessionsから「申込数/sessions」を計算し、ページ間で申込率に大きな差（おおよそ2倍以上）がある場合。特にアクセスの多いページの申込率が低い場合は優先度「高」 | 申込率が高いページの訴求内容・タイトル・CTA配置を、申込率が低い（かつアクセスが多い）ページに応用する提案 |
| キーイベント未設定 | `conversions`が0だが、2-7で申込・フォーム送信系イベント（`*apply*`, `form_submit`等）が1件以上発生 | GA4管理画面でこれらをキーイベントに設定し、コンバージョンを正しく計測するよう提案 |
| フォーム離脱 | 2-8で同じフォーム系ページの`form_start`と`form_submit`を比較し、完了率が低い（おおよそ60%未満）ページがある | そのページのフォーム項目数・入力負荷・エラー表示等を見直す提案。完了率が高い他ページと比較する |
| オーガニック検索の弱さ | チャネル別でオーガニック検索が全体の20%未満 | セミナーページ等のSEO（タイトル・見出し・メタ情報）強化で新規流入を増やす提案 |
| チャネル偏重 | 上位1チャネルがセッションの50%超 | 他チャネルの強化によるリスク分散を提案 |
| モバイル比率 | モバイルが70%以上 | モバイルでの表示・フォーム操作性の確認を提案 |
| リピーター率の低さ | リピーターが全体の20%未満 | メルマガ・リターゲティング等の再訪施策を提案 |
| エンゲージメント率/セッション時間の低さ | エンゲージメント率50%未満、または平均セッション時間60秒未満 | ファーストビュー・コンテンツとニーズのマッチ度の見直しを提案 |
| 日次トレンドの変化 | 直近1週間と前の期間で平均セッション数に20%以上の増減 | 増加要因の深掘り、または減少への対策を提案 |

---

## ステップ5: HTMLレポート生成

`index.html` を1ファイル完結（インラインCSS、外部JS依存なし）で生成する。
既存ファイルのスタイル基調（白背景、見出しに `border-bottom: 2px solid #4285F4`、角丸バッジ）を継承する。

### 全体構成

```
<header>
  タイトル: "GA4 分析レポート - fi-labo.com"
  対象期間: YYYY年MM月DD日 〜 YYYY年MM月DD日（直近30日間）
  比較期間: YYYY年MM月DD日 〜 YYYY年MM月DD日（前期間）
  生成日: YYYY年MM月DD日
</header>

<section> KPIサマリーカード（グリッド表示） </section>
<section> 日次トレンド（SVG折れ線グラフ） </section>
<section> トラフィックチャネル別（CSS横棒グラフ） </section>
<section> 人気ページ TOP10（テーブル） </section>
<section> デバイス別（CSS横棒グラフ） </section>
<section> 新規 vs リピーター（CSS横棒グラフ） </section>
<section> イベント別 TOP10（テーブル） </section>
<section> 改善提案（カード形式） </section>

<footer>
  注記: GA4プロパティ properties/538409062 (fi-labo.com) のデータを基にClaude Codeで自動生成
</footer>
```

### KPIサマリーカード

各カードに: 指標名、当期間の値、前期間比のバッジ（例: ▲+12.3% / ▼-5.1%）。

カード一覧: セッション数, アクティブユーザー数, ページビュー数, コンバージョン数, エンゲージメント率, 平均セッション時間

CSS: `display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 16px;`

### 日次トレンド（SVGグラフ）

viewBox例: `0 0 800 240`。`sessions` と `activeUsers` の2本の折れ線を `<path>` (`stroke`, `fill="none"`) で描画。

- Y軸: 0〜データ最大値でスケーリング
- X軸ラベル: 最初・中間・最後の3点のみ表示
- 配色: sessions = #4285F4, activeUsers = #34A853（凡例を添える）
- グリッド線: #eee の薄い水平線

### CSS横棒グラフ（チャネル別・デバイス別・新規vsリピーター共通）

各行を `<div>` で構成: ラベル + バー（`width: calc(value/max*100%)`、背景色 #4285F4）+ 数値。

### テーブル（人気ページ・イベント別）

シンプルな `<table>`。`pagePath` が長い場合は `text-overflow: ellipsis; white-space: nowrap; overflow: hidden;` で省略し、`title` 属性にフルパスを入れる。

### 改善提案カード

ステップ4で作成した3〜5件の提案を、それぞれ次の形式のカードで表示する。

```html
<div class="suggestion-card priority-high"> <!-- 優先度「高」以外は priority-high を付けない -->
  <h3>見出し（観点を一言で）</h3>
  <p class="fact">現状: 具体的な数値を含む事実</p>
  <p class="action">提案: 推奨アクション</p>
</div>
```

CSS:
```css
.suggestion-card { border: 1px solid #eee; border-left: 4px solid #4285F4; border-radius: 8px; padding: 16px; margin: 12px 0; background: #fafafa; }
.suggestion-card.priority-high { border-left-color: #EA4335; }
.suggestion-card h3 { margin: 0 0 8px 0; font-size: 1em; }
.suggestion-card .fact { color: #666; font-size: 0.9em; margin: 4px 0; }
.suggestion-card .action { font-size: 0.9em; margin: 4px 0; }
.suggestion-card .action::before { content: "💡 "; }
```

### CSSデザイン方針

- `font-family: -apple-system, "Helvetica Neue", Arial, sans-serif;`
- `max-width: 960px; margin: 40px auto; padding: 0 20px; line-height: 1.6; color: #333; background-color: #fff;`
- `<head>` に `<meta name="color-scheme" content="light">` を入れる（ブラウザ/プレビューのダークモードで背景のみ反転し文字が読めなくなるのを防ぐ）
- セクション間は適度な余白または `<hr>` で区切る

---

## ステップ6: 保存・公開

1. `Write` ツールで `index.html` を上書き保存する（リポジトリルート: `/Users/yasuakisato/Local Sites/filab/app/public/ga4-report/index.html`）
2. 保存後、レポートの主要な変化（増減率が大きい指標など）を簡潔にユーザーに報告する
3. **コミット・pushはユーザーの明示的な承認を得てから行う**（GitHub Pagesで公開されるため）。承認後:
   ```
   git add index.html
   git commit -m "Update GA4 report (YYYY-MM-DD)"
   git push
   ```

---

## エッジケース

| 状況 | 対応 |
|---|---|
| `conversions` が全期間で0 | カードに「0」と表示し、増減率は「比較対象なし」 |
| 比較期間のデータが0または存在しない | 増減率を「-」表示、ゼロ除算しない |
| イベントデータが空 | 「イベントデータなし」とセクションに注記 |
| 日次データが30件未満（プロパティ開設直後など） | 取得できた日数分のみグラフ化し、注記を入れる |
| `run_report` がエラーを返す | エラー内容をユーザーに報告し、ADC認証の再確認を促す |
| ステップ4の判定基準に該当する項目が1件もない | 「現状特に大きな問題は見られません」という旨を1件だけ記載する |
