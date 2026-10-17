# GitHub・CI・保護設定

## 新規repositoryと初回導入

GitHubのTemplate機能はファイルを複製する入口です。GitHub設定、認証、ローカルhooks、Project、実行中状態は別途設定します。[公式Template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository)

1. 公開範囲を決めてrepositoryを作成し、正規の `gh auth login` で必要なアカウントへログイン。
2. cloneし `gh repo view --json nameWithOwner,visibility,defaultBranchRef` で接続先を確認。
3. 小さいIssueを作成してclaimし、専用branch/worktreeからDraft PRを作成。
4. CIの `harness-checks` を実行して成功を確認。内容は候補自身の回帰検証で、検査改変に耐える承認gateではありません（[信頼境界](ci.md#既存の入口と信頼境界)）。
5. required checksとして登録する前に、同名jobの実行実績と導入先アプリの必要な検証jobを確認。
6. 設定後はrulesetとbranchへの実適用を読み戻す。API失敗やプラン制限は未適用と記録。

## 保護方針

統合branchへのPR必須、force push/削除の禁止、必要checks、会話解決を基本にします。レビュー承認数は実際のレビュアー構成に合わせます。同一アカウントの別session reviewはGitHub上の別ユーザーApproveと同じではありません。独立sessionの固定SHAレビューを証拠として記録しても、サーバー強制がない場合は運用上の確認に留まります。

Rulesetの利用範囲はrepositoryの公開範囲と契約に依存します。対応を確認し、非対応時に「保護済み」と書かないでください。[公式Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)

```bash
gh api repos/OWNER/REPOSITORY/rulesets
gh api repos/OWNER/REPOSITORY/rules/branches/main
gh pr checks PR_NUMBER
```

OWNER/REPOSITORY/PR_NUMBERは導入先の値です。元のruleset ID、Project番号、label体系、CODEOWNERS、必須4check名は移植しません。存在しないcheckをrequiredにするとmergeできなくなります。

## CIの信頼境界

[CIの品質と効率](ci.md)を正本に、read-only検証、成果物同一性、自動判定を追加する場合のtrusted code、全量/選択検査と変更時の測定・証拠再利用・独立レビュー・復旧を確認します。元環境の製品配置/buildに結合したclassifierはコピーしません。

## Issue / Project / Milestone

Issueは具体作業、Projectは全体表示、Milestoneは到達目標にします。親子は実際の分解、blocked byは真の依存だけを登録します。単なる分類のための親を作りません。Projectが必要な導入先は登録・Status/Priority・関係を作成/更新/終了時に読み戻します。小規模の新templateでProjectを必須にする必要はありません。

このtemplateには元のPR policy / Agent review / Acceptance gateを実装したCIは含まれていません。独立reviewと受入は[workflow](../workflow.md)で記録し、将来自動化する場合だけ実装・検証してからrequiredに追加します。
