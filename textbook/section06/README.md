# セクション6：データベースを操作するAPIを作る

## このセクションの目的

セクション5では、ターミナルから直接SQLiteを操作しました。  
このセクションでは、PHPからSQLiteに接続し、セクション2で設計したAPIエンドポイントを実際に実装します。

```text
フロントエンド ──[HTTPリクエスト]─▶ バックエンド(PHP) ──[SQL]─▶ SQLite
フロントエンド ◀─[JSONレスポンス]── バックエンド(PHP) ◀─[結果]── SQLite
```

このセクションが終わると、curlやブラウザからAPIを呼び出して、データの保存・取得・更新・削除が動作する状態になります。

---

## PDOとは何か

### PHPからデータベースに接続する仕組み

PHPからSQLiteに接続するには **PDO（PHP Data Objects）** を使います。  
PDOはPHPに標準で組み込まれたデータベース接続のための仕組みです。

### なぜPDOを使うのか

PHPでSQLiteに接続する方法はいくつかありますが、PDOには次のメリットがあります。

| メリット | 説明 |
|---------|------|
| 統一されたインターフェース | SQLite・MySQL・PostgreSQLなど、DBが変わっても書き方が同じ |
| プリペアドステートメントが使える | SQLインジェクション対策として重要（セクション9で詳しく扱います） |
| PHPに標準搭載 | 追加インストール不要 |

### PDOでSQLiteに接続する

```php
$pdo = new PDO('sqlite:contacts.db');
```

- `sqlite:` の後にDBファイルのパスを指定します
- `contacts.db` はセクション5で作ったファイルです

> `contacts.db` のパスは、PHPファイルの実行場所を基準とした相対パスです。  
> `api/index.php` から見て `contacts.db` が `project/contacts.db` にある場合は、`'sqlite:../contacts.db'` と書きます。

---

## APIのファイル構成

このセクションでは、`api/index.php` 1ファイルにすべてのエンドポイントをまとめます。

URLとHTTPメソッドの組み合わせで処理を分岐させる設計です。

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

まず、`api/index.php` の共通部分（DB接続・ルーティングの骨格）を作ります。

```php
<?php
header("Content-Type: application/json");

$pdo = new PDO('sqlite:../contacts.db');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$method = $_SERVER["REQUEST_METHOD"];
$uri    = parse_url($_SERVER["REQUEST_URI"], PHP_URL_PATH);

// /api/contacts または /api/contacts/1 の形式にマッチ
if (preg_match('#^/api/contacts(?:/(\d+))?$#', $uri, $matches)) {
  $id = $matches[1] ?? null;
} else {
  http_response_code(404);
  echo json_encode(["error" => "Not Found"]);
  exit;
}
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `PDO::ATTR_ERRMODE` | エラー発生時に例外を投げる設定 |
| `$_SERVER["REQUEST_METHOD"]` | GET / POST / PUT / DELETE のいずれかが入る |
| `parse_url(..., PHP_URL_PATH)` | URLからパス部分だけを取り出す |
| `preg_match(...)` | 正規表現でURLのパターンを確認し、`{id}` 部分を取り出す |
| `$id` | URLに `/1` のような数字があればその値、なければ `null` |

---

## 一覧取得 API（GET /api/contacts）

### 実装

```php
if ($method === "GET" && $id === null) {
  $stmt = $pdo->query("SELECT * FROM contacts");
  $contacts = $stmt->fetchAll(PDO::FETCH_ASSOC);
  echo json_encode($contacts, JSON_UNESCAPED_UNICODE);
  exit;
}
```

### 確認

```bash
curl http://localhost:8000/api/contacts
```

```json
[
  {"id":"1","name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"}
]
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `$pdo->query(...)` | SQLを実行し、結果オブジェクトを返す |
| `fetchAll(PDO::FETCH_ASSOC)` | 全行をカラム名をキーとした連想配列で取得する |

---

## 1件取得 API（GET /api/contacts/{id}）

### 実装

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

### 確認

```bash
curl http://localhost:8000/api/contacts/1
```

```json
{"id":"1","name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"}
```

存在しないIDの場合：

```bash
curl http://localhost:8000/api/contacts/999
```

```json
{"error":"Not Found"}
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `$pdo->prepare("... WHERE id = ?")` | `?` をプレースホルダーとした準備済みSQL |
| `$stmt->execute([$id])` | `?` の部分に `$id` の値を渡して実行する |
| `fetch(PDO::FETCH_ASSOC)` | 1行だけをカラム名をキーとした連想配列で取得する |
| `=== false` | データが見つからなかった場合の判定 |

> **プリペアドステートメントとは**  
> `?` を使った書き方をプリペアドステートメントと呼びます。  
> URLや入力から来た値を直接SQLに埋め込まず、PDOが安全に処理します。  
> SQLインジェクションという攻撃への対策として重要です（セクション9で詳しく扱います）。

---

## 登録 API（POST /api/contacts）

### 実装

```php
if ($method === "POST") {
  $input = json_decode(file_get_contents("php://input"), true);

  $stmt = $pdo->prepare(
    "INSERT INTO contacts (name, email, phone) VALUES (?, ?, ?)"
  );
  $stmt->execute([
    $input["name"],
    $input["email"] ?? null,
    $input["phone"] ?? null,
  ]);

  $newId = $pdo->lastInsertId();
  http_response_code(201);
  echo json_encode(["id" => $newId, "message" => "Created"]);
  exit;
}
```

> 今回は学習を優先するため、入力チェックは最小限にしています。実際のアプリでは、`name` が空でないか、メールアドレスの形式が正しいかを確認します。

### 確認

```bash
curl -X POST http://localhost:8000/api/contacts \
  -H "Content-Type: application/json" \
  -d '{"name":"田中 太郎","email":"taro@example.com","phone":"090-1111-2222"}'
```

```json
{"id":"2","message":"Created"}
```

一覧取得で確認します。

```bash
curl http://localhost:8000/api/contacts
```

```json
[
  {"id":"1","name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"},
  {"id":"2","name":"田中 太郎","email":"taro@example.com","phone":"090-1111-2222"}
]
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `file_get_contents("php://input")` | リクエストボディ（JSON文字列）を取得する |
| `json_decode(..., true)` | JSON文字列をPHPの連想配列に変換する |
| `$input["email"] ?? null` | `email` キーがない場合は `null` を使う |
| `lastInsertId()` | 直前のINSERTで発行されたIDを取得する |
| `http_response_code(201)` | 「作成成功」を意味するHTTPステータスコードを返す |

---

## 更新 API（PUT /api/contacts/{id}）

### 実装

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

### 確認

```bash
curl -X PUT http://localhost:8000/api/contacts/2 \
  -H "Content-Type: application/json" \
  -d '{"name":"田中 次郎","email":"jiro@example.com","phone":"090-3333-4444"}'
```

```json
{"message":"Updated"}
```

```bash
curl http://localhost:8000/api/contacts/2
```

```json
{"id":"2","name":"田中 次郎","email":"jiro@example.com","phone":"090-3333-4444"}
```

---

## 削除 API（DELETE /api/contacts/{id}）

### 実装

```php
if ($method === "DELETE" && $id !== null) {
  $stmt = $pdo->prepare("DELETE FROM contacts WHERE id = ?");
  $stmt->execute([$id]);

  echo json_encode(["message" => "Deleted"]);
  exit;
}
```

### 確認

```bash
curl -X DELETE http://localhost:8000/api/contacts/2
```

```json
{"message":"Deleted"}
```

```bash
curl http://localhost:8000/api/contacts
```

```json
[
  {"id":"1","name":"鈴木 花子","email":"hanako@example.com","phone":"080-9876-5432"}
]
```

### 未対応メソッド・不正URLへのレスポンス

例えば `POST /api/contacts/1` や `PUT /api/contacts` のような未対応メソッド・不正URLに対するエラー処理を最後に追加します。

```php
http_response_code(405);
echo json_encode(["error" => "Method Not Allowed"]);
exit;
```

---

## api/index.php の全体像

ここまでの実装をまとめると、`api/index.php` は次のようになります。

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

// 一覧取得
if ($method === "GET" && $id === null) {
  $stmt = $pdo->query("SELECT * FROM contacts");
  echo json_encode($stmt->fetchAll(PDO::FETCH_ASSOC), JSON_UNESCAPED_UNICODE);
  exit;
}

// 1件取得
elseif ($method === "GET" && $id !== null) {
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

// 登録
elseif ($method === "POST") {
  $input = json_decode(file_get_contents("php://input"), true);
  $stmt  = $pdo->prepare("INSERT INTO contacts (name, email, phone) VALUES (?, ?, ?)");
  $stmt->execute([$input["name"], $input["email"] ?? null, $input["phone"] ?? null]);
  http_response_code(201);
  echo json_encode(["id" => $pdo->lastInsertId(), "message" => "Created"]);
  exit;
}

// 更新
elseif ($method === "PUT" && $id !== null) {
  $input = json_decode(file_get_contents("php://input"), true);
  $stmt  = $pdo->prepare("UPDATE contacts SET name = ?, email = ?, phone = ? WHERE id = ?");
  $stmt->execute([$input["name"], $input["email"] ?? null, $input["phone"] ?? null, $id]);
  echo json_encode(["message" => "Updated"]);
  exit;
}

// 削除
elseif ($method === "DELETE" && $id !== null) {
  $stmt = $pdo->prepare("DELETE FROM contacts WHERE id = ?");
  $stmt->execute([$id]);
  echo json_encode(["message" => "Deleted"]);
  exit;
}
```

---

## CRUDをまとめる（設計の視点）

### セクション5との対応

ターミナルで実行したSQLが、PHPのコードにどう対応しているかを整理します。

| 操作 | SQLite直接 | PHP（PDO）|
|------|-----------|----------|
| 全件取得 | `SELECT * FROM contacts;` | `$pdo->query("SELECT ...")` |
| 1件取得 | `SELECT * FROM contacts WHERE id = 1;` | `$pdo->prepare(...)->execute([$id])` |
| 登録 | `INSERT INTO contacts ...;` | `prepare` + `execute` + `lastInsertId()` |
| 更新 | `UPDATE contacts SET ... WHERE id = 1;` | `prepare` + `execute` |
| 削除 | `DELETE FROM contacts WHERE id = 1;` | `prepare` + `execute` |

### HTTPメソッドとCRUDの対応

| HTTPメソッド | CRUD操作 | SQL |
|------------|---------|-----|
| GET | Read | SELECT |
| POST | Create | INSERT |
| PUT | Update | UPDATE |
| DELETE | Delete | DELETE |

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
