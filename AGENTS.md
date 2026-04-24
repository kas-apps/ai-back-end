# AGENTS.md — ai-back-end プロジェクト固有指示

## プロジェクト概要

**講座名：** AI 時代のためのバックエンド開発入門｜「動くけど分からない」を卒業する  
**技術スタック：** PHP / SQLite / VSCode  
**対象受講者：** AIでプロダクト開発を始めた初心者（バックエンド未経験）  
**作成物：** 簡易アドレス帳アプリ（単一テーブル CRUD）

---

## このリポジトリの目的

動画講座用の **教材・スライドを作成するリポジトリ**。  
実際のアプリケーションコードではなく、教材コンテンツが主体。

---

## フォルダ構成

```text
ai-back-end/
├── README.md          # コース全体設計（仕様書）
├── AGENTS.md          # このファイル
├── assets/            # 教材・スライド共用の画像・図解
│    └── section01〜11/
├── textbook/          # 教科書用教材（スライド作成のベース）
│    └── section01〜11/
│         └── README.md
└── slides/            # Marp スライド（動画収録用）
     └── section01〜11.md
```

---

## コース構成（全11セクション）

| # | セクション名 | 内容要約 |
|---|------------|---------|
| 1 | はじめに（問題提起） | AI開発の限界→バックエンドを学ぶ動機づけ |
| 2 | バックエンドの基礎概念 | HTTP・リクエスト/レスポンス・JSON・APIの概念 |
| 3 | PHPで最小のバックエンドを作る | 環境構築・PHP内蔵サーバー・JSON API作成 |
| 4 | データ永続化の問題 | LocalStorageの限界・DBが必要な理由 |
| 5 | SQLite入門 | CRUD操作・テーブル・SQLの基本 |
| 6 | DBを操作するAPIを作る | PDO・CRUD API実装 |
| 7 | フロントエンドと連携する | fetch・アドレス帳アプリ完成 |
| 8 | 設計と改善 | 責務分離・API設計・バリデーション |
| 9 | Webセキュリティ入門 | SQLインジェクション・XSS・CSRF対策 |
| 10 | AIとの向き合い方 | プロンプト設計・AIの限界・役割分担 |
| 11 | まとめと次のステップ | SQLiteの限界・MySQL・Laravel・ロードマップ |

---

## DB設計

```sql
CREATE TABLE contacts (
  id    INTEGER PRIMARY KEY AUTOINCREMENT,
  name  TEXT NOT NULL,
  email TEXT,
  phone TEXT
);
```

## API設計（REST風）

```text
GET    /contacts        一覧取得
GET    /contacts/{id}   詳細取得
POST   /contacts        作成
PUT    /contacts/{id}   更新
DELETE /contacts/{id}   削除
```

---

## 教材作成のルール

### textbook（教科書）

- `textbook/section{nn}/README.md` に各セクションの教科書本文を書く
- 読んで理解できる「読み物」として構成する
- スライド作成のベースになるので、構造を明確に（見出し・箇条書きを活用）
- 初心者向けに「なぜ？」を丁寧に説明する
- 口語表現を避け、「〜です・〜ます」調の文章とする

### slides（Marpスライド）

- `slides/section{nn}.md` に Marp 形式で書く
- Marp のフロントマターは以下を基本とする：

  ```yaml
  ---
  marp: true
  theme: default
  paginate: true
  ---
  ```

- 1スライド1メッセージを徹底する
- コードブロックは最小限に絞る
- README.md の「キラーフレーズ」「👉 メッセージ」を積極的に使う

### assets（画像・図解）

- 図解は ASCII アートまたは Mermaid で作成する
- ファイル命名規則：`section{nn}_{説明}.png`

---

## 教材作成時の注意点

- **シンプルを保つ**：複雑な設計・余分な概念は入れない
- **「なぜ」を先に**：「何を」「どうやって」の前に「なぜ必要か」を説明する
- **デモを重視**：概念説明だけでなく、実際に動かす体験を設計に入れる
- **前のセクションと接続する**：学習の流れが途切れないよう、前セクションへの言及を入れる
- **キラーフレーズを活かす**：README.md に記載されたフレーズを教材に落とし込む
