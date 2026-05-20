---
marp: true
theme: udemy
footer: "<div>データベースを操作する API を作る</div>"
paginate: false
---

<!-- _class: title -->
<!-- _footer: "" -->
<!-- _paginate: skip -->

# AI 時代のためのバックエンド開発入門

## セクション 6: データベースを操作するAPIを作る

---

<!-- _class: heading -->
<!-- _footer: "" -->

# PHPとSQLiteを接続する

---

## PDOとは何か

PHPからSQLiteに接続するには **PDO（PHP Data Objects）** を使います。
PDOはPHPに標準で組み込まれたデータベース接続の仕組みです。

| メリット | 説明 |
|---------|------|
| 統一されたインターフェース | SQLite・MySQL・PostgreSQLなど、DBが変わっても書き方が同じ |
| プリペアドステートメント | SQLインジェクション対策として重要（セクション9で詳しく扱います） |
| PHPに標準搭載 | 追加インストール不要 |

---

## PDOでSQLiteに接続する

> ```text
>  project/
>   ├── api/
>   │   └── index.php
>   └── contacts.db
> ```

```php
$pdo = new PDO('sqlite:../contacts.db');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

- `sqlite:` の後にDBファイルのパスを指定する
- `../contacts.db` は `api/index.php` から見た `project/contacts.db` への相対パス
- `PDO::ERRMODE_EXCEPTION` はエラー発生時に例外を投げる設定

---

## pdo_sqlite / sqlite3 の有効化（Windows）

Windowsでは `php.ini` を編集して、PDOでSQLiteに接続できるように設定します。

`C:\Users\[ユーザー名]\scoop\apps\php\current\cli\php.ini` を開いて、以下の行を探します。

```ini
;extension=pdo_sqlite
;extension=sqlite3
```

先頭のセミコロン（`;`）を削除して保存します。

```ini
extension=pdo_sqlite
extension=sqlite3
```

保存後、PHP内蔵サーバーを再起動してください。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# APIエンドポイントの構成

---

## 実装するエンドポイント

`api/index.php` 1ファイルに、5つのエンドポイントをまとめます。

```text
GET    /api/contacts        → 一覧取得
GET    /api/contacts/{id}   → 1件取得
POST   /api/contacts        → 登録
PUT    /api/contacts/{id}   → 更新
DELETE /api/contacts/{id}   → 削除
```

今回の `api/index.php` は、大きく3つの処理に分かれます。

1. DBに接続する
2. リクエストのURLとHTTPメソッドを調べる
3. 条件に合うSQLを実行してJSONを返す

---

## 共通部分：DB接続とルーティングの骨格（コード）

`api/index.php` を以下のコードに書き換えます。

```php
<?php
header("Content-Type: application/json");

$pdo = new PDO('sqlite:../contacts.db');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$method = $_SERVER["REQUEST_METHOD"];
$uri    = parse_url($_SERVER["REQUEST_URI"], PHP_URL_PATH);

if (preg_match('#^/api/contacts(?:/(\d+))?$#', $uri, $matches)) {
  $id = $matches[1] ?? null;
} else {
  http_response_code(404);
  echo json_encode(["error" => "Not Found"]);
  exit;
}
```

---

## 共通部分：DB接続とルーティングの骨格（解説）

| 部分 | 意味 |
|------|------|
| `$_SERVER["REQUEST_METHOD"]` | GET / POST / PUT / DELETE のいずれかが入る |
| `parse_url(..., PHP_URL_PATH)` | URLからパス部分だけを取り出す |
| `preg_match(...)` | URLのパターンを確認し、`{id}` 部分を取り出す |
| `$id = $matches[1] ?? null` | URLに `/1` のような数字があればその値、なければ `null` |

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 取得API（GET）

---

## 一覧取得（GET /api/contacts）

`api/index.php` に以下のコードを追記します。

```php
if ($method === "GET" && $id === null) {
  $stmt = $pdo->query("SELECT * FROM contacts");
  $contacts = $stmt->fetchAll(PDO::FETCH_ASSOC);
  echo json_encode($contacts, JSON_UNESCAPED_UNICODE);
  exit;
}
```

| 部分 | 意味 |
|------|------|
| `$pdo->query(...)` | SQLを実行し、結果オブジェクトを返す |
| `fetchAll(PDO::FETCH_ASSOC)` | 全行をカラム名をキーとした連想配列で取得する |

---

## 動作確認（一覧取得）

ブラウザで `http://localhost:8000/api/contacts` にアクセスするか、ターミナルで以下のコマンドを実行します。

```bash
# macOS / Linux / PowerShell 7.x
curl http://localhost:8000/api/contacts

# Windows PowerShell 5.1
curl.exe http://localhost:8000/api/contacts
```

#### 実行結果

```JSON
[{"id":2,"name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"}]
```

---

## 1件取得（GET /api/contacts/{id}）

`api/index.php` に以下のコードを追記します。

```php
if ($method === "GET" && $id !== null) {
  $stmt = $pdo->prepare("SELECT * FROM contacts WHERE id = ?");
  $stmt->execute([$id]);
  $contact = $stmt->fetch(PDO::FETCH_ASSOC);

  if ($contact === false) {
    http_response_code(404);
    echo json_encode(["error" => "Not Found"]);
  } else {
    echo json_encode($contact, JSON_UNESCAPED_UNICODE);
  }
  exit;
}
```

---

## 1件取得（解説）

| 部分 | 意味 |
|------|------|
| `prepare("... WHERE id = ?")` | `?` をプレースホルダーとした準備済みSQL |
| `execute([$id])` | `?` の部分に `$id` を渡して実行する |
| `fetch(PDO::FETCH_ASSOC)` | 1行だけを取得する |

---

## 動作確認（1件取得）

<!-- #### 存在するIDの場合 -->

```bash
# macOS / Linux / PowerShell 7.x
curl http://localhost:8000/api/contacts/2
# Windows PowerShell 5.1
curl.exe http://localhost:8000/api/contacts/2
# 実行結果
{"id":2,"name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"}
```

#### 存在しないIDの場合

```bash
# macOS / Linux / PowerShell 7.x
curl http://localhost:8000/api/contacts/999
# Windows PowerShell 5.1
curl.exe http://localhost:8000/api/contacts/999
# 実行結果
{"error":"Not Found"}
```

---

## プリペアドステートメントとは

`?` を使った書き方を **プリペアドステートメント** と呼びます。

```php
// NG：URLから来た値をそのままSQLに埋め込む
$pdo->query("SELECT * FROM contacts WHERE id = $id");

// OK：プレースホルダーを使い、PDOが安全に処理する
$stmt = $pdo->prepare("SELECT * FROM contacts WHERE id = ?");
$stmt->execute([$id]);
```

URLや入力から来た値を直接SQLに埋め込まず、PDOが安全に処理します。

<div class="important">

> **SQLインジェクション**という攻撃への対策として重要です。セクション9で詳しく扱います。

</div>

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 登録API（POST）

---

## 登録（POST /api/contacts）

`api/index.php` に以下のコードを追記します。

```php
if ($method === "POST" && $id === null) {
  $input = json_decode(file_get_contents("php://input"), true);

  $stmt = $pdo->prepare(
    "INSERT INTO contacts (name, email, phone) VALUES (?, ?, ?)"
  );
  $stmt->execute([
    $input["name"],
    $input["email"] ?? null,
    $input["phone"] ?? null,
  ]);

  http_response_code(201);
  echo json_encode(["id" => $pdo->lastInsertId(), "message" => "Created"]);
  exit;
}
```

---

## 登録（解説）

| 部分 | 意味 |
|------|------|
| `file_get_contents("php://input")` | リクエストボディ（JSON文字列）を取得する |
| `json_decode(..., true)` | JSON文字列をPHPの連想配列に変換する |
| `$input["email"] ?? null` | キーがない場合は `null` を使う |
| `lastInsertId()` | 直前のINSERTで発行されたIDを取得する |
| `http_response_code(201)` | 「作成成功」を意味するHTTPステータスコード |

---

## 登録の確認

```bash
# macOS / Linux
curl -X POST http://localhost:8000/api/contacts \
-H "Content-Type: application/json" \
-d '{"name":"田中 太郎","email":"taro@example.com","phone":"090-1111-2222"}'

# PowerShell 7.x
curl -X POST http://localhost:8000/api/contacts `
-H "Content-Type: application/json" `
-d '{"name":"田中 太郎","email":"taro@example.com","phone":"090-1111-2222"}'

# Windows PowerShell 5.1
curl.exe -X POST http://localhost:8000/api/contacts `
-H "Content-Type: application/json" `
-d '{"""name""":"""田中 太郎""","""email""":"""taro@example.com""","""phone""":"""090-1111-2222"""}'

# 実行結果
{"id":3,"message":"Created"}
```

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 更新API（PUT）

---

## 更新（PUT /api/contacts/{id}）

`api/index.php` に以下のコードを追記します。

```php
if ($method === "PUT" && $id !== null) {
  $input = json_decode(file_get_contents("php://input"), true);

  $stmt = $pdo->prepare(
    "UPDATE contacts SET name = ?, email = ?, phone = ? WHERE id = ?"
  );
  $stmt->execute([
    $input["name"],
    $input["email"] ?? null,
    $input["phone"] ?? null,
    $id,
  ]);

  echo json_encode(["message" => "Updated"]);
  exit;
}
```

---

## 動作確認（更新）

```bash
# macOS / Linux
curl -X PUT http://localhost:8000/api/contacts/3 \
-H "Content-Type: application/json" \
-d '{"name":"田中 次郎","email":"jiro@example.com","phone":"090-3333-4444"}'

# PowerShell 7.x
curl -X PUT http://localhost:8000/api/contacts/3 `
-H "Content-Type: application/json" `
-d '{"name":"田中 次郎","email":"jiro@example.com","phone":"090-3333-4444"}'

# Windows PowerShell 5.1
curl.exe -X PUT http://localhost:8000/api/contacts/3 `
-H "Content-Type: application/json" `
-d '{"""name""":"""田中 次郎""","""email""":"""jiro@example.com""","""phone""":"""090-3333-4444"""}'

# 実行結果
{"message":"Updated"}
```

---

## 更新内容の確認

```bash
# macOS / Linux / PowerShell 7.x
curl http://localhost:8000/api/contacts/3

# Windows PowerShell 5.1
curl.exe http://localhost:8000/api/contacts/3

# 実行結果
{"id":3,"name":"田中 次郎","email":"jiro@example.com","phone":"090-3333-4444"}
```

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 削除API（DELETE）

---

## 削除（DELETE /api/contacts/{id}）

`api/index.php` に以下のコードを追記します。

```php
if ($method === "DELETE" && $id !== null) {
  $stmt = $pdo->prepare("DELETE FROM contacts WHERE id = ?");
  $stmt->execute([$id]);
  echo json_encode(["message" => "Deleted"]);
  exit;
}
```

未対応メソッド・不正URLへのエラー処理を最後に追加します。

```php
http_response_code(405);
echo json_encode(["error" => "Method Not Allowed"]);
exit;
```

---

## 動作確認（削除）

```bash
# macOS / Linux / PowerShell 7.x
curl -X DELETE http://localhost:8000/api/contacts/3

# Windows PowerShell 5.1
curl.exe -X DELETE http://localhost:8000/api/contacts/3

# 実行結果
{"message":"Deleted"}
```

---

## 削除結果の確認

```bash
# macOS / Linux / PowerShell 7.x
curl http://localhost:8000/api/contacts

# Windows PowerShell 5.1
curl.exe http://localhost:8000/api/contacts

# 実行結果
[{"id":2,"name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"}]
```

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 全体像を整理する

---

## HTTPメソッド・CRUD・SQLの対応

| HTTPメソッド | CRUD操作 | SQL | PHPの処理 |
|------------|---------|-----|----------|
| GET | Read | SELECT | `query` / `fetch` |
| POST | Create | INSERT | `prepare` + `execute` + `lastInsertId` |
| PUT | Update | UPDATE | `prepare` + `execute` |
| DELETE | Delete | DELETE | `prepare` + `execute` |

セクション2で「メソッドの意味」として紹介した内容が、ここでSQLと結びつきます。

---

## セクション6のまとめ

| 項目 | 内容 |
|------|------|
| PDO | PHPからSQLiteへ接続するための標準的な仕組み |
| プリペアドステートメント | `?` を使った安全なSQL実行方法 |
| ルーティング | URLとメソッドの組み合わせで処理を分岐 |
| CRUD実装 | 全5エンドポイントをPHPで実装 |

次のセクションでは、フロントエンドのHTMLからこのAPIを呼び出し、アドレス帳アプリとして完成させます。
