---
name: operating-kbx-environment
description: 求人ボックス(採用ボード)運用の状態確認、接続の復旧、新しいクライアントの立ち上げ、シートの整理を行う。次の発火で起動すること - 「入った?」「反映された?」「どうなってる」「エラーが出てる」「つながらない」「接続が切れた」「別のクライアントに切り替えて」「新しいクライアントを始めたい」「シートを作って」「終わった求人を整理して」、または処理の進行状況・接続状態に言及したとき。原稿づくりは drafting-kbx-jobs、入稿は revising-and-submitting-kbx-jobs、数字の分析は analyzing-kbx-performance。
---

# 環境を整える

「入ったのか」「つながらないのか」「新しく始めたい」に答える。

## 最初にすること

```
kba_get_skill_guidance({ topic: "ops" })
```

食い違ったら guidance に従う。

## 状態を確認する

| 知りたいこと | ツール |
|---|---|
| 生成・入稿の進行状況 | `kba_get_job_status` |
| 全体を横断で見る | `kba_get_ops_dashboard` |
| 求人ボックス側の接続 | `kba_get_kbox_connection_status` |
| 手動更新の結果 | `kba_get_monitor_run` |

表示が古いときは `kba_trigger_monitor_run` で明示的に取り直し、`kba_get_monitor_run` で完了を待つ。
定期実行とは別枠の手動更新なので、**範囲を絞って使う。** 全件の取り直しを既定にしない。

## つながらないとき

`kba_get_kbox_connection_status` で診断する。

**この skill は認証情報を受け取らない。** チャットで ID やパスワードを聞かない。再接続が必要なら
Web アプリ側でやり直してもらう。

別のクライアントのアカウントで操作したいときは `kba_switch_account`。詳しい手順と落とし穴は
`kba_get_skill_guidance({ topic: "account-switch" })`。**コネクタを削除して追加し直すだけでは
切り替わらないことがある。**

## 新しいクライアントを始める

```
- [ ] 1. kba_get_review_sheet_readiness      不足を洗い出す
- [ ] 2. kba_provision_review_sheet          シートを発行する
- [ ] 3. kba_get_review_sheet                場所と容量を確認する
```

1 で、シートが無いクライアントと、生成済みなのにシートへ載っていないものが分かる。

複数まとめて発行するなら `kba_provision_missing_review_sheets`。**まず dry-run で対象を確認してから**
実行する。

`kba_get_review_sheet` はシートの場所と容量を返す。**行の中身は返らない。** 中身を見るなら
revising-and-submitting-kbx-jobs。

## 整理する

終了・停止した求人がシートに溜まったら `kba_archive_ended_review_sheet_rows`。

**既定は dry-run。** 対象件数を確認してから実行する。記録は残り、表示だけが整理される。

シートが壊れた、作り直しが要る、という場合だけ `kba_regenerate_review_sheet`。
**通常の発行では使わない。** 既存のシートを置き換える操作なので、影響を説明してから実行する。

## 守ること

**認証情報をチャットで受け取らない。** 再接続は Web アプリへ案内する。

**ID を人に転記させない。** 返ってきた ID をそのまま内部で引き回す。

**破壊的な操作は確認してから。** 整理と再発行は、対象と影響を示して同意を得てから実行する。

**設定変更を利用者に依頼しない。** 権限や本番設定で止まった場合は、返ってきた理由をそのまま伝えて
運用者に確認してもらう。

## 対象を選ぶ

クライアントの一覧は `kba_list_clients`、パートナーは `kba_list_partners`。
掲載中の求人を探すなら `kba_search_existing_jobs`。
