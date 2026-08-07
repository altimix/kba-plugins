# kba-tools

求人ボックス(採用ボード)運用を Claude から行うための **kba プラグイン配布マーケットプレイス**です。

## 使い方

### claude.ai / Claude デスクトップアプリ

設定 → カスタマイズ → プラグイン → **追加** → マーケットプレイスを追加 → リポジトリから追加

```
altimix/kba-plugins
```

同期したら、一覧から **kba** をインストールします。

### Claude Code

```bash
claude plugin marketplace add altimix/kba-plugins
```

```bash
claude plugin install kba@kba-tools
```

## 接続

プラグインには kba Remote MCP server の接続設定が含まれています。認証は接続時の OAuth で行います。手動で追加する場合のエンドポイントは次のとおりです。

```
https://mcp.kba.bazaarinc.dev/mcp
```

利用できるクライアントと操作範囲はアカウントごとに決まります。

## このリポジトリについて

配布専用のミラーです。**直接編集しないでください。** 内容は非公開の開発リポジトリを正として、リリース時に自動で同期されます。

秘密情報は含みません。認証情報・トークン・Cookie はプラグインに含めず、すべて接続時の OAuth と環境変数で扱います。

## 更新

更新のお知らせがあったら、プラグイン画面でマーケットプレイス `kba-plugins` を **同期** してください。最新版に入れ替わります。

運用ルール・媒体仕様・チェック内容など変わりやすいものは、プラグインではなくサーバ側から配信されます。そちらは同期なしで常に最新です。プラグイン自体の更新はまれです。

## 問い合わせ

Bazaar Japan — https://app.kba.bazaarinc.dev
