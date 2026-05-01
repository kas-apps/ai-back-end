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

## セクション 3: PHPで最小のバックエンドを作る

<!-- ## このセクションのゴール

### セクション2で学んだこと

HTTP・リクエスト/レスポンス・JSON・API という概念

### このセクションでやること

それらの概念を **PHPのコードとして動かす** 体験をします。

「URLにアクセスするとJSONが返ってくるAPI」を自分で作ります。 -->

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 環境を整える

---

## 開発環境の全体像

このセクションで使うものは3つだけです。

| ツール | 役割 |
|--------|------|
| VSCode | コードを書くエディタ |
| PHP | バックエンドを動かすプログラミング言語 |
| PHP内蔵サーバー | ローカルでHTTPサーバーを動かす仕組み |

PHPには開発用の内蔵サーバーが用意されており、コマンド1行で起動できます。  
この講座では、環境構築のハードルを下げるためにこれを使います。

---

## VSCodeの準備

### インストール

Microsoftが提供する無料のコードエディタです。  
公式サイトからインストーラーをダウンロードして実行します。

### 使い方の基本

- **フォルダを開く** → プロジェクトの起点になります
- **ターミナルを開く** → メニューの「ターミナル」→「新しいターミナル」

### 推奨拡張機能

**PHP Intelephense** を入れるとコード補完が効いて書きやすくなります（なくても動作に問題はありません）。

---

## Homebrewのインストール（macOS）

### Homebrewとは

macOSやLinuxで動作するパッケージ管理ツールです。Homebrewを使うとターミナルにコマンドを打ち込むだけで、さまざまなツールを簡単にインストール・管理できるようになります。

### インストール

ターミナルで以下のコマンドを実行します。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> インストールコマンドは[公式ページ（https://brew.sh/ja/）](https://brew.sh/ja/)から確認できます。

---

## Scoopのインストール（Windows）

### Scoopとは

Windowsで利用できるパッケージ管理ツールです。macOSのHomebrewのように、PowerShellにコマンドを入力して、さまざまなツールをインストール・管理できます。

### インストール

PowerShell（バージョン5.1以降）を開き、`PS C:\>` プロンプトから以下を実行します。

```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

> インストールコマンドは[公式ページ（https://scoop.sh/）](https://scoop.sh/)から確認できます。

---

## PHPのインストール（macOS）

Homebrewを使ってインストールします。

```bash
brew install php
```

### インストールの確認

```bash
php -v
```

`PHP 8.x.x (cli)` と表示されれば成功です。

> エラーになる場合は、ターミナルを一度閉じて開き直してください。

---

## PHPのインストール（Windows）

Scoopを使ってインストールします。

```bash
scoop install php
```

### インストールの確認

```bash
php -v
```

`PHP 8.x.x (cli)` と表示されれば成功です。

> エラーになる場合は、ターミナルを一度閉じて開き直してください。

---

## プロジェクト構成を作る

```text
project/
├── api/
│   └── index.php
└── public/
```

| フォルダ / ファイル | 役割 |
|-----------------|------|
| `api/` | バックエンドのPHPファイルを置く場所 |
| `api/index.php` | 最初に作るAPIファイル |
| `public/` | フロントエンドのHTMLファイルを置く場所 |

`api/` と `public/` を分けるのは、**役割を明確に分離するため**です。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# バックエンドを動かす

---

## 最初のPHPコードを書く

`api/index.php` を作成して、以下のコードを書きます。

```php
<?php
echo "Hello, Backend!";
```

| 部分 | 意味 |
|------|------|
| `<?php` | ここからPHPのコードが始まるという宣言 |
| `echo` | 文字列を出力する命令 |

まだAPIではありません。PHPが動くことを確認するための第一歩です。

---

## PHP内蔵サーバーを起動する

`project/` フォルダでターミナルを開き、以下を実行します。

```bash
php -S localhost:8000
```

| 部分 | 意味 |
|------|------|
| `php -S` | PHP内蔵サーバーを起動するオプション |
| `localhost` | 自分のコンピューター上で動かす |
| `8000` | 使用するポート番号 |

---

## ブラウザで確認する

ブラウザで以下のURLにアクセスします。

```text
http://localhost:8000/api/index.php
```

`Hello, Backend!` と表示されれば成功です。

<div class="text-center">

![h:240px](../assets/section03/browser-hello.png)

</div>

---

## 「リクエスト / レスポンス」が動いている瞬間

ブラウザで確認したとき、何が起きていたのか？

<div class="text-center">

![h:300px](../assets/section03/browser-php_server.png)

</div>

<!-- ```text
ブラウザ（クライアント） ───[リクエスト]──▶︎ PHPサーバー（バックエンド）
ブラウザ（クライアント） ◀︎──[レスポンス]─── PHPサーバー（バックエンド）
``` -->

セクション2で学んだ「リクエスト / レスポンス」が、  実際に動いている瞬間です。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# JSONを返すAPIにする

---

## コードを変更する

`api/index.php` の内容を以下に書き換えます。

```php
<?php
header("Content-Type: application/json");

echo json_encode([
  "message" => "Hello, API!"
]);
```

| 部分 | 意味 |
|------|------|
| `header(...)` | レスポンスの形式がJSONであることを伝える |
| `json_encode(...)` | PHPの配列をJSON文字列に変換する |

ブラウザで同じURLにアクセスすると `{"message":"Hello, API!"}` が返ってきます。

---

## curlでAPIを確認する

### curlとは

`curl` は、ターミナルからHTTPリクエストを送るコマンドです。
<!-- ブラウザを使わずにAPIの動作を確認できます。 -->
<!-- Windowsでは、使用しているPowerShellのバージョンによってコマンドが変わるので注意 -->

### macOS / Linux / PowerShell 7.x

```bash
curl http://localhost:8000/api/index.php
```

### Windows PowerShell 5.1

`curl` コマンドに `.exe` を付けます。

```bash
curl.exe http://localhost:8000/api/index.php
```

<!-- PowerShell 5.1 では、`curl` は `Invoke-WebRequest` という PowerShell コマンドの 「エイリアス（別名）」 です。そのため、Linux の `curl` と同じ感覚でオプション（-d や -H など）を使うとエラーになります。 -->

---

## curlの実行結果

ターミナルに以下のようなレスポンスが表示されます。

```json
{"message":"Hello, API!"}
```

**ここが「API」です。**  HTMLではなくJSONを返すようになりました。  セクション2で学んだ「なぜHTMLではなくJSONなのか」が、ここで実感できます。

> ブラウザとcurlの両方でAPIを確認できることで、「APIはURLにアクセスすれば使えるもの」という理解が深まります。

---

<!-- _class: heading -->
<!-- _footer: "" -->

# 簡単なエンドポイントを作る

---

## URLで処理を分ける（ルーティング）

<!-- URLごとに異なる処理を返します。 -->
`api/index.php` の `header(...)` 以降の内容を書き換えます。

```php
$path = $_SERVER["REQUEST_URI"];

if ($path === "/api/hello") {
  echo json_encode(["message" => "Hello"]);
} else {
  http_response_code(404);
  echo json_encode(["error" => "Not Found"]);
}
```

| 部分 | 意味 |
|------|------|
| `$_SERVER["REQUEST_URI"]` | アクセスされたURLのパス部分を取得する |
| `http_response_code(404)` | HTTPステータスコードを指定する |

---

## 動作を確認する

以下のURLにアクセスして、それぞれ異なるレスポンスが返ることを確認します。

```JSON
http://localhost:8000/api/hello
─▶︎ {"message":"Hello"}

http://localhost:8000/api/other
─▶︎ {"error":"Not Found"}
```

> セクション2で「URLで機能を分ける」と説明した内容が、コードとして動いています。

###### URLごとに処理を分けることが、APIの基本的な設計

---

## セクション3のまとめ

| 項目 | 内容 |
|------|------|
| 環境構築 | VSCode・PHPのインストールと確認 |
| サーバー起動 | `php -S` で内蔵サーバーを動かす |
| JSON返却 | `header()` と `json_encode()` でAPIを作る |
| ルーティング | URLによって処理を分岐させる |

今のコードは、アクセスのたびに同じ結果を返すだけです。「連絡先を登録する」操作をしたとしても、データはどこにも保存されません。

次のセクションでは、この問題を解決するために**データベース**が必要な理由を考えます。
