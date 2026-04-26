# セクション3：PHPで最小のバックエンドを作る

## このセクションの目的

セクション2では、HTTP・リクエスト/レスポンス・JSON・APIという概念を学びました。  
このセクションでは、それらの概念を **PHPのコードとして動かす** 体験をします。

環境構築から始め、最終的には「URLにアクセスするとJSONが返ってくるAPI」を自分で作ります。  
「バックエンドが実際に動く」という体験を得ることが、このセクションのゴールです。

---

## 開発環境の全体像

このセクションで使うものは3つです。

| ツール | 役割 |
|--------|------|
| VSCode | コードを書くエディタ |
| PHP | バックエンドを動かすプログラミング言語 |
| PHP内蔵サーバー | ローカルでHTTPサーバーを動かす仕組み |

通常のWeb開発では、ApacheやNginxといったサーバーソフトウェアを別途インストールする必要があります。  
PHPには開発用の内蔵サーバーが用意されており、コマンド1行で起動できます。  
この講座では、環境構築のハードルを下げるためにこの内蔵サーバーを使います。

---

## VSCodeの準備

### インストール

VSCode（Visual Studio Code）は、Microsoftが提供する無料のコードエディタです。  
公式サイトからインストーラーをダウンロードして実行します。

### フォルダを開く

VSCodeでは「フォルダを開く」ことがプロジェクトの起点になります。  
作業用のフォルダを作成し、VSCodeのメニューから「フォルダを開く」で開きます。

### ターミナルを使う

VSCodeにはターミナルが内蔵されています。  
メニューの「ターミナル」→「新しいターミナル」で開けます。  
コマンドの実行はこのターミナルから行います。

### 推奨拡張機能

PHPのコードを書くときは、**PHP Intelephense** という拡張機能を入れておくと、コード補完が効いて書きやすくなります。  
ただし、なくても動作には問題ありません。

---

## PHPのインストール

### インストール方法

**macOS の場合：**

Homebrew（パッケージマネージャー）を使ってインストールします。

まず、Homebrewをインストールします。（<https://brew.sh/ja/）>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

続いて、PHPをインストールします。

```bash
brew install php
```

**Windows の場合：**

Scoop（パッケージマネージャー）または公式サイトのバイナリを使います。

まず、Scoopをインストールします。

PowerShell（バージョン5.1以降）を開き、`PS C:\>` プロンプトから以下を実行します。

```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

続いて、PHPをインストールします。

```bash
scoop install php
```

### インストールの確認

以下のコマンドを実行して、バージョンが表示されればインストール成功です。

```bash
php -v
```

出力例：

```text
PHP 8.x.x (cli)
```

このセクションのゴールは「PHPが動くこと」です。バージョンの数字にこだわる必要はありません。

> `php -v` でエラーになる場合は、ターミナルを一度閉じて開き直してください。
> それでも動かない場合は、PHPのインストール先にPATHが通っていない可能性があります。

---

## プロジェクト構成を作る

### フォルダ構成

この講座では、以下のフォルダ構成を使います。

```text
project/
├── api/
│   └── index.php
└── public/
```

| フォルダ/ファイル | 役割 |
|-----------------|------|
| `api/` | バックエンドのPHPファイルを置く場所 |
| `api/index.php` | 最初に作るAPIファイル |
| `public/` | フロントエンドのHTMLファイルを置く場所（後のセクションで使用） |

### なぜこの構成か

`api/` と `public/` を分けるのは、**役割を明確に分離するため**です。  
バックエンドのコードとフロントエンドのコードが混在すると、どこに何があるかが分かりにくくなります。  
最初からフォルダで分けておくことで、コードの見通しが良くなります。

---

## 最初のPHPコードを書く

### index.phpを作成する

`api/index.php` を作成し、以下のコードを書きます。

```php
<?php
echo "Hello, Backend!";
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `<?php` | ここからPHPのコードが始まるという宣言 |
| `echo` | 文字列を出力する命令 |

このコードはまだAPIではありません。  
ただ文字を出力するだけのプログラムですが、PHPが動くことを確認するための第一歩です。

---

## PHP内蔵サーバーを起動する

### サーバーの起動

ターミナルで以下のコマンドを実行します。
このコマンドは、`api/` と `public/` が入っている `project/` フォルダで実行します。

```bash
php -S localhost:8000
```

### コマンドの意味

| 部分 | 意味 |
|------|------|
| `php -S` | PHP内蔵サーバーを起動するオプション |
| `localhost` | 自分のコンピューター上で動かすことを示す |
| `8000` | 使用するポート番号（接続先の「部屋番号」のようなもの） |

### ブラウザで確認する

ブラウザのアドレスバーに以下のURLを入力します。

```text
http://localhost:8000/api/index.php
```

`Hello, Backend!` と表示されれば成功です。

> **体験のポイント：**  
> このとき、ブラウザ（クライアント）がPHPサーバー（バックエンド）にリクエストを送り、  
> サーバーが `Hello, Backend!` というレスポンスを返しています。  
> セクション2で学んだ「リクエスト/レスポンス」が、実際に動いている瞬間です。

### サーバーの終了

PHP内蔵サーバーを終了するには、サーバーを起動中のターミナルで`ctrl+c`を実行します。

---

## JSONを返すAPIにする

### コードを変更する

`api/index.php` の内容を以下に書き換えます。

```php
<?php
header("Content-Type: application/json");

echo json_encode([
  "message" => "Hello, API!"
]);
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `header("Content-Type: application/json")` | レスポンスの形式がJSONであることをブラウザに伝える |
| `json_encode(...)` | PHPの配列をJSON文字列に変換する |

### ブラウザで確認する

同じURLにアクセスすると、今度は以下のようなJSONが返ってきます。

```json
{"message":"Hello, API!"}
```

> **ここが「API」です。**  
> HTMLではなくJSONを返すようになりました。  
> セクション2で「なぜHTMLではなくJSONなのか」を学びましたが、その理由がここで実感できます。

---

## curlでAPIを確認する

### curlとは

`curl` は、ターミナルからHTTPリクエストを送るコマンドです。  
ブラウザを使わずにAPIの動作を確認できます。

### 実行する

macOS の場合：

```bash
curl http://localhost:8000/api/index.php
```

Windows の場合： `curl` コマンドに `.exe` を付けます。

```bash
curl.exe http://localhost:8000/api/index.php
```

ターミナルに以下のようなレスポンスが表示されます。

```json
{"message":"Hello, API!"}
```

ブラウザとcurlの両方でAPIを確認できることで、「APIはURLにアクセスすれば使えるもの」という理解が深まります。

---

## 簡単なエンドポイントを作る

### URLによって処理を変える

ここまでのコードは、`/api/`以下にどのようなURLを入力しても同じ結果を返します。  
実際のAPIでは、URLごとに異なる処理を返します。

```php
<?php
header("Content-Type: application/json");

$path = $_SERVER["REQUEST_URI"];

if ($path === "/api/hello") {
  echo json_encode(["message" => "Hello"]);
} else {
  http_response_code(404);
  echo json_encode(["error" => "Not Found"]);
}
```

### コードの意味

| 部分 | 意味 |
|------|------|
| `$_SERVER["REQUEST_URI"]` | アクセスされたURLのパス部分を取得する |
| `if ($path === "/api/hello")` | パスが `/api/hello` のときだけ処理を分岐する |

### 確認する

以下のURLにアクセスして、それぞれ異なるレスポンスが返ることを確認します。

macOS の場合：

```bash
curl http://localhost:8000/api/hello
# → {"message":"Hello"}

curl http://localhost:8000/api/other
# → {"error":"Not Found"}
```

Windows の場合：

```bash
curl.exe http://localhost:8000/api/hello
# → {"message":"Hello"}

curl.exe http://localhost:8000/api/other
# → {"error":"Not Found"}
```

> **ポイント：**  
> URLによって処理を分けることが、APIの基本的な設計です。  
> セクション2で「URLで機能を分ける」と説明した内容が、コードとして動いています。

---

## セクション3のまとめ

このセクションでできるようになったこと：

| 項目 | 内容 |
|------|------|
| 環境構築 | VSCode・PHPのインストールと確認 |
| サーバー起動 | `php -S` で内蔵サーバーを動かす |
| JSON返却 | `header()` と `json_encode()` でAPIを作る |
| ルーティング | URLによって処理を分岐させる |

### 次の課題

現時点のAPIには、大きな問題があります。

> **データが保存されない**

今のコードは、アクセスのたびに同じ結果を返すだけです。  
「連絡先を登録する」という操作をしても、データはどこにも保存されていません。  
ページをリロードすれば、何もなかった状態に戻ります。

次のセクションでは、この問題を解決するために**データベース**が必要な理由を考えます。
