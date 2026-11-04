# GUI lease・private引継ぎ・復旧

## GUI lease

[GUI の観察・入力](setup/gui-and-remote.md)では、操作ごとに入力先、結果が現れる先、他担当の作業・証拠への影響を確認します。別ウィンドウや別 worktree、headless モードだけでは独立とは判断できません。例えば背景のボタン操作でも共有ダイアログが開くことがあります。

- 対象の instance・window・最新の観察を照合し、入力先と結果先をそれぞれ独立に所有し、他の観察・入力・保存先に干渉しないと確認できた背景操作は、全体予約なしで並行できます。
- フォーカス、物理入力、clipboard、共通 profile・device・daemon、録画・fixture・出力先などを共有する区間は直列化します。具体的な共通影響を切り分けられない区間も同様です。観察だけでも他担当の証拠に干渉する場合は予約します。
- 既存の全体排他や範囲不明の予約があれば、owner と対象・副作用の範囲を合意するまで操作を重ねません。競合しない準備や解析は続けられます。GUI 経路の切替で影響が変わる場合は次の操作前に判定し直します。

`scripts/gui_lease.py` はホスト/OSユーザーごとの協調予約です。OS の入力遮断、対象別の予約、担当の優先順位は実装していません。GitHub 上の担当割当と人間の操作を確認し、必要な区間の空き予約を既存の許可内で取得します。

```bash
python3 scripts/gui_lease.py status
python3 scripts/gui_lease.py acquire --owner gui-session --issue 12 --run run-12-a --head FULL_40_CHARACTER_SHA --minutes 45
python3 scripts/gui_lease.py renew --token PRIVATE_LEASE_TOKEN --minutes 15
python3 scripts/gui_lease.py release --token PRIVATE_LEASE_TOKEN
```

SHAとtokenは実値へ置き換えます。返却にはtoken・host等が含まれるため、チャット/GitHubにそのまま貼らず、private記録に保存します。1回の期限は1〜45分。renewは試験全体の予算を延長する許可ではありません。

既定保存先は `/tmp/agent-harness-gui-<uid>` で全project/clone/worktreeが共有します。`--state-dir`は隔離テスト専用です。projectごとの保存先で同じ画面を並行操作しません。既存ハーネスが別のleaseを使うホストでは、この新leaseだけで操作を始めず、全操作者と旧lease解放・移行先を調整してください。

予約が必要な区間では最初の操作とinstall/起動/再起動/送信/復元の直前にtoken/期限を確認します。期限切れでも別ownerのacquireは拒否します。元ownerまたは調整担当が旧処理停止・画面状態・未保存データを確認し、現在tokenでreleaseしてから次担当へ渡します。PIDや時間だけで自動削除しません。

画面で対象アプリ・fixture・buildを識別し、source SHAと実ロードartifactのhashを照合します。古い画面要素・座標を使い回さず最新状態から操作します。終了時は自分の処理だけ停止/引継ぎ、Case別結果と次操作を残し、取得した予約をreleaseします。

### ホスト限定の設定例

ある共有ホストで、利用者が「準備済みのプロジェクト A を次に優先する」と指定した場合だけ、実際の競合時にその順序を適用します。先行 owner が安全な区切りで保存・通常復帰・返却した後、A が準備済みなら次に取得します。横取り、強制終了、未準備の仕事の先取り予約はしません。実プロジェクト名・対象ホスト・有効範囲は非公開の引継ぎで照合します。テンプレートを複製しても、別ユーザー・ホストにこの優先順位は継承されません。

## private handoff registry

`scripts/handoff_registry.py`は公開可能なrecordと、実パス/hostのprivate対応表を結ぶ小さなPython APIです。coordinatorそのものではありません。

- `register(storage, public, source, host)` はopaque IDとversionをpublicに追加し、digestとsource/hostをprivate側へ保存。
- `resolve(storage, public, host)` はID、所有者、0600、host、digestを照合し、不一致なら拒否。
- storageはgit common-dir配下など、同じrepositoryの本人専用の保管先。storage/registry directoryは本人所有の0700を要求し、symlinkや権限不整合を変更せず拒否する。公開ファイルへ置かない。
- publicにはowner/Issue/PR/HEAD/base/target/scope/停止宣言などを含め、パスや秘密情報は入れない。
- `source`は実在するowned worktreeの絶対パスを呼出側で確認する。registryだけではwriter停止・Git clean・remote SHAは保証しない。

public recordの変更は古い登録のdigestと一致しなくなります。registry喪失・別host・改変・権限不整合時に勝手に再生成やowner差替えをせず、元ownerと復旧します。バックアップはprivateに保ち、実行中の登録を別projectへ移しません。

## Git hooks

bootstrapは当該repositoryのcore.hooksPathだけを設定し、既存custom hooksを検出すると停止します。共通git-dirを持つworktreeにも影響するので導入を共有します。guardはmain/master/developへの直接push/削除、Issue番号のないbranch、別branch/別HEAD/dirtyのpush、非fast-forwardを拒否します。branch名が正しくてもIssueの実在やclaimを確認するものではありません。

pre-pushは一時テストrepositoryへのGit環境変数の混入を防いでからcheck.pyを呼びます。ローカルhooksは事故防止で、サーバー保護の代用ではありません。ローカル設定を無効にして通す運用を作りません。
