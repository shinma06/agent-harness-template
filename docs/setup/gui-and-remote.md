# GUI・ブラウザー・リモート環境

通常の GUI 操作と GUI 試験では、観察と入力の両方で `agent-desktop` を第一選択にします。特定の純正 Computer Use 等を明示した受入は、その経路で確認します。コード編集やテンプレートの CI に GUI 権限・CLI の追加は不要です。

## セットアップ

1. GUI 作業のあるホストで、[公式の導入手順](https://github.com/vercel-labs/agent-desktop/blob/d3dc190afbf34cd2377252398ddd5c30efbd093a/docs/install.md)と対応 OS を確認します。既存の導入を優先し、`agent-desktop version` と `agent-desktop snapshot --help` で実際の版・引数を確かめます。画面収録・アクセシビリティ等は必要な操作について確認し、OS が新しい同意を求めたときだけ本人に依頼します。
2. 操作前に[共有資源と予約](../operations.md#gui-lease)を確認します。対象限定で独立した背景操作は並行でき、前面入力など競合する区間は予約します。使い捨て fixture で対象のアプリ・プロジェクト・build を識別します。
3. [v1.1.0 の操作手順](https://github.com/vercel-labs/agent-desktop/blob/d3dc190afbf34cd2377252398ddd5c30efbd093a/skills/agent-desktop/SKILL.md)では、`list-windows --app` で実 window を選び、`snapshot --app … --window-id … -i` の最新 ref を使って入力します。再観察で期待結果を確認し、別 window に反映された場合はそこを識別します。画像が必要な受入は実画像も確認し、部分取得や dispatch、CLI 正常終了だけで成功にしません。確認後は fixture を復元します。

## 主経路で対象操作を完了できない場合

未導入、対象を選べない、必要な画像を取得できない、入力が未対応など、対象操作の具体的な制約を記録します。同じ受入を満たす [Peekaboo](https://github.com/openclaw/Peekaboo) または利用中の agent の純正 Computer Use を選びます。両者に固定の優先順はありません。純正経路を明示する受入を別ツールや人間操作で代替しません。既存の観察限定・予算・保全・予約条件を守り、実際の拒否や認証要求を別経路で迂回しません。

切替先の一覧で同じ対象を選び直し、新しい snapshot で要素を特定します。経路間で ref・window ID・座標を流用しません。送信後の結果不明は未送信と分け、状態を照合できるまで再送しません。実経路、観察結果、残条件、次の担当を既存 Issue/QA に残します。画像・診断・cookie・個人 profile は公開しません。ブラウザーでは対象 tab を限定し、純正連携の内部 helper や設定を別環境へ複製しません。

## リモートhost

元の会話環境にはローカルと別OSホストへの接続機能が提供されていましたが、遠隔機の設定・認証・実プロセスは今回調査していません。接続先一覧にあるだけでは操作可能としません。

導入先の公式 remote connection 手順で登録し、接続するユーザー・root・branch・worktree・runtime・対象画面を再照合します。ローカルの権限・PATH・MCP・trust・registry・優先順位を遠隔機へ暗黙継承しません。同じ資源を扱うクライアント間では[共通の予約手順](../operations.md#gui-lease)で競合区間を調整します。host 名や実パスは非公開の運用記録に残します。

## テスト環境

使い捨てfixtureにはテスト用データだけを用意し、本番の個人会話や実tokenを採取しません。実modelへ送信する試験と、fake ACP/MCP serverを使う合成試験を分けます。合成fixtureは通信処理の確認であり、providerの実能力やGUI観察の代替にはなりません。
