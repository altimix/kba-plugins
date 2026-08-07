---
name: using-kbx-automation
description: 求人ボックス(採用ボード)の求人原稿づくり・レビューシートでの修正・本番入稿・写真登録・月次リライト・成果分析を kba コネクタで行う。次の発火で起動すること - 「求人を作って」「CSVを作って」「原稿を直して」「入稿して」「反映して」「写真を登録して」「新着化して」「リライトして」「数字を見て」「応募率」「掲載保留」、または求人ボックス/採用ボード/KBX/kba に言及したとき。
---

# 求人ボックス運用 (kba)

## 最初にすること

**運用ルールはサーバが持っている。** 作業に入る前に、該当する topic を必ず取得する。

```
kba_get_skill_guidance({ topic: "index" })
```

topic: `csv-spec` / `generation` / `upload` / `rotation` / `analysis` / `account-switch` / `split-differentiation`

このファイルの内容よりサーバの guidance が新しい。食い違ったら guidance に従う。

## 対象と入口

| やりたいこと | 入口 |
|---|---|
| 対象クライアントを決める | `kba_list_clients` |
| 既存の掲載求人を探す | `kba_search_existing_jobs` |
| 原稿を作る | `kba_choose_csv_photo_policy` → `kba_generate_csv_from_config` → `kba_append_generation_job_to_review_sheet` |
| 内容を直す | `kba_preview_review_sheet_edits` → `kba_apply_review_sheet_edits` |
| 本番へ入稿する | `kba_import_review_sheet` → `kba_review_sheet_submit_prepare` → 人の承認 → `kba_review_sheet_submit_confirm` |
| 外部で作った CSV を入れる | `kba_csv_upload_create_url` → PUT → `kba_csv_upload_review` → `kba_csv_upload_submit` |
| 写真を用意する | `kba_create_photo_upload_url` → `kba_archive_photo_upload` → `kba_upload_photo_to_kbox` |
| 成果を見る | `kba_get_kbox_analytics_snapshot` → `kba_get_kbox_ops_recommendations` |
| 状態を確認する | `kba_get_job_status` / `kba_get_ops_dashboard` / `kba_get_kbox_connection_status` |

ツール名は `tools/list` に出ている名前で検索して、スキーマを取得してから呼ぶ。

## 守ること

**利用者の明示した値を最優先する。** 指示された値を内容上の理由で書き換え・空欄化・既定値化・元データへ復元しない。「おまかせ」「仮置き」のような明示的な委任も同じ扱いにする。

**内容の指摘は警告であって停止要因ではない。** 法令・媒体仕様・品質上の懸念は行・列・理由・推奨対応として示すが、それだけで生成・保存・承認・送信を止めない。利用者は警告を確認したうえで、直すか「このまま進む」かを選べる。可否の最終判断は求人ボックス側の応答で確認する。

**停止要因は別物。** `canSubmit=false` が表す停止要因は、構造の破損・security・tenant/client scope・content hash・期限・replay・人の承認といった実行上のゲートだけ。内容の警告がこれになることはない。

**編集の正本は Google レビューシート。** 生成結果のスナップショットを直接書き換えない。内容の修正はレビューシート経由で行う。

**本番送信は人の承認が要る。** prepare で会社名・件数・ステータス・警告・送信先を人に示し、承認を得てから confirm する。承認なしに本番キューへ送らない。

**勤務先名の公開設定だけは明示確認する。** `hidden` / `published` / 既存維持 のどれかを必ず確認する。

**経路を混ぜない。** 同じ CSV を複数の経路へ二重に送らない。各ツールが成功したら、返ってきた nextAction だけを続行し、前段のファイル本文を再送しない。

**ID を人に転記させない。** kba が返した partner / client / job / upload / review / photo の ID をそのまま内部で引き回す。利用者にコピーさせない。

## 接続していないとき

ツールが見つからない場合は、コネクタが未接続。設定 → コネクタ → カスタムコネクタを追加 → `https://mcp.kba.bazaarinc.dev/mcp` を登録してもらう。

別のクライアントのアカウントに切り替えるときは `kba_switch_account`。コネクタを削除して追加し直すだけでは元のアカウントに戻ることがある。
