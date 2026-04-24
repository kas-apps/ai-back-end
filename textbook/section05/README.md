# セクション5：SQLite入門

## このセクションの目的

セクション4では、「データを永続的に保存するにはデータベースが必要」という理解に至りました。  
このセクションでは、SQLiteを実際に操作して、データの保存・取得・更新・削除を体験します。

コードを書く前に「DBを自分で触れる」という体験を積むことが、このセクションのゴールです。

---

## SQLiteとは何か

### シンプルに理解する

SQLiteの最大の特徴は、**1つのファイルがデータベースそのもの**という点です。

```text
contacts.db ← このファイルがデータベース
```

MySQLやPostgreSQLのような一般的なデータベースは、別途サーバープロセスを起動する必要があります。  
SQLiteはファイルがあるだけで動作します。サーバーは不要で、インストール作業もほぼありません。

### このセクションで行うこと

1. DBファイル（`contacts.db`）を作る
2. テーブルを定義する
3. データを追加・取得・更新・削除する（CRUD）

---

## DBファイルを作る

### sqlite3コマンド

SQLiteは `sqlite3` というコマンドで操作します。  
macOSにはあらかじめインストールされています。Windowsの場合はScoopでインストールできます。

```bash
scoop install sqlite
```

インストールを確認します。

```bash
sqlite3 --version
```

> macOSでは最初から使えることが多いですが、見つからない場合はHomebrewでインストールできます。
>
> ```bash
> brew install sqlite
> ```

### DBファイルを作成して接続する

`project/` フォルダで以下を実行します。

> 必ず `api/` や `public/` が入っている `project/` フォルダで実行します。  
> 別の場所で実行すると、`contacts.db` が別フォルダに作られてしまい、後でPHPから見つけにくくなります。

```bash
sqlite3 contacts.db
```

このコマンドを実行すると、`contacts.db` というファイルが作られ、SQLiteのプロンプトに入ります。

```text
SQLite version 3.x.x
Enter ".help" for usage hints.
sqlite>
```

`sqlite>` というプロンプトが表示されれば成功です。ここからSQLコマンドを入力できます。

> sqlite3を終了するには `.quit` または `Ctrl + D` を入力します。

---

## テーブルとは何か

### Excelの表で考える

データベースのテーブルは、Excelのシートに似ています。

```text
id | name     | email              | phone
---|----------|--------------------|-------------
 1 | 田中 太郎 | taro@example.com   | 090-1234-5678
 2 | 鈴木 花子 | hanako@example.com | 080-9876-5432
```

| 用語 | Excelで言うと | 意味 |
|------|-------------|------|
| テーブル | シート | データをまとめる単位 |
| カラム（列） | 列ヘッダー | データの種類（name, emailなど） |
| レコード（行） | 1行のデータ | 1件の連絡先 |
| 主キー（id） | 行番号 | レコードを一意に識別するID |

### 主キーとは何か

`id` カラムは **主キー（Primary Key）** と呼ばれます。  
テーブルの中で、各レコードを区別するための唯一の値です。

同じ名前の連絡先が2件あっても、`id` が違えば別のレコードとして管理できます。

---

## テーブルを作る

### CREATE TABLE

`sqlite>` プロンプトで以下を実行します。

```sql
CREATE TABLE contacts (
  id    INTEGER PRIMARY KEY AUTOINCREMENT,
  name  TEXT NOT NULL,
  email TEXT,
  phone TEXT
);
```

> すでに作成済みの状態でもう一度実行すると、`table contacts already exists` と表示されます。  
> これは「すでに contacts テーブルがある」という意味なので、作成自体は成功しています。

### 各カラムの意味

| カラム | 型 | 制約 | 意味 |
|--------|---|------|------|
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT | 自動で連番が振られる主キー |
| `name` | TEXT | NOT NULL | 文字列・必須項目 |
| `email` | TEXT | — | 文字列・任意 |
| `phone` | TEXT | — | 文字列・任意 |

> **「必須と任意」の違い**  
> `NOT NULL` をつけると、そのカラムへの値の入力が必須になります。  
> `name` は必須、`email` と `phone` は任意、という設計の意図がここに表れています。

テーブルが作成できたか確認します。

```text
sqlite> .tables
contacts
```

`contacts` と表示されれば成功です。

---

## データを追加する（INSERT）

> ここから先のSQLは、すべて `sqlite>` プロンプトの中で実行します。

### INSERT文

```sql
INSERT INTO contacts (name, email, phone)
VALUES ('田中 太郎', 'taro@example.com', '090-1234-5678');
```

もう1件追加します。

```sql
INSERT INTO contacts (name, email, phone)
VALUES ('鈴木 花子', 'hanako@example.com', '080-9876-5432');
```

### 確認する

追加したデータが保存されているかを確認するには、次のSELECT文を使います。

```sql
SELECT * FROM contacts;
```

以下のような結果が表示されます。

```text
1|田中 太郎|taro@example.com|090-1234-5678
2|鈴木 花子|hanako@example.com|080-9876-5432
```

この出力は | 区切りで少し読みにくいです。SQLiteでは以下を入れると表形式になります。

```sql
-- SELECT文の検索結果にカラム名（ヘッダー）を表示
sqlite> .headers on

-- 表示を見やすく整形
sqlite> .mode column
```

```sql
SELECT * FROM contacts;

id  name       email               phone        
--  ---------  ------------------  -------------
1   田中 太郎  taro@example.com    090-1234-5678
2   鈴木 花子  hanako@example.com  080-9876-5432
```

> **「DBすごい」体験：**  
> このデータはファイルに保存されています。  
> sqlite3を終了して再接続しても、データは残ったままです。

SQLite を終了します。

```text
sqlite> .quit
```

再度、SQLite を起動します。

```bash
sqlite3 contacts.db
```

データを確認します。

```sql
SELECT * FROM contacts;
```

---

## データを取得する（SELECT）

### 全件取得

```sql
SELECT * FROM contacts;
```

`*` は「すべてのカラム」を意味します。

### 特定のカラムだけ取得する

```sql
SELECT name, email FROM contacts;
```

### 条件を指定して取得する

```sql
SELECT * FROM contacts WHERE id = 1;
```

```sql
SELECT * FROM contacts WHERE name = '田中 太郎';
```

### 並び替える

```sql
SELECT * FROM contacts ORDER BY name;
```

LocalStorageでは苦手だった「検索」や「並び替え」が、SQLでは数行で実現できます。

---

## データを更新する（UPDATE）

### UPDATE文

`id = 1` のレコードの名前を変更します。

```sql
UPDATE contacts SET name = '田中 次郎' WHERE id = 1;
```

確認します。

```sql
SELECT * FROM contacts WHERE id = 1;

1|田中 次郎|taro@example.com|090-1234-5678
```

> **`WHERE` を忘れないように：**  
> `WHERE` を書き忘れると、テーブル内のすべてのレコードが更新されます。  
> 更新・削除のSQL文では、`WHERE` による絞り込みを必ず確認する習慣をつけましょう。

---

## データを削除する（DELETE）

### DELETE文

`id = 1` のレコードを削除します。

```sql
DELETE FROM contacts WHERE id = 1;
```

確認します。

```sql
SELECT * FROM contacts;
```

```text
2|鈴木 花子|hanako@example.com|080-9876-5432
```

`id = 1` のレコードが消え、`id = 2` のレコードだけが残っています。

> `DELETE FROM contacts;` のように `WHERE` を書かないと、全件削除になります。

---

## SQLは何をしているのか

### 4つの操作だけ覚える

ここまでに使ったSQLを整理します。

| SQL文 | 操作 | 意味 |
|-------|------|------|
| INSERT | 追加 | テーブルに1件データを追加する |
| SELECT | 取得 | テーブルからデータを読み出す |
| UPDATE | 更新 | 既存のデータを書き換える |
| DELETE | 削除 | データを削除する |

この4つを合わせて **CRUD**（Create / Read / Update / Delete）と呼びます。  
データベースを扱う操作の大部分は、このCRUDに集約されます。

### SQLとLocalStorageの違い

セクション4でLocalStorageと比較したポイントを、SQLで体験できました。

| 操作 | LocalStorage | SQL |
|------|-------------|-----|
| 保存 | `setItem()` | `INSERT INTO ...` |
| 取得 | `getItem()` | `SELECT * FROM ...` |
| 検索 | 自前で実装が必要 | `WHERE name = '...'` |
| 並び替え | 自前で実装が必要 | `ORDER BY name` |
| 削除 | `removeItem()` | `DELETE FROM ... WHERE id = ...` |

SQLは、データを扱うための専用言語です。  
検索・並び替え・集計など、LocalStorageでは難しかった操作をシンプルな文で記述できます。

---

## なぜバックエンドと組み合わせるのか

### SQLiteを単体で操作する限界

ここまではターミナルで直接SQLiteを操作しました。  
しかし、この方法ではフロントエンドからデータを操作することができません。

フロントエンドはブラウザ上で動くJavaScriptです。  
JavaScriptからSQLiteのファイルに直接アクセスする手段はありません。

### バックエンドが橋渡しをする

フロントエンドとSQLiteの間に、バックエンド（PHP）が入ります。

```text
フロントエンド ──[HTTPリクエスト]─▶ バックエンド(PHP) ──[SQL]─▶ SQLite
フロントエンド ◀─[JSONレスポンス]── バックエンド(PHP) ◀─[結果]── SQLite
```

PHPがSQLiteに接続し、フロントエンドからのリクエストに応じてSQLを実行します。  
結果をJSONに変換してフロントエンドに返すのも、PHPの役割です。

---

## セクション5のまとめ

| 項目 | 内容 |
|------|------|
| SQLiteの特徴 | ファイル1つで動くDB、サーバー不要 |
| テーブル | Excelの表のようなデータ構造 |
| CRUD | INSERT / SELECT / UPDATE / DELETE の4操作 |
| SQLの強み | 検索・並び替えをシンプルな文で記述できる |

次のセクションでは、PHPからSQLiteに接続し、CRUDの各操作をAPIとして実装します。
