# Zenn Article Generation Rules

このプロジェクトはZennの記事を効率的に作成・公開するためのワークフローを実装しています。

## ワークフロー

1. **content_memo.txt** に記事の内容（アイデア、構成、キーポイント）を記載
2. Claudeがそれを参照して、Zenn形式の記事を自動生成
3. 生成された記事をレビュー・編集後、GitHubにプッシュして公開
4. **完了後、content_memo.txtから該当内容を削除し、content_done_list.txtに転記**

## 記事生成時の指示

`content_memo.txt`を参照して記事を生成する際は、以下の手順に従ってください：

### 1. content_memo.txtの読み込み
- 必ず最初に`content_memo.txt`を読み込む
- メモの内容を理解し、記事の方向性を把握する

### 2. Zenn記事の生成
以下のコマンドで新規記事を作成：
```bash
npx zenn new:article --slug [記事のslug] --title "[記事タイトル]" --type tech --emoji "📝"
```

### 3. 記事フォーマット
生成する記事は以下の構成に従う：

```markdown
---
title: "[記事タイトル]"
emoji: "[適切な絵文字]"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["topic1", "topic2", "topic3"] # 最大5つまで
published: false # 最初はfalse
---

# はじめに
[導入部分]

# 本文
[content_memo.txtの内容を元に構成]

# まとめ
[結論]
```

### 4. 記事の品質チェック
- 技術的な正確性を確認
- 読みやすい構成になっているか確認
- コードブロックには適切な言語指定を追加
- 画像やリンクは適切に配置

### 5. プレビューとコミット
```bash
# ローカルプレビュー
npx zenn preview

# 記事をコミット
git add .
git commit -m "Add article: [記事タイトル]"
git push
```

### 6. 完了処理（重要）
記事作成が完了したら、以下の処理を必ず実行：

1. **content_done_list.txtへの転記**
   - 完了日時、記事タイトル、記事slug、元のメモ内容を記録
   - フォーマット：
   ```
   ===== [完了日時] =====
   記事タイトル: [タイトル]
   記事slug: [slug]
   元のメモ:
   [content_memo.txtから転記した内容]
   ====================
   ```

2. **content_memo.txtからの削除**
   - 転記が完了した内容をcontent_memo.txtから削除
   - 他のメモがある場合は、それらは残す

3. **変更をコミット**
   ```bash
   git add content_memo.txt content_done_list.txt
   git commit -m "Move completed content to done list: [記事タイトル]"
   git push
   ```

## 記事のトピックス選定ルール

- 記事内容に最も関連性の高いトピックを選択
- 一般的で検索されやすいトピック名を使用
- 英語の場合は小文字で統一
- 最大5つまで（3〜4つが理想的）

## 注意事項

- `published: false`で作成し、レビュー後に`true`に変更
- 画像を使用する場合は`images/`ディレクトリに配置
- 外部リンクは信頼できるソースのみ使用
- コードサンプルは実行可能で、エラーがないことを確認
- **content_memo.txtの内容は必ずcontent_done_list.txtに転記してから削除**

## よく使うコマンド

```bash
# 記事一覧を確認
ls articles/

# 特定の記事を編集
code articles/[slug].md

# プレビューサーバー起動
npx zenn preview

# 記事を公開状態に変更後
git add .
git commit -m "Publish article: [記事タイトル]"
git push
```

## ファイル管理

- **content_memo.txt**: 記事のアイデアや下書きを記載（作成前）
- **content_done_list.txt**: 完成した記事の履歴（作成後）
- 必ず記事完成後は内容を移動させること