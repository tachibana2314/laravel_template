# Laravel Template
It's a template for develop project "rapidly" under the environment Laravel / Docker

##MakeコマンドについてはMakefileを参照

## TIPS
### mailhogについて
メール送信機能のテスト用のSMTPサーバーとしてsmtpコンテナを用意してあります。<br>
これはmailhogというテスト用のSMTPサーバーソフトウェアで、.envにはデフォルトでこのコンテナへの接続情報を記してあります。<br>
使用するにあたって、特に設定は必要ありません。（.envに設定済みのため）<br>

アプリケーションから送信されたメールの確認はブラウザで以下のURLに接続することで確認できます。<br>
```
http://localhost:8025
```

### 各コンテナのログを確認したい
- web
    - ./docker/web/nginx/volumes/log ディレクトリ内に格納されている<br>

|file name|description|
|:---|:---|
| access.log | アクセスログ |
| error.log | エラーログ |

- db
    - ./docker/db/mysql/volumes/log ディレクトリ内に格納されている<br>
    
|file name|description|
|:---|:---|
| error.log | エラーログ |
| slowquery.log | 実行に1秒以上時間を要したクエリが記録される |
| mysql-general.log | 実行したクエリが記録される |

### 本番環境用にhttps化したい
### 認証局、鍵の準備
mkcert, nssのインストール
https://github.com/FiloSottile/mkcert
ローカルで信頼できる開発証明書を作成するためのツール

https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS

Network Security Services
FirefoxでlocalhostのSSLを有効化するために必要
ChromeのSSL警告をlocalhostの時だけ表示しないようにする。
秘密鍵、公開鍵の作成
コマンドの実行は docker-laravel ディレクトリルートで行ってください。
```
$ mkcert -cert-file ./infra/docker/nginx/localhost.pem -key-file ./infra/docker/nginx/localhost-key.pem localhost
```

Dockerのコード書き換え
docker-compose.yml
公開ポートを80から443へ変更する。
HTTPSは、ウェルノウンポート番号として443が使われるのが一般的です。
ローカルでは他のコンテナと被らない番号であれば何でも良いです。

https://ja.wikipedia.org/wiki/HTTPS
```
#docker-compose.yml
services:
  web:
    ports:
      - 443:443

```
下記のコードを追記する。
```
#docker/nginx/default.conf
server {
    # listen 80;
    listen 443 ssl;
    server_name localhost;
    ssl_certificate /etc/nginx/conf.d/localhost.pem; # サーバ証明書
    ssl_certificate_key /etc/nginx/conf.d/localhost-key.pem; # 秘密鍵

    # ... 省略
}
```
#docker/nginx/Dockerfile
下記のコードを追記する。

```
#docker/nginx/Dockerfile
COPY ./*.pem /etc/nginx/conf.d/
```
秘密鍵、公開鍵をコピーしてます。
Dockerイメージの作成、コンテナ再生成
コマンドの実行は docker-laravel ディレクトリルートで行ってください。
```
$ docker-compose build web
$ make restart
```
```
chrome://flags/#allow-insecure-localhost アドレスバーに入力する。
```
$ mkcert -install
```


## 環境構成
|コンテナ名|環境|ベースイメージ|  
|:---|:---|:---|
|web|nginx|nginx:1.19.2 - alpine|
|db|MySQL 5.7|mysql:5.7|
|app|PHP 7.4 & Laravel 6.2 LTS|php:7.4 - fpm|
|smtp|mailhog|alpine:3.4|


## コンテナ構造
```
client --> web <--> app <--> db
  |                  |
  └------> smtp <----┘        
```
