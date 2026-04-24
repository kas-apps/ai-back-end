# セクション7：フロントエンドと連携する

## このセクションの目的

セクション6では、curlコマンドでAPIを呼び出してCRUDの動作を確認しました。  
このセクションでは、ブラウザ上のHTMLとJavaScriptからそのAPIを呼び出し、アドレス帳アプリとして完成させます。

このセクションが終わると、ブラウザから連絡先の一覧表示・登録・編集・削除が動作する状態になります。

```text
フロントエンド(HTML/JS) ──[fetch]─▶ バックエンド(PHP) ──[SQL]─▶ SQLite
フロントエンド(HTML/JS) ◀─[JSON]── バックエンド(PHP) ◀─[結果]── SQLite
```

---

## フロントエンドのファイル構成

このセクションでは `public/` フォルダにフロントエンドのファイルを作ります。

```text
project/
├── api/
│   └── index.php       ← セクション6で作ったAPI
├── public/
│   ├── index.html      ← このセクションで作るフロントエンド（画面）
│   └── app.js          ← このセクションで作るフロントエンド（処理）
└── contacts.db         ← セクション5で作ったDB

```

`public/index.html` に画面のHTMLを書き、`public/app.js` に一覧・登録・編集・削除の処理を書きます。

---

## fetch APIとは何か

### JavaScriptからHTTPリクエストを送る

`fetch` は、JavaScriptからHTTPリクエストを送るブラウザ標準の機能です。  
セクション3でcurlを使ってAPIを確認しましたが、fetchはJavaScriptコードの中でそれを行います。

### 基本的な書き方

```javascript
// GETリクエスト（最もシンプルな形）
const response = await fetch("/api/contacts");
const data = await response.json();
```

| 部分 | 意味 |
|------|------|
| `fetch("...")` | 指定したURLにHTTPリクエストを送る |
| `await` | レスポンスが返ってくるまで待つ |
| `response.json()` | レスポンスのJSON文字列をJavaScriptのオブジェクトに変換する |

### なぜ `await` が必要なのか

HTTPリクエストは時間がかかる処理です。  
`await` を使うことで、レスポンスが返ってくるまで次の処理を待たせることができます。

`await` を使うには、その関数を `async` として定義する必要があります。

```javascript
async function loadContacts() {
  const response = await fetch("/api/contacts");
  const data = await response.json();
  console.log(data);
}
```

---

## 連絡先一覧を表示する

### HTMLの基本構造

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

### JavaScriptで一覧を取得・表示する

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

> **fetch のエラー処理について**  
> 今回はコードをシンプルにするため、エラー処理は最小限にしています。
> うまく動かない場合は、ブラウザの開発者ツールのConsoleとNetworkタブで、`/api/contacts` のレスポンスを確認します。
>
> **`li.textContent` について**  
> ここでは `innerHTML` ではなく `textContent` を使っています。
> 入力された文字列をHTMLとして解釈させないため、セキュリティ上も安全な書き方です。詳しくはセクション9で扱います。

### 確認する

PHPサーバーが起動中であることを確認し、ブラウザで以下にアクセスします。

```text
http://localhost:8000/public/index.html
```

セクション5・6でDBに登録した連絡先が一覧表示されれば成功です。

> **セクション2との接続：**  
> `fetch("/api/contacts")` がリクエストを送り、PHPが処理してJSONを返しています。  
> セクション2で説明した「リクエスト/レスポンス」の流れが、ブラウザ上で動いている瞬間です。

---

## 連絡先を登録する

### フォームを追加する

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

### フォーム送信をJavaScriptで処理する

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

### コードの意味

| 部分 | 意味 |
|------|------|
| `event.preventDefault()` | フォームのデフォルト動作（ページ遷移）を止める |
| `method: "POST"` | POSTリクエストとして送信する |
| `headers: { "Content-Type": "application/json" }` | ボディがJSON形式であることを伝える |
| `body: JSON.stringify({...})` | JavaScriptのオブジェクトをJSON文字列に変換して送る |
| `loadContacts()` | 登録後に一覧を再取得して画面を更新する |

### 確認する

フォームに名前・メールアドレス・電話番号を入力して「登録する」ボタンを押します。  
一覧に新しい連絡先が追加されれば成功です。

---

## 連絡先を削除する

### 削除ボタンを追加する

`loadContacts()` 関数の中で、各連絡先に削除ボタンを追加します。
`list.appendChild(li);` （連絡先リストの追加）の上に以下のコードを追記します。

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

### 確認する

一覧の各連絡先に「削除」ボタンが表示されます。  
ボタンを押すと連絡先が削除され、一覧が更新されれば成功です。

---

## 連絡先を編集する

### 編集フォームを追加する

編集には「対象のデータを取得してフォームに表示する」→「変更内容をPUTで送る」という流れが必要です。  
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

### 編集ボタンと更新処理を追加する

`loadContacts()` に編集ボタンを追加します。
`li.appendChild(deleteButton);` （削除ボタンの追加）の上に以下のコードを追記します。

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

フォームの送信処理を追加します。

```javascript
document.getElementById("edit-form").addEventListener("submit", async (event) => {
  event.preventDefault();

  const id    = document.getElementById("edit-id").value;
  const name  = document.getElementById("edit-name").value;
  const email = document.getElementById("edit-email").value;
  const phone = document.getElementById("edit-phone").value;

  await fetch(`/api/contacts/${id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name, email, phone }),
  });

  document.getElementById("edit-section").style.display = "none";
  loadContacts();
});

document.getElementById("cancel-edit").addEventListener("click", () => {
  document.getElementById("edit-section").style.display = "none";
});
```

### 確認する

一覧の「編集」ボタンを押すと編集フォームが表示されます。  
名前などを変更して「更新する」ボタンを押すと一覧が更新されれば成功です。

---

## index.html の全体像

```html
<!DOCTYPE html>
<html lang="ja">

<head>
  <meta charset="UTF-8">
  <title>アドレス帳</title>
</head>

<body>
  <h1>アドレス帳</h1>

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

  <!-- 連絡先一覧 -->
  <section>
    <h2>連絡先一覧</h2>
    <ul id="contact-list"></ul>
  </section>

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

  <script src="app.js"></script>
</body>

</html>
```

## app.js の全体像

```javascript
// 一覧を取得して表示する
async function loadContacts() {
  const response = await fetch("/api/contacts");
  const contacts = await response.json();

  const list = document.getElementById("contact-list");
  list.innerHTML = "";

  contacts.forEach(contact => {
    const li = document.createElement("li");
    const { name, email, phone } = contact;
    li.textContent = `${name} | ${email ?? "-"} | ${phone ?? "-"} `;

    // 編集ボタン
    const editButton = document.createElement("button");
    editButton.textContent = "編集";
    editButton.addEventListener("click", () => {
      document.getElementById("edit-section").style.display = "block";
      document.getElementById("edit-id").value    = contact.id;
      document.getElementById("edit-name").value  = contact.name;
      document.getElementById("edit-email").value = contact.email ?? "";
      document.getElementById("edit-phone").value = contact.phone ?? "";
    });

    // 削除ボタン
    const deleteButton = document.createElement("button");
    deleteButton.textContent = "削除";
    deleteButton.addEventListener("click", async () => {
      if (!confirm("削除しますか？")) return;
      await fetch(`/api/contacts/${contact.id}`, { method: "DELETE" });
      loadContacts();
    });

    li.appendChild(editButton);
    li.appendChild(deleteButton);
    list.appendChild(li);
  });
}

// 登録フォームの送信
document.getElementById("contact-form").addEventListener("submit", async (event) => {
  event.preventDefault();
  await fetch("/api/contacts", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name:  document.getElementById("name").value,
      email: document.getElementById("email").value,
      phone: document.getElementById("phone").value,
    }),
  });
  document.getElementById("contact-form").reset();
  loadContacts();
});

// 編集フォームの送信
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

// 編集キャンセル
document.getElementById("cancel-edit").addEventListener("click", () => {
  document.getElementById("edit-section").style.display = "none";
});

// 初期表示
loadContacts();
```

---

## セクション7のまとめ

| 操作 | JavaScript | PHPのエンドポイント |
|------|-----------|------------------|
| 一覧表示 | `fetch("/api/contacts")` | GET /api/contacts |
| 登録 | `fetch(..., { method: "POST", body: ... })` | POST /api/contacts |
| 更新 | `fetch(..., { method: "PUT", body: ... })` | PUT /api/contacts/{id} |
| 削除 | `fetch(..., { method: "DELETE" })` | DELETE /api/contacts/{id} |

これで、フロントエンドからAPIを呼び出す簡易アドレス帳アプリが完成しました。

### 現時点のアプリでできること

- ブラウザから連絡先を登録・表示・編集・削除できる
- データはSQLiteに保存されるため、リロードしても消えない
- curlでAPIを叩いても、ブラウザからでも同じデータが使える

### 現時点の課題

動いてはいますが、コードを改めて見ると気になる点があります。

- `api/index.php` が長く、すべての処理が1ファイルに詰まっている
- バリデーション（入力チェック）が最小限
- セキュリティ上の問題が残っている可能性がある

次のセクションでは、現在のコードの問題点を整理し、設計と改善について学びます。
