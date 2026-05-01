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

## セクション 7: フロントエンドと連携する

---

<!-- _class: heading -->
<!-- _footer: "" -->

# フロントエンドの構成

---

## フロントエンドからAPIを呼び出す

このセクションでは、ブラウザ上のHTMLとJavaScriptからバックエンドのAPIを呼び出し、アドレス帳アプリとして完成させます。

ブラウザから連絡先の一覧表示・登録・編集・削除が動作する状態になります。

<div class="text-center">

![h:240px](../assets/section07/fetch-sql.png)

</div>

<!-- ```text
フロントエンド(HTML/JS) ──[fetch]─▶︎ バックエンド(PHP) ──[SQL]─▶︎ SQLite
フロントエンド(HTML/JS) ◀︎─[JSON]── バックエンド(PHP) ◀︎─[結果]── SQLite
``` -->

---

## フロントエンドのファイル構成

```text
project/
├── api/
│   └── index.php       ← セクション6で作ったAPI
├── public/
│   ├── index.html      ← このセクションで作る（画面）
│   └── app.js          ← このセクションで作る（処理）
└── contacts.db         ← セクション5で作ったDB
```

`public/index.html` に画面のHTMLを書き、`public/app.js` に一覧・登録・編集・削除の処理を書きます。

このセクションが終わると、ブラウザから連絡先の一覧表示・登録・編集・削除が動作する状態になります。

---

## fetchの基本

`fetch` は、JavaScriptからHTTPリクエストを送るブラウザ標準の機能です。  
セクション3でcurlを使ってAPIを確認しましたが、fetchはJavaScriptコードの中でそれを行います。

```javascript
// GETリクエスト（最もシンプルな形）
const response = await fetch("/api/contacts");
const data = await response.json();
```

| 部分 | 意味 |
|------|------|
| `fetch("...")` | 指定したURLにHTTPリクエストを送る |
| `await` | レスポンスが返ってくるまで待つ |
| `response.json()` | レスポンスのJSONをJavaScriptのオブジェクトに変換する |

---

## なぜ `await` が必要なのか

HTTPリクエストは時間がかかる処理です。  
`await` を使うことで、レスポンスが返ってくるまで次の処理を待てます。

`await` を使うには、関数を `async` として定義する必要があります。

```javascript
async function loadContacts() {
  const response = await fetch("/api/contacts");
  const data = await response.json();
  console.log(data);
}
```

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 連絡先を表示する

---

## HTMLの基本構造

`public/index.html` を作成します。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>アドレス帳</title>
</head>
<body>
  <h1>アドレス帳</h1>

  <!-- 連絡先一覧 -->
  <section>
    <h2>連絡先一覧</h2>
    <ul id="contact-list"></ul>
  </section>

  <script src="app.js"></script>
</body>
</html>
```

---

## 連絡先の一覧を取得して表示する

`public/app.js` を作成します。

```javascript
async function loadContacts() {
  const response = await fetch("/api/contacts");
  const contacts = await response.json();

  const list = document.getElementById("contact-list");
  list.innerHTML = "";

  contacts.forEach(contact => {
    const li = document.createElement("li");
    const { name, email, phone } = contact;
    li.textContent = `${name} | ${email ?? "-"} | ${phone ?? "-"} `;
    list.appendChild(li);
  });
}

loadContacts();
```

<!-- `li.textContent` を使うのは、入力された文字列をHTMLとして解釈させないためです。セキュリティ上も安全な書き方です（セクション9で詳しく扱います）。 -->

---

## 取得・表示処理のコードの意味

| 部分 | 意味 |
|------|------|
| `await fetch("/api/contacts")` | バックエンドの一覧取得APIにGETリクエストを送る |
| `await response.json()` | 返ってきたJSONをJavaScriptの配列に変換する |
| `list.innerHTML = ""` | 一覧を再表示する前に既存の内容をいったん空にする |
| `contacts.forEach(...)` | 取得した連絡先の数だけ繰り返し処理する |
| `document.createElement("li")` | 連絡先1件分のリスト要素を動的に作る |
| `li.textContent = \`...\`` | 名前・メール・電話番号をテキストとして表示する |
| `list.appendChild(li)` | 作ったリスト要素を画面に追加する |

---

## ブラウザで確認する

PHPサーバーが起動中であることを確認し、ブラウザで以下のURLにアクセスします。

```text
http://localhost:8000/public/index.html
```

セクション5・6でDBに登録した連絡先が一覧表示されれば成功です。

> `fetch("/api/contacts")` がリクエストを送り、PHPが処理してJSONを返しています。
> セクション2で説明した「リクエスト / レスポンス」の流れが、ブラウザ上で動いている瞬間です。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 連絡先を登録する

---

## 登録フォームを追加する

`index.html` の連絡先一覧の上に登録フォームを追加します。

```html
<!-- 連絡先登録フォーム -->
<section>
  <h2>連絡先を登録する</h2>
  <form id="contact-form">
    <div>
      <label>名前（必須）：<input type="text" id="name" required></label>
    </div>
    <div>
      <label>メール：<input type="email" id="email"></label>
    </div>
    <div>
      <label>電話番号：<input type="tel" id="phone"></label>
    </div>
    <button type="submit">登録する</button>
  </form>
</section>
```

---

## フォームを送信する

`app.js` に以下を追加します。

```javascript
document.getElementById("contact-form").addEventListener("submit", async (event) => {
  event.preventDefault();

  const name  = document.getElementById("name").value;
  const email = document.getElementById("email").value;
  const phone = document.getElementById("phone").value;

  await fetch("/api/contacts", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name, email, phone }),
  });

  document.getElementById("contact-form").reset();
  loadContacts();
});
```

---

## 登録処理のコードの意味

| 部分 | 意味 |
|------|------|
| `addEventListener("submit", ...)` | データ送信時の非同期処理を登録する |
| `event.preventDefault()` | フォームのデフォルト動作（ページ遷移）を止める |
| `method: "POST"` | POSTリクエストとして送信する |
| `headers: {...}` | ボディがJSON形式であることを伝える |
| `body: JSON.stringify({...})` | JavaScriptのオブジェクトをJSON文字列に変換して送る |
| `loadContacts()` | 登録後に一覧を再取得して画面を更新する |

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 連絡先を削除する

---

## 削除ボタンを追加する

`public/app.js` の `loadContacts()` の中で、各連絡先に削除ボタンを追加します。  
`list.appendChild(li)` の前に以下を追記します。

```javascript
const deleteButton = document.createElement("button");
deleteButton.textContent = "削除";
deleteButton.addEventListener("click", async () => {
  if (!confirm("削除しますか？")) return;
  await fetch(`/api/contacts/${contact.id}`, { method: "DELETE" });
  loadContacts();
});

li.appendChild(deleteButton);
```

「削除」ボタンを押すと確認アラートが表示され、「OK」をクリックすると連絡先が削除されて一覧が更新されます。

---

## 削除処理のコードの意味

| 部分 | 意味 |
|------|------|
| `document.createElement("button")` | 削除ボタン要素を動的に作る |
| `deleteButton.textContent = "削除"` | ボタンのラベルを「削除」にする |
| `addEventListener("click", ...)` | クリック時の非同期処理を登録する |
| `!confirm("削除しますか？")` | 確認ダイアログを表示し、キャンセルなら処理を止める |
| `method: "DELETE"` | バックエンドの削除APIにDELETEリクエストを送る |
| `loadContacts()` | 削除後に一覧を再取得して画面を更新する |
| `li.appendChild(deleteButton)` | 削除ボタンを連絡先のリスト要素に追加する |

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 連絡先を編集する

---

## 編集フォームを追加する

<!-- 編集には「対象のデータを取得してフォームに表示する」→「変更内容をPUTで送る」という流れが必要です。 -->
`index.html` の連絡先一覧の下に編集フォームを追加します。

```html
<!-- 連絡先編集フォーム（初期状態では非表示） -->
<section id="edit-section" style="display: none;">
  <h2>連絡先を編集する</h2>
  <form id="edit-form">
    <input type="hidden" id="edit-id">
    <div>
      <label>名前（必須）：<input type="text" id="edit-name" required></label>
    </div>
    <div>
      <label>メール：<input type="email" id="edit-email"></label>
    </div>
    <div>
      <label>電話番号：<input type="tel" id="edit-phone"></label>
    </div>
    <button type="submit">更新する</button>
    <button type="button" id="cancel-edit">キャンセル</button>
  </form>
</section>
```

<!-- 初期状態では `display: none` で非表示にしておきます。 -->

---

## 編集ボタンを追加する

`public/app.js` の `loadContacts()` の中で、各連絡先に編集ボタンを追加します。  
`li.appendChild(deleteButton)` の前に以下を追記します。

```javascript
const editButton = document.createElement("button");
editButton.textContent = "編集";
editButton.addEventListener("click", () => {
  document.getElementById("edit-section").style.display = "block";
  document.getElementById("edit-id").value    = contact.id;
  document.getElementById("edit-name").value  = contact.name;
  document.getElementById("edit-email").value = contact.email ?? "";
  document.getElementById("edit-phone").value = contact.phone ?? "";
});

li.appendChild(editButton);
```

「編集」ボタンを押すと、編集フォームが表示され対象の値が入力された状態になります。

---

## 編集ボタン追加のコードの意味

| 部分 | 意味 |
|------|------|
| `document.createElement("button")` | 編集ボタン要素を動的に作る |
| `editButton.textContent = "編集"` | ボタンのラベルを「編集」にする |
| `addEventListener("click", () => {...})` | クリック時の処理を登録する |
| `style.display = "block"` | 編集フォームを表示する |
| `edit-id`, `edit-name`, ... `.value = ...` | フォームに対象の連絡先の値をセットする |
| `li.appendChild(editButton)` | 編集ボタンを連絡先のリスト要素に追加する |

---

## 更新処理を追加する

`public/app.js` にフォームの送信処理を追加します。

```javascript
document.getElementById("edit-form").addEventListener("submit", async (event) => {
  event.preventDefault();
  const id = document.getElementById("edit-id").value;

  await fetch(`/api/contacts/${id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name:  document.getElementById("edit-name").value,
      email: document.getElementById("edit-email").value,
      phone: document.getElementById("edit-phone").value,
    }),
  });

  document.getElementById("edit-section").style.display = "none";
  loadContacts();
});

document.getElementById("cancel-edit").addEventListener("click", () => {
  document.getElementById("edit-section").style.display = "none";
});
```

<!-- 一覧の「編集」ボタンを押すと編集フォームが表示されます。  
名前などを変更して「更新する」ボタンを押すと一覧が更新されれば成功です。 -->

---

## 更新処理のコードの意味

| 部分 | 意味 |
|------|------|
| `addEventListener("submit", ...)` | フォーム送信時の非同期処理を登録する |
| `event.preventDefault()` | フォームのデフォルト動作（ページ遷移）を止める |
| `getElementById("edit-id").value` | 非表示フィールドから編集対象のIDを取得する |
| `method: "PUT"` | PUTリクエストとして送信する |
| `body: JSON.stringify({...})` | フォームの入力値をJSON文字列に変換して送る |
| `style.display = "none"` | 更新後に編集フォームを非表示にする |
| `loadContacts()` | 更新後に一覧を再取得して画面を更新する |

---

## セクション7のまとめ

| 操作 | JavaScript | PHPのエンドポイント |
|------|-----------|------------------|
| 表示 | `fetch("/api/contacts")` | GET /api/contacts |
| 登録 | `fetch(..., { method: "POST", body: ... })` | POST /api/contacts |
| 削除 | `fetch(..., { method: "DELETE" })` | DELETE /api/contacts/{id} |
| 更新 | `fetch(..., { method: "PUT", body: ... })` | PUT /api/contacts/{id} |

次のセクションでは、現在のコードの問題点を整理し、設計と改善について学びます。

<!-- ### 現時点の課題 -->

- `api/index.php` が長く、すべての処理が1ファイルに詰まっている
- バリデーション（入力チェック）が最小限
- セキュリティ上の問題が残っている可能性がある
