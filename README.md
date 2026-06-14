# GA4 分析レポート自動化

GA4の公式MCPサーバー（[googleanalytics/google-analytics-mcp](https://github.com/googleanalytics/google-analytics-mcp)）とClaude CodeのSKILL.mdワークフローを使って、GA4分析レポートを自動生成し、GitHub Pagesで共有するためのプロジェクト。

## Phase 1: 環境セットアップ（完了）

- [x] Python 3.10+（3.13.3 / Homebrew）
- [x] pipx（1.14.0）
- [x] Google Cloud CLI（572.0.0）
- [x] `pipx run analytics-mcp` で MCP サーバーが起動することを確認済み

## Phase 2: GA4データ連携・レポート生成（完了）

- [x] GCPプロジェクトの作成・選択、Google Analytics Admin API / Data API の有効化
- [x] `gcloud auth application-default login`（サービスアカウントimpersonation）による認証設定
- [x] Claude Code への `analytics-mcp` のMCPサーバー登録（userスコープ）
- [x] レポート生成ワークフローを `.claude/skills/ga4-report/SKILL.md` として実装
- [x] `index.html` をGA4実データのレポートに更新（直近30日間 + 前期間比較）
- [x] 新規GitHubリポジトリの作成・GitHub Pages設定

## 次のステップ

- `index.html` の内容を確認し、問題なければ `git push` してGitHub Pagesに公開
- 必要に応じて `.claude/skills/ga4-report/SKILL.md` を実行してレポートを再生成（直近30日間・前期間比較）
