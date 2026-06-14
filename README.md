# GA4 分析レポート自動化

GA4の公式MCPサーバー（[googleanalytics/google-analytics-mcp](https://github.com/googleanalytics/google-analytics-mcp)）とClaude CodeのSKILL.mdワークフローを使って、GA4分析レポートを自動生成し、GitHub Pagesで共有するためのプロジェクト。

## Phase 1: 環境セットアップ（完了）

- [x] Python 3.10+（3.13.3 / Homebrew）
- [x] pipx（1.14.0）
- [x] Google Cloud CLI（572.0.0）
- [x] `pipx run analytics-mcp` で MCP サーバーが起動することを確認済み

## 次のステップ（未実施）

- GCPプロジェクトの作成・選択、Google Analytics Admin API / Data API の有効化
- `gcloud auth application-default login` による認証設定（OAuthスコープ: `analytics.readonly`, `cloud-platform`）
- Claude Code への `analytics-mcp` のMCPサーバー登録（プロジェクトスコープの `.mcp.json`）
- レポート生成ワークフローを `SKILL.md` として実装
- 新規GitHubリポジトリの作成・GitHub Pages設定
