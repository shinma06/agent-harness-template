# 開発と終了の手順

## 開始

1. Issueとコメント、関連PR、実際の依存、既存claim、branch/worktree/dirtyを確認。
2. 目的と受入を具体化し、重複がなければIssueを作成。必要なProject/Milestoneを設定し、関係なしはStandalone。
3. owner、scope、base SHA、target、reviewer、GUI要否、次の操作をclaimして読み戻す。未解放claimは時間だけで失効しない。
4. 設定済み統合branchをfetchし、Issue専用branch/worktreeを作る。初期upstreamが統合branchなら解除。

```bash
git status --short --branch
git worktree list
git fetch --prune origin
# 例: Issue 12、main運用。導入先に合わせる
git worktree add -b agent/12-example ../worktrees/issue-12 origin/main
cd ../worktrees/issue-12
git branch --unset-upstream
python3 scripts/bootstrap.py
```

claim例（実パス・host・tokenは公開しない）:

```text
status: in-progress
owner: implementation-session; issue: #12; parent: none
base: <full SHA>; target: main
branch: agent/12-example; worktree: isolated (private record)
scope: <files and acceptance>; excluded: <out of scope>
dependencies: none; reviewer: separate session pending
gui: not-required (<reason>)
next: <concrete action>
```

## 実装と検証

要件と呼出しを読み、保証したい結果 → 適用できる固定証拠 → 不足している事実 → 最小の十分な検証方法を既存Issueで対応付けます。証拠のsource/artifact・契約・環境・関連依存を確認し、影響のない保証は再利用、変更/不明な保証は必要な再検証を行います。過去のPASSやコマンド成功だけを現在の受入にせず、全内容の確認と代表的な実連携の確認を区別します。

harnessの検証は `python3 scripts/check.py`、導入先アプリはproject.mdの実コマンドです。同じclean committed HEAD・入力・環境・目的への必須検査を有効な既存hookが実行する場合は、その直前に同じ手動全量検査を重ねず、hook設定と実行結果を確認します。dirty中の早期確認、入力/外部条件の変化、別目的、hook未装備には適用しません。必須CI・独立レビューを省く理由にはしません。コード以外でもruntime/buildへの影響を確認し、混在/未知を文書だけとしてskipしません。CI実行・入力・環境を変える場合は[CI変更時の判断](setup/ci.md)も使います。

最初の意味あるpushでDraft PRを作り、目的、Issue、検証、GUI理由、残条件を書きます。公開範囲はユーザーの承認どおりにします。read-only相談ではIssue/claim/PRを新設しません。

## 独立レビューと統合

別セッションへ固定HEAD/baseと受入を渡します。[レビュー依頼文](../prompts/review.md)。providerが同じでも別セッションならよく、自己レビューは代替しません。子agentの利用可否は実環境の規約で判断します。実レビュー経路がなければpacketを完成させて担当へ引き継ぎます。

管理専任のPMを置く場合は、PMが割当・依存・共有資源・証拠と残責任の回収を担当し、実装・技術調査・テスト・独立レビューは各担当へ分離します。必要な入力と独立性が揃った固定候補のレビューを、無関係なPRの統合順だけで待たせません。役割は準備済みの仕事に必要なときだけ、実際に許可された経路で配置します。文書に役割名があるだけでagentを起動しません。

解消していない欠陥、失敗した必須test、受入未達を隠してmergeしません。HEAD/base/target/受入の変更で旧レビューが適用できなくなれば再確認します。現行CIと会話解決を読み戻し、保護付きの通常PR経路で統合します。main/developへ直接pushしません。

実装統合とGUI/releaseを分離する導入先だけ、[二段階運用](setup/automation.md)を適用します。保留受入のQAには初期条件、操作、期待、証拠、owner、再開条件を付けて双方向linkを読み戻します。子PR mergeだけで親IssueやQAを閉じません。

## 中断・終了・cleanup

未完了の停止/引継ぎ前に、次を既存Issueで判断します。本人の停止、実際の権限拒否、安全・実資源の限界は先に該当範囲を止めます。実行全体の時間・費用・合計試行回数の上限/期限に達したら全runを即時停止します。操作別の再試行/修正上限はその操作を止め、別作業の継続は全体の許可・予算・期限が残る場合だけです。方法変更でどちらの上限もリセット・延長しません。

1. 止める範囲と未達を特定し、局所的な試行終了を目的全体の断念へ広げない。
2. 原返却と現在状態で原因を確認し、事実・仮説・不明を分ける。timeoutは実終了の証拠ではなく、接続失敗/未試行を権限拒否にしない。
3. 制約の出所を確認する。本人/toolの限界と必須品質は保持し、自分で選んだ方法や試行枠と分ける。
4. 全体の許可・予算・期限内で、原証拠の再分析、同じ目的を満たす許可済みの別手段、独立して進められる作業から有用な次操作を選ぶ。実拒否を別経路で迂回しない。
5. 継続できれば具体的に進める。計画変更は新しい根拠・有限の上限・復元/結果確認・実権限を次の試行前に定め、悪い結果を理由に失敗や上限をリセットしない。止める必要があれば原因/証拠、保全済み/未達、解決担当・再開条件・既知の手順を残す。人にしか解消できない選択/境界だけを確認し、無回答を許可にしない。

停止/引継ぎはowner、状態、branch/target、HEAD、PR、dirty、検証、残条件、次操作、claimの継続/解放を記録。新ownerは旧writer停止・解放または明示再割当後に別worktreeで開始します。

mergeとcleanupを分けて確認します。remote実ref、local branch、tracking ref、worktreeを照合し、自分の停止済みclean資源だけ整理します。squash後の `--merged` や `[gone]` だけで削除せず、PR HEADと統合結果を確認します。main/master/develop、他owner、不明process、GUI使用中、未保存成果物を保全します。保留は理由/owner/再開条件をIssueへ残します。

## 方法の改善と節目の点検

繰り返す欠陥、待ち、再試行、重複操作、管理負担を観測したら、必要な保証と現在の手段を分けます。十分な方法は維持し、変更する場合は原因 → 保証を保つ代案 → 範囲・上限・保全/復元・成功/中止条件を決めた有限試行 → 実結果による採用/修正/撤回を既存Issueに残します。結果不明の書込みは実状態を読んでから再試行し、同じ条件で成功まで繰り返しません。試行未実施はownerと次操作を持つ未完了です。必要な契約改訂/独立レビューを省かず、[現行指示の整合](context.md)まで含めます。

主要な目的・品質・役割・環境の変更、構造的な問題、導入先で定めた実merge件数、計画済みの中間/終了点を契機に必要な範囲を点検します。役割を置くチームではPMが既存のIssue/PR・原証拠を渡し、点検担当が目的・方法・品質・全役割の負担を有限に確認します。少数標本は問題例だけでなく通常成功や未完了も含め、選定理由と時間/件数の上限を実課題に合わせます。日常作業を助言待ちにせず、毎PRの全体監査、別台帳、日次報告、監視jobを追加しません。

同じイベントの再通知は既存結果へ結び、不確かな配送は元結果/相手の状態を確認します。不在なら未処理事項と保管ownerを残し、停止中の役割を自動再開しません。点検は結果と残責任を返して終了します。成果の受領、実行停止、保存/後の参照、claim解放は別に確認します。元失敗・未実施・本人の助言が契機だった事実を保持し、局所試行や通信成功を全体の時間/品質/費用改善へ拡張しません。
