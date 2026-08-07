---
name: preparing-kbx-photos
description: 求人ボックス(採用ボード)で使う求人写真を用意する。手持ち画像の登録、AI での生成、求人ボックスへの反映までを行い、CSV に入れる写真 ID を確定する。次の発火で起動すること - 「写真を登録して」「画像をアップして」「写真を作って」「AI で画像生成」「参考画像を渡す」「登録済みの写真を見せて」「写真を差し替えて」、または写真 ID・画像の一覧に言及したとき。原稿づくりは drafting-kbx-jobs、写真を付けた CSV の入稿は revising-and-submitting-kbx-jobs。
---

# 写真を用意する

求人に付ける写真を、**求人ボックス側で使える状態**にするまでを担う。ここでの成果物は
**求人ボックス側で有効になった写真ID**。CSV への差し込みと入稿は別 skill。

## 原則: 画像そのものを会話に載せない

画像のバイト列を tool の引数や会話に載せない。**必ず署名付き URL へ直接 PUT する。**
tool 引数はモデルの出力トークンなので、サイズに関わらずこの経路を使う。

## 経路は3つ

| やりたいこと | 経路 |
|---|---|
| すでに登録済みの写真を使う | 一覧から選ぶ |
| 手元の画像を登録する | アップロード |
| AI で作る | 生成 |

CSV 作成の会話から来た場合は、`kba_choose_csv_photo_policy` で3択を確定してから該当する経路へ入る。

## 登録済みから選ぶ

- `kba_list_uploaded_photos` — カタログの一覧。絞り込みとページ送りができる
- `kba_select_uploaded_photos` — 最大6枚を順序付きで選ぶ。**人に ID を転記させない**

## 手元の画像を登録する

```
- [ ] 1. kba_create_photo_upload_url    署名付きURLを取る
- [ ] 2. curl などで PUT               会話に画像を載せない
- [ ] 3. kba_archive_photo_upload      KBA のカタログへ登録する
- [ ] 4. kba_upload_photo_to_kbox      求人ボックスへ反映する
```

URL は短命で1回きり。期限が切れたら取り直す。**3 で終わりではない。** カタログに入っただけでは
求人ボックスでは使えない。4 まで通して初めて CSV に入れられる ID になる。

## AI で作る

```
- [ ] 1. kba_list_photo_generation_model_presets   使えるモデルを選ぶ
- [ ] 2. kba_generate_photo                        生成する
- [ ] 3. kba_get_photo_generation                  仕上がりを確認する
- [ ] 4. kba_review_photo_generation               採用 / 作り直し / 中止
- [ ] 5. kba_catalog_photo_generation_output       採用したものを登録する
- [ ] 6. kba_upload_photo_to_kbox                  求人ボックスへ反映する
```

参考画像を渡す場合も参照渡し。`kba_create_photo_reference_upload_url` → PUT →
`kba_register_photo_generation_reference_image`。これは生成の入力専用で、求人ボックスへの
写真登録には使わない。

過去の生成は `kba_list_photo_generations` で辿れる。

**生成しただけでは何も確定していない。** 4 で採用し、5 で登録し、6 で反映するまでは CSV に入れない。
作り直しのたびに前の結果は捨てる。

## 守ること

**求人ボックスへの反映は本番への書き込み。** `kba_upload_photo_to_kbox` の前に、対象クライアント・
ファイル名・警告を人に示して確認する。黙って実行しない。

**CSV に入れてよいのは、求人ボックス側で有効になった写真 ID だけ。** 生成の途中結果や、KBA カタログに
入れただけの ID を使わない。

**1求人に付けられる枚数と、写真 ID の形式には上限と決まりがある。** 現行値は
`kba_get_skill_guidance({ topic: "csv-spec" })` を参照する。

**掲載保留になっている写真は使えない。** 一覧で状態を確認する。

**ID を人に転記させない。** 返ってきた ID をそのまま内部で引き回す。

## CSV へ渡す

新しく原稿を作る流れなら、確定した写真 ID を drafting-kbx-jobs へ渡す。
すでにあるファイルへ写真 ID を付けてから入稿するなら、revising-and-submitting-kbx-jobs の
`kba_prepare_photo_csv_upload` を使う。

## うまくいかないとき

- 署名付き URL の期限切れ → 取り直す
- 同じ URL へ2回 PUT した → 取り直す。**PUT は1回だけ**
- 反映が有効にならない → 対象クライアントと写真の紐づけを確認する
- 本番反映が止まる → 返ってきた理由をそのまま利用者へ伝え、運用者に確認してもらう
