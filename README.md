# Agent Harness Template

Codex・Claude Code・Cursorで使う開発ハーネスの再利用用テンプレートです。指示だけでなく、実行環境・権限・外部ツール・検証・レビュー・引継ぎを含みます。特定の製品、言語、IDE、モデル、GitHubアカウントを必須にしません。

## まず使う

GitHubの **Use this template** で新規リポジトリを作りcloneします。既存プロジェクトには[移植手順](docs/adoption.md)に従い、既存の指示・hooksを上書きせず統合してください。

```bash
python3 --version                 # Python 3.11以上
python3 scripts/doctor.py         # 認証値や個人設定の内容は読み出さない
python3 scripts/check.py          # テンプレート自身の検証
python3 scripts/bootstrap.py      # 内容確認後、ローカルGit hooksを設定
```

hooks導入後の変更は `agent/1-description` 等のIssue番号付きbranchと専用worktreeで行います。GitHubサーバーの保護は[別途設定](docs/setup/github.md)します。

1. [プロジェクト設定](docs/project.md)に目的、実装言語、実際のbuild/test、完了条件を記入。
2. [外部環境のセットアップ](docs/setup/README.md)から使うクライアントだけ導入。
3. 新規セッションでAGENTS/CLAUDEとstart-workの読込を確認。
4. 小さいIssueを1件進め、テスト・別セッションレビュー・PR・終了確認まで通す。

## 内容

| 層 | 同梱物・入口 |
| --- | --- |
| 常時指示 | [AGENTS.md](AGENTS.md)、CLAUDE互換入口、Cursor rule |
| 作業手順 | start-work / finish-work Skills、[開発フロー](docs/workflow.md) |
| 実行可能な保護 | Git branch/push guard、既存hooksを保全するbootstrap |
| 実行状態 | GUI lease、private handoff registry、[運用手順](docs/operations.md) |
| 検証 | 共通checkコマンド、Python標準unittest、GitHub Actions |
| 外部ハーネス | CLI、認証、MCP、Plugins、通知、GUI、リモート環境、自動実行 |
| 調査・移植判断 | [棚卸し](docs/inventory.md)、[互換性と確認範囲](docs/validation.md) |

## 移植の境界

実行可能な共通部と、導入先で接続する運用手順を分けています。元環境の製品build、成果物配布、製品QA fixture、専用PR coordinator本体は同梱しません。自動進行は[汎用coordinator手順](docs/setup/automation.md)と[依頼文](prompts/coordinator.md)で、選んだクライアントの独立セッションへ接続します。元の専用engineと同等の自動実装・自動mergeが有効になったわけではありません。

認証値、個人履歴、既存claim、ホスト名、絶対パス、trust hash、実行中registry、元環境の自動実行設定は含みません。元環境は変更していません。「設定あり」「実接続済み」「今回の導入試験済み」は別々に記録しています。

共通CLIはmacOS/Linux向けです。Windowsは[環境差分](docs/setup/README.md)を確認してください。検証成功は導入先アプリや全クライアントの動作保証ではありません。
