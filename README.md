# Laravelをローカル環境で動かすテンプレート

LaravelをDockerで動かすためのテンプレートです。
Laravelをお試しで動かしてみたい、というときに使ってみてください。

(免責)このテンプレートを利用されることで生じたいかなる問題に関して、作成者であるこのリポジトリの所有者は責任を負いません。

## 構成

### Dockerのコンテナグループ

**phpコンテナ**
- Apache, PHP, composer, Node.js がインストールされている。Laravelが動作している。

**dbコンテナ**
- MySQLのDBサーバ。

**phpmyadminコンテナ**
- dbコンテナ専用のDBクライアント。コメントアウトされているので、使いたい場合はコメント解除して有効にすること。

**Node.jsの動作環境**
- プロジェクトによって、Node.jsが必要になるかもしれない。これは、毎回使い捨てのコンテナで対応する。


| System         | Version                             | Container    |
| -------------- | ----------------------------------- | ------------ |
| PHP            | 8.3.1                               | php          |
| MySQL          | 8.0                                 | db           |
| Apache         | 2.4                                 | php          |
| Debian (Linux) | 12 (bookworm)                       | php          |
| Node.js        | イメージプル時のLTSバージョン             | node:lts-slim |
| Xdebug         | コンテナビルド時の最新バージョン            | php          |
| DBクライアント    | 自由に選ぶこと(phpMyAdminだけ用意している) | -             |


## 「初回」環境構築

### 前提条件
以下のアプリをインストールしておくこと。
- Docker
- VSCode(任意。デバッガを使いたい場合は必要)

また、phpMyAdminを用意されたコンテナで利用したい場合は、先にdocker-compose.ymlのコメントアウト部分を解除しておく。

以下の項目について理解できている人を対象としている。
- Docker (仮想マシン, コンテナ, docker-compose, Linuxなど)
- PHP/Laravel
- データベース/MySQL


### 作業手順
docker-compose.ymlがあるディレクトリで、以下のコマンドを順に実行する。  
(もし、プロジェクト名を名付けたかったら、全てのdocker composeコマンドの"compose"の直後に"-p プロジェクト名"とつけること。  
プロジェクト名をつけるメリットは、同じ構成ファイルで複数のコンテナグループを作れることと、コマンド実行のディレクトリがどこでもOKになること。  
デメリットは、コマンドに"-p プロジェクト名"を毎回つけなくてはいけなくなること)
```sh
docker compose up -d --build
```
```sh
docker compose exec php bash -c "cd / && composer create-project --prefer-dist laravel/laravel /laravel-tmp"
```
```sh
docker compose exec php bash -c "rsync -av --ignore-existing /laravel-tmp/ /var/www/html/src/ && rsync -av /laravel-tmp/storage/framework /var/www/html/src/storage/framework && rsync -av /laravel-tmp/vendor /var/www/html/src/vendor && rm -R /laravel-tmp"
```
```sh
docker compose exec php bash -c "chmod 707 -R storage"
```
```sh
docker compose cp php:/var/www/html/src/vendor ./src
```

Laravelがデータベースに接続するためのDBホスト/ユーザ名/パスワード等を、src/.envに入力する。
.envの中の、以下の項目をそれぞれ、この通りに書き換えること。
```ini
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=user
DB_PASSWORD=password
```

以上で環境構築は終了。解説は下の方で。

## 開発で使う機能

### 1. PHPデバッグ
- PHPのデバッガであるXdebugを、VSCodeから実行することができる。
- デバッガでは、phpファイルの任意の行にブレークポイントを設定することで、そのブレークポイントの時点で処理を一時停止することができる。
- 一時停止した後は、処理をステップごとに1つずつ実行したり、停止した処理の時点での変数の値の確認、コールスタックの確認などができる。

**使い方**
- VSCodeに、「PHP Debug/Xdebug」という拡張機能をインストールする
- VSCodeの左のタブから「実行とデバッグ」タブを選ぶと、左上に「▶ PHPデバッグ」と欄が出るので、それを押下。
- VSCodeの画面のどこかに、ポーズ, リロード, 停止などを表すアイコンが横並びになっている小さいパネルが表示される。
- この状態が、デバッガが起動して、ブレークポイントまで処理が到達するのを待機している状態。デバッガを終了するときは、オレンジの「停止」アイコンを押下する。
- デバッガが起動した状態で、ブレークポイントをどこかに置き、ブラウザからアクセスするなどして対象のファイルが実行されるようにすると、ブレークポイントの時点で処理が一時停止する。

### 2. phpMyAdmin
- 開発時に、データベースのテーブルを確認したりデータを直に投入するためのアプリ(DBクライアント)として、phpMyAdminを使うことができる。
(個人的には、A5SQLやDBeaverの方が多機能で使いやすいと思うが、インストールとか設定とかが面倒であれば、これを使うのもアリ)

**使い方**
- まず、「環境構築」の手順を行う前に、docker-compose.ymlの"phpmyadmin"コンテナの記述部分のコメントを解除しておくこと。
- コンテナが起動している状態で、以下のURLからphpMyAdminにアクセスできる。  
  [http://localhost:3001/index.php?route=/database/structure&db=laravel](http://localhost:3001/index.php?route=/database/structure&db=laravel)

### 3. VSCodeの拡張機能
以下の拡張機能を入れておくとよい(<パッケージ名>/<作者>で表記している)。
- PHP Intelephense/Ben Mewburn
- PHP Namespace Resolver
- Laravel Blade Snipets/Winnie Lin
- PHP Extension Pack/Xdebug

## 開発で使うコマンド

プロジェクト名をつけていない場合、全てのdocker compose から始まるコマンドは、docker-compose.ymlがあるディレクトリで実行する必要がある。プロジェクト名をつけている場合、docker compose コマンドの"compose"の直後に"-p プロジェクト名"とつけて読み替えること。

---

### コンテナグループを起動するコマンド

```sh
docker compose start
```
---

### コンテナグループを停止するコマンド(PCの再起動・シャットダウン時や作業終了時に実行すること)

```sh
docker compose stop
```
---

### phpコンテナのシェルを呼び出すコマンド

```sh
docker compose exec php bash
```
- phpコマンド、composerコマンドを実行したいときは、上のphpコンテナのシェルを呼び出して、そのコンテナで実行すること(php artisanなど)。
- phpコンテナのシェルでは、コマンド入力カーソルの左側が「root@xxxxxxxx:/var/www/html/src#」という表示になる。
---

### ./src/vendorフォルダを、現在のphpコンテナのvendorと同期するコマンド

```sh
docker compose cp php:/var/www/html/src/vendor ./src
```
- このコマンドと同じ効果のあるVSCodeタスクを作成しているので、「タスクの実行」から「./src/vendor の同期」を実行することでもvendorを同期できる。
- **注意** - タスクは`.vscode/tasks.json`に記載しているが、プロジェクト名をつけてコンテナグループを作成した場合は、このtasks.jsonのファイルも編集して、実行コマンドに"-p プロジェクト名"とつけておくこと。
- 使いどころは、以下の「解説」で説明している。
---

### Laravel Viteの npmパッケージをインストールしてビルドするコマンド(srcフォルダがあるディレクトリで実行すること)

```sh
docker run -it --rm -v "$(pwd)/src:/src" -v "/src/storage/framework" -v "/src/vendor" -v "node_modules:/src/node_modules" node:lts-slim bash -c "cd /src && npm i && npm run build"
```
- このコマンドと同じ効果のあるVSCodeタスクを作成しているので、「タスクの実行」から「Laravel Vite のビルド」を実行することでもビルドできる。
---

### Laravel Viteの開発用サーバを起動するコマンド(srcフォルダがあるディレクトリで実行すること)

**前提条件**
- src/vite.config.jsに、以下のように追記しておくこと。
```js
// ...省略...
export default defineConfig({

    // 追記ここから
    server: {
        host: '0.0.0.0',
        hmr: {
            host: 'localhost'
        }
    },
    // 追記ここまで

plugins: [
      // ...省略...
    ],
});

```

そのうえで、srcがあるディレクトリで以下のコマンドを実行すると、Vite開発用ローカルサーバが起動する。
```sh
docker run -it --rm -p 5173:5173 -v "$(pwd)/src:/src" -v "/src/storage/framework" -v "/src/vendor" -v "node_modules:/src/node_modules" node:lts-slim bash -c "cd /src && npm i && npm run dev"
```
- このコマンドと同じ効果のあるVSCodeタスクを作成しているので、「タスクの実行」から「Laravel Vite の開発用サーバを起動」を実行することでも起動できる。


## 2回目以降の環境構築について

あなたがこのテンプレートを使って開発環境を作り、このテンプレートごとソースコードをGit等で共有して別のコンピュータなどで再び環境構築する、という場合は、「初回環境構築」のときとは違う手順になる。

なぜなら、すでにLaravelがインストールされているからだ。
Laravelインストール時には.envなどのGitで追跡しないファイルも自動で生成してくれているが、同じ環境を再現する場合は、それを手動で作らなくちゃいけない。

### 手順

1. src/.envを作成する。.envの内容はそれぞれ任意のものにすること。
2. docker-compose.ymlがあるディレクトリで、以下のコマンドを順に実行する
```sh
docker compose up -d --build
```
```sh
docker compose exec php composer install
```
```sh
docker compose cp "$(pwd)/src/storage/framework" "php:/var/www/html/src/storage"
```
  (phpコンテナの`src/storage/framework`はホストOSをマウントしていないので、サブディレクトリの構造をphpコンテナに伝える必要があるため初回のみこのように/storage/frameworkフォルダをコピーする)
```sh
docker compose exec php bash -c "chmod -R 707 storage && php artisan key:generate && php artisan migrate --seed"
```

以上。


## おまけ -PHPのみの動作環境を作る-

PHPの実行環境を手軽に作る手順は以下の通り。

### 【準備編】
docker-compose.ymlがあるディレクトリで、以下のコマンドを実行する。
```sh
docker build ./docker-compose/php -t php-local-image
```

以上で、実行環境の準備は完了。いつでも、手軽に使い捨てのPHP実行環境を起動できる。


### 【実行編】
使い捨てのPHPサーバを起動するときは、ドキュメントルートにしたいディレクトリで以下のコマンドを実行する。
```sh
docker run -it --rm -v "$(pwd):/var/www/html/src/public" php-local-image
```

また、PHPデバッガが起動したいときは、ドキュメントルートにしたいディレクトリに対して.vscodeフォルダをコピーして設置しておく。

### 【このPHP環境について解説】

この2つのコマンドはそれぞれ何をしているかを解説する。

```sh
docker build ./docker-compose/php -t php-local-image
```
このコマンドは、あなたのPCのストレージに「php-local-image」という名前のDockerイメージを作成して保存している。  
このイメージの構成は、`./docker-compose/php`ディレクトリのDockerfileを参照している(docker-compose.ymlでいうphpコンテナ)。  
このDockerfileの構成では、PHPとComposer等がインストールされているApacheサーバが起動する設定になっている。  
このようにイメージを保存しておくことで、いつでもこの構成の新規コンテナを作ることができる。

---

```sh
docker run -it --rm -v "$(pwd):/var/www/html/src/public" php-local-image
```
このコマンドでは、指定したディレクトリをマウントしてphp-local-imageをもとにしたコンテナを新規作成して実行している。  
このコマンドを実行することで、PHPサーバが起動する。起動中は実行ログがコンソールに表示されていて、`Ctrl + C`を押せばコンテナごと終了する。

## 解説

### 環境構築手順の解説

Laravelのソースコードは、VSCodeなどのホストOSのアプリで編集したい。  
そのため、Dockerではソースコードのディレクトリ(srcフォルダ)をホストOSと仮想マシン(phpコンテナ)とで共有している。  
これをマウントという。  
だが、マウントはパフォーマンス低下の原因にもなる。ホストOS側/仮想マシン側のそれぞれでファイルの読み書きがあるたびに、それをもう一方に伝える処理が走るためだ。  
特に、ライブラリのソースコードやキャッシュファイルはファイル数,容量が多かったり、頻繁に参照されているため、Docker仮想環境でのパフォーマンス低下の最大の原因になっている。  
そのため、ライブラリのソースコードフォルダである「/src/vendor」と「/src/node_modules」、キャッシュファイル置き場である「/src/storage/framework」の3つを、ホストOSとのマウントの対象外としている(別途Volumeに向けて再マウントしている)  

しかし、vendorフォルダの中身がホストOSに存在しないと、VSCodeのコード補完、クラス補完が効かなくなるので、手動のタイミングで同期(phpコンテナのvendorフォルダをコピー)する。  
VSCodeのコード補完がいらなければ、この作業はやらなくていい。

#### 環境構築コマンドの説明

```sh
docker compose up -d --build
```
このコマンドは、単にdocker-compose.ymlの構成通りにコンテナグループを作成するコマンド。

---

```sh
docker compose exec php bash -c "cd / && composer create-project --prefer-dist laravel/laravel /laravel-tmp"
```
このコマンドは、phpコンテナに対して、Laravelをインストールするcomposerコマンドを実行している。  
ただし、いったん/laravel-tmpという一時フォルダにインストールしている。  
なぜなら、本当のインストール先である`/var/www/html/src`フォルダには、別途マウントしたvendorフォルダがあり、
Laravelはカレントディレクトリに/vendorなどのフォルダがある場合はインストールが失敗してしまうから。  
次のコマンドで、一時フォルダ/laravel-tmpにインストールしたLaravelを`/var/www/html/src`に引っ越す。

---

```sh
docker compose exec php bash -c "rsync -av --ignore-existing /laravel-tmp/ /var/www/html/src/ && rsync -av /laravel-tmp/storage/framework /var/www/html/src/storage/framework && rsync -av /laravel-tmp/vendor /var/www/html/src/vendor && rm -R /laravel-tmp"
```
このコマンドでは、/laravel-tmp内のすべてのファイルを`/var/www/html/src`に移動している。移動先に同名ディレクトリがある場合は、そのディレクトリだけ除外してコピーしている。  
そして、マウント対象である`src/storage/framework`と`src/vendor`はすでに空フォルダが作成されてしまっているので、ここに対し個別でファイルを引っ越している。  
最後に、一時フォルダであった/laravel-tmpを完全に削除している。

---

```sh
docker cpmpose exec php bash -c "chmod 707 -R storage"
```
phpコンテナのプログラムに対して、storageフォルダのファイル操作権限を与えている。  
storageフォルダに対して権限を与えているのは、画像の保存やログファイルの書き込み等のファイル操作をPHP/Laravelのプログラムが行うため。  

---

```sh
docker compose cp php:/var/www/html/src/vendor ./src
```
phpコンテナの`/var/www/html/src`フォルダにLaravelがインストールされた。そして、このフォルダは、ホストOSの./srcフォルダにマウントされている。  
しかし、上でも書いたように、vendorフォルダはその例外であるので、ホストOSからはvendorフォルダの中身は確認できない。  
このままでもアプリは動くし、vendorフォルダ内のファイルを編集することはないので全然いいのだが、vendorフォルダがホストOSにないことは、以下の2つの欠点がある。  
1. VSCodeの拡張機能「PHP Intelephense」のクラス・コード補完が効かない
2. 不具合などの調査で、vendorフォルダの確認がしたいときに、vendorフォルダが空なためできない
3. Xdebug??

上記の問題を解決するため、手動で`/var/www/html/src/vendor`を./src/vendorにコピーする。そのために、docker compose cpコマンドを使う。  
ただしこれだと、コピーした後にphpコンテナのvendorに対して変更があっても、ホストOSには反映されず、不整合な状態になってしまう。  
vendorを手動で編集することはないので基本的にはそれは起こらないが、composer requireなどで新しいパッケージをインストールしたりすると、この不整合な状態が起きる。  
そのため、新しいパッケージをインストールしたり、アップグレードしたり、パッケージを削除したときなどは、このコマンドを使ってホストOSとphpコンテナのvendorフォルダを手動で同期しよう。

---

```sh
docker run -it --rm -v "$(pwd)/src:/src" -v "/src/storage/framework" -v "/src/vendor" -v "node_modules:/src/node_modules" node:lts-slim bash -c "cd /src && npm i && npm run build"
```
Laravelでは、Viteを使ってJavaScriptライブラリのプログラムを使うことができる。JavaScriptライブラリは、Node.jsで動作するnpmなどを使ってバージョン管理・インストールしたり、ソースコードをビルドしたりするため、Node.js/npmのプログラムが動作する環境が必要。  
そのため今回は、**Node.jsが必要な時だけ**別途nodeコンテナを起動して、処理が終わったら終了するという形をとっている。

例えばcssに関数・変数・インポートなどプログラマブルな概念を搭載できるSCSSは、最終的にcssファイルに変換(ビルド)する必要があるのだが、それをNode.jsにやらすというやり方が主流である。

このコマンドでは、docker-compose.ymlで構築したコンテナグループとは無関係な、node:lts-slimというdockerイメージのコンテナを起動している。その時のオプションは以下の通り。
- `--rm` ... コマンド終了時に、コンテナを終了(削除)する
- `-v "$(pwd)/src:/src"` ... ./srcフォルダをマウントする
- `-v "/src/storage/framework"` ... /src/storage/frameworkは、ホストOSをマウントせず一時Volumeをマウントする
- `-v "/src/vendor"` ... /src/vendorは、ホストOSをマウントせず一時Volumeをマウントする
- `-v "node_modules:/src/node_modules"` ... /src/node_modulesは、ホストOSをマウントせず"node_modules"という名前付きVolumeをマウントする
- `bash -c "cd /src && npm i && npm run build"` ... nodeコンテナで実行するプログラムを指定。ここでは、bashを実行して、その引数(bashで実行したいコマンド)で「/srcフォルダに移動し、npm iとnpm run buildを実行する」というコマンドを指定している

---

```sh
docker run -it --rm -p 5173:5173 -v "$(pwd)/src:/src" -v "/src/storage/framework" -v "/src/vendor" -v "node_modules:/src/node_modules" node:lts-slim bash -c "cd /src && npm i && npm run dev"
```
Laravel Viteで開発するとき、JavaScriptなどのソースコードを変更するたびにビルドするとなると、十数秒の待ち時間が毎度発生するためDXが悪い。  
そのため、Viteの**ホットリロード**機能を使うのがよい。  
Node.jsで、Viteの開発用ローカルサーバを5173番ポートで起動し続ける。  
この開発用サーバを起動している間は、LaravelはViteのビルド済みコードを読み込むのではなく5173番ポートのlocalhostにアクセスしてファイルを取得するようになる。  
このローカルサーバは、ソースコードの変更を検知して、即座に反映する機能を持っているため、効率の良い開発ができるようになる。

このコマンドを実行することで、Viteのローカルサーバが起動する。オプションはビルド時のものとだいたい一緒だが、5173番ポートを開放するオプションを追記している。

また、前提条件として、以下のようにsrc/vite.config.jsを修正する必要があった。
```js
// ...省略...
export default defineConfig({

    // 追記ここから
    server: {
        host: '0.0.0.0',
        hmr: {
            host: 'localhost'
        }
    },
    // 追記ここまで

plugins: [
      // ...省略...
    ],
});

```

追記部分には、開発環境時に、どのホストからのリクエストを受け入れるかなどの設定を記述しており、この記述により「どんなホストからのリクエストも受け付けるぜ！」といった設定になる。phpコンテナとnodeのコンテナは別のOS、別のIPという扱いになっているので、このようにホストの設定をガバガバにしてあげる必要があるらしい。
