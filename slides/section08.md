---
marp: true
theme: udemy
footer: "<div>AI時代のためのバックエンド開発入門</div>"
paginate: true
---

<!-- _class: title -->
<!-- _footer: "" -->
<!-- _paginate: skip -->

# AI 時代のためのバックエンド開発入門

## セクション 8: 設計と改善

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 現在のコードを振り返る

---

## api/index.php の問題点

現在の `api/index.php` には、次のすべての処理が1ファイルに詰まっています。

- データベースへの接続
- URLとメソッドの解析（ルーティング）
- 一覧取得・1件取得・登録・更新・削除の5つの処理

処理が増えるほどファイルが長くなり、どこに何があるかが分かりにくくなります。

今後ファイルを分けていくと、DB接続のような共通処理を複数の場所に書いてしまう可能性があります。

---

## バリデーション不足の問題

現状のコードには**バリデーション（入力チェック）が不足しています**。

ブラウザのフォームでは `required` 属性があるため、通常は空のまま送信できません。  
しかし、APIはブラウザのフォーム以外からも呼び出せます。

`curl` を使えば、フロントエンドのチェックを通らずに直接APIへリクエストを送れます。

> 名前が空のまま登録フォームを送信した場合、どうなるでしょうか？

---

<!-- _class: tight -->

## 実際に名前を空のまま送信する

#### macOS / Linux

```
curl -X POST http://localhost:8000/api/contacts \
  -H "Content-Type: application/json" \
  -d '{"name":"","email":"test@example.com"}'
```

#### PowerShell 7.x

```
curl -X POST http://localhost:8000/api/contacts `
  -H "Content-Type: application/json" `
  -d '{"name":"","email":"test@example.com"}'
```

#### Windows PowerShell 5.1

```
curl.exe -X POST http://localhost:8000/api/contacts `
  -H "Content-Type: application/json" `
  -d '{"""name""":"""""","""email""":"""test@example.com"""}'
```

結果：`{"id":"4","message":"Created"}` データの登録に成功します。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 責務分離

---

## 各層の役割を明確にする

「責務分離」とは、**それぞれのコードが担当する役割を明確に分けること**です。

| 層 | 担当する責務 |
|---|------------|
| フロントエンド | 画面の表示・ユーザーの入力受け取り・APIの呼び出し |
| バックエンド | データの検証・処理・DBへの命令 |
| データベース | データの永続的な保存 |

---

## バリデーションはどこで行うか

バリデーションは、フロントエンドとバックエンドの両方で行います。

| 場所 | 役割 | 例 |
|------|------|---|
| フロントエンド | 即時フィードバックのようなUXの改善 | `required` 属性・JavaScriptでのチェック |
| バックエンド | データの正確性を保証する | PHPでの必須チェック・形式チェック |

<div class="important">

> **サーバーサイドのバリデーションが本番です。**
> フロントエンドのバリデーションはあくまで補助です。  最終的にバックエンドがデータの正確性を保証する責務を持ちます。

</div>

---

<!-- _class: heading -->
<!-- _footer: "" -->

# バリデーションを追加する

---

## name の必須チェック（POST）

登録（POST）の処理に、`name` が空でないかを確認する処理を追加します。

```php
if ($method === "POST") {
  $input = json_decode(file_get_contents("php://input"), true);

  if (empty($input["name"])) {
    http_response_code(400);
    echo json_encode(["error" => "name is required"]);
    exit;
  }

  // 以降は既存のINSERT処理...
}
```

---

## name の必須チェック（PUT）

更新（PUT）の処理にも同様のチェックを追加します。

```php
if ($method === "PUT" && $id !== null) {
  $input = json_decode(file_get_contents("php://input"), true);

  if (empty($input["name"])) {
    http_response_code(400);
    echo json_encode(["error" => "name is required"]);
    exit;
  }

  // 以降は既存のUPDATE処理...
}
```

---

<!-- _class: tight -->

## 確認する

curlで `name` を空にして送信します。

#### macOS / Linux

```
curl -X POST http://localhost:8000/api/contacts \
  -H "Content-Type: application/json" \
  -d '{"name":"","email":"test@example.com"}'
```

#### PowerShell 7.x

```
curl -X POST http://localhost:8000/api/contacts `
  -H "Content-Type: application/json" `
  -d '{"name":"","email":"test@example.com"}'
```

#### Windows PowerShell 5.1

```
curl.exe -X POST http://localhost:8000/api/contacts `
  -H "Content-Type: application/json" `
  -d '{"""name""":"""""","""email""":"""test@example.com"""}'
```

結果：`{"error":"name is required"}`  HTTPステータスコードが `400` で返ってきます。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# DB接続を切り出す

---

## ファイル単位でも責務を分ける

層レベルの責務分離と同じ考え方は、**ファイル単位**でも重要です。

「1ファイルには1つの役割」を意識することで、コードの見通しが良くなります。

| ファイル | 責務 |
|---------|------|
| `index.php` | ルーティング・各CRUDの処理 |
| `db.php` | DB接続の管理 |

DB接続は「接続を管理する」という独立した責務です。`index.php` に混在させず、専用のファイルに切り出します。

---

<!-- _class: tight -->

## db.php に切り出す

`api/db.php` を新規作成します。

```php
<?php
function get_pdo(): PDO {
  $pdo = new PDO('sqlite:../contacts.db');
  $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
  return $pdo;
}
```

`api/index.php` の冒頭を書き換えます。

```php
<?php
header("Content-Type: application/json");

require_once __DIR__ . '/db.php';
$pdo = get_pdo();
```

---

## 変更の意味

| 変更前 | 変更後 |
|-------|-------|
| `index.php` に直接 `new PDO(...)` | `db.php` の `get_pdo()` を呼ぶ |
| DBパスの変更は `index.php` を直接編集 | `db.php` だけを修正すればよい |

この変更は機能には影響しません。「DBの設定は `db.php` に書く」というルールができることで、コードの見通しが良くなります。

> **リファクタリング**とは、動作を変えずにコードの構造だけを改善します。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# API設計を振り返る

---

## 設計時の判断を確認する

セクション2で設計したAPIエンドポイントを振り返ります。

```text
GET    /api/contacts        → 一覧取得
GET    /api/contacts/{id}   → 1件取得
POST   /api/contacts        → 登録
PUT    /api/contacts/{id}   → 更新
DELETE /api/contacts/{id}   → 削除
```

| 判断 | 理由 |
|------|------|
| URLにリソース名（contacts）を使う | 「何を操作するか」がURLで分かる |
| 操作の種類はHTTPメソッドで表す | URLを増やさずに操作を区別できる |
| IDはURLのパスに含める | `/contacts/1` と `/contacts/2` を区別できる |

---

## 設計がないと何が起きるか

設計のルールを決めずに作ると、次のようなAPIになることがあります。

```text
/getContacts
/addContact
/deleteContact?id=1
/updateContactById
```

動きはしますが、命名に一貫性がなく、他の人（や未来の自分）が使いにくくなります。

RESTful API でよく使われる設計ルールは、こうした問題を防ぐための**共通の考え方**です。

---

## セクション8のまとめ

| 項目 | 内容 |
|------|------|
| 責務分離 | フロント・バックエンド・DBそれぞれの役割を明確にする |
| バリデーション | サーバーサイドでのチェックが最終的な保証になる |
| リファクタリング | `db.php` への切り出しで、修正箇所を1か所にまとめる |
| API設計 | 命名の一貫性を保つことで、コードの見通しが良くなる |

動くようになったアプリですが、**セキュリティ上の脆弱性が残っている可能性があります**。
次のセクションでは、セキュリティ上の脆弱性を狙った攻撃の仕組みとその対策を学びます。
