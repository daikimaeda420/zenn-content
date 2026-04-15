# zenn-content

このリポジトリは、Zenn 記事を GitHub 管理し、Zenn CLI でローカル確認したうえで公開するための公開用リポジトリです。Obsidian の Vault とは分離し、公開対象だけをここで管理します。

## このリポジトリの役割

- Zenn に公開する記事・本の管理
- Zenn CLI によるローカルプレビュー
- GitHub 経由での変更履歴管理と Zenn 反映の基盤

## Obsidian 側との役割分担

- `05_Public/zenn/` は下書き置き場
- この `zenn-content/` は公開用置き場
- 下書きをそのまま全部入れず、公開用に整えたものだけ `articles/` に置く
- 1記事1Markdownファイルで管理する
- まずは最小構成で始める

## ディレクトリ構成

```text
zenn-content/
├── articles/   # 公開記事
├── books/      # 公開する本
├── package.json
└── README.md
```

記事ファイルは `articles/` に置きます。公開する本を使う場合は `books/` に置きます。

## 基本的な運用フロー

1. Obsidian の `05_Public/zenn/` で下書きを作る
2. 公開対象だけを `articles/` に 1 記事 1 ファイルで移す
3. Zenn 用の frontmatter や見出し、リンク、画像参照を公開向けに整える
4. `npm run preview` でローカル確認する
5. GitHub に commit / push する
6. Zenn と GitHub を連携済みなら、push 内容が Zenn 側に反映される

## Zenn CLI の基本コマンド

```bash
npm run preview
npm run new:article
npm run new:book
npx zenn --help
```

- `npm run preview`: ローカルプレビューを起動する
- `npm run new:article`: 新規記事ファイルを作成する
- `npm run new:book`: 新規本を作成する

## GitHub へ push して Zenn へ反映する流れ

1. `articles/` か `books/` に公開用コンテンツを配置する
2. ローカルでプレビュー確認する
3. Git で差分確認し、commit する
4. GitHub に push する
5. Zenn 側で GitHub 連携済みなら内容が反映される

## 公開前の確認ポイント

- 公開対象だけが `articles/` に入っているか
- frontmatter の `title`、`emoji`、`type`、`topics`、`published` が適切か
- 誤字、リンク切れ、画像参照ミスがないか
- Obsidian 向けのメモ記法や非公開メモが残っていないか
- `npm run preview` で表示崩れがないか

## 補足ルール

- 公開前の整形や見直しはこのリポジトリ側で行う
- 公開しない下書きは Obsidian 側に残し、このリポジトリには持ち込まない
- 運用が固まるまでは構成を増やしすぎず、`articles/` 中心で管理する
