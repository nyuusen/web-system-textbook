# サーバーの証明書の取得

## 事前準備

- EC2(ubuntu)を起動する
- セッションマネージャーでEC2にログインする

```
cd ~
mkdir web-server
cd web-server
openssl version
```

## CSR作成用の秘密鍵の作成

- 下記コマンドを実行すると、パスフレーズを聞かれるので、入力することで秘密鍵の作成が完了する

```
openssl genrsa -des3 2048 > server.key
```

## CSRファイルの作成

- 認証局がサーバー証明書を発行に必要なCertificate Signing Requestファイルを作成する
- このファイルがSSL/TLS証明書を取得するために必要な情報を含んでいる
  - 具体的にはSSL/TLS証明書を証明書発行機関(CA)にリクエストする際に使用される
- 秘密鍵を指定することで、内部的に公開鍵を作成してくれている
- 構成要素
  - 公開鍵
  - サーバーの所有者や組織の情報

```
openssl req -new -key server.key -out server.csr
```

## CSRファイルの確認

- 所有者や公開鍵情報(アルゴリズムや鍵長)が確認できる

```
openssl req -noout -text -in server.csr
```

## 秘密鍵(server.key)の確認

```
$ more server.key

-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFHDBOBgkqhkiG9w0BBQ0wQTApBgkqhkiG9w0BBQwwHAQIuIJ9ByK76jgCAggA
...
-----END ENCRYPTED PRIVATE KEY-----

```

## 秘密鍵の復号

- 暗号化されたままの秘密鍵を使用すると、Webサーバー起動の度にパスフレーズを聞かれたりする等、不都合があるため。

```
openssl rsa -in server.key -out server.key
```

## 復号された秘密鍵の確認

- ENCRYPTEDというワードが消えており、復号できたことが確認できる

```
$ more server.key
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDDdymWr9Wlw1/9
...
-----END PRIVATE KEY-----
```

## CSRの提出とサーバー証明書の取得

- 認証局にCSRを提出すると、ドメイン名の登録者に、サーバー証明書の申請者がドメイン名を使用する権利を持っているかを確認する
  - 一般的にはメール認証（メール内認証用リンクやコードを設置する）
  - 会社の登記情報や第三者機関のデータベース、銀行口座の保有状況で証明する場合がある
- 認証局から指定されたTXTレコードを、ドメインのDNSに追加するよう求める
  - 追加されることで、そのドメインの管理権限を保持しているかを確認できる
- 認証局が提供する認証用ファイルを、そのドメインのWebサーバーの特定のパスにアップロードする
  - 認証局がそのURLにアクセスし、ファイルが正しく配置されていることを確認する

# プライベート認証局でサーバー証明書を自前で発行する

パブリックな認証局は有料且つ時間を要するため、プライベート認証局を使用する

## 自己署名して証明書を作成

- CSRにファイルに対して自己署名する

```
openssl x509 -req -in server.csr -days 365 -signkey server.key -out server.crt
```

(補足)
- 署名の役割は「あるデータ(CSR)に対して、このデータは特定の主体によって作成された」ことを証明すること
- 署名の検証のプロセスは、
  - 秘密鍵(server.key)を使用して、CSR(server.csr)の内容からハッシュ値を生成する
  - このハッシュ値を秘密鍵で暗号化する
  - 証明書を受け取ったクライアントは、証明書に含まれる公開鍵を使用して、署名を検証する
  - 署名を解読して、元のハッシュ値と証明書内容から新たに計算したハッシュ値が一致すれば、証明書の内容が改竄されておらず、指定された公開鍵を持つサーバーによって署名されたことを確認できる

## 証明書の内容を確認

- 暗号化された内容を確認
```
$ cat server.crt
-----BEGIN CERTIFICATE-----
MIID2zCCAsMCFEnir2vffiuz30ytnn4QS+fgAI2lMA0GCSqGSIb3DQEBCwUAMIGp
...
L9ULvW8zl5ri1Md6XJtjx/3StBxuzlrUxlL90S4Mgw==
-----END CERTIFICATE-----
```

- Webサイトの運営者の情報や暗号化アルゴリズム、サーバー公開鍵を確認
```
$ openssl x509 -in server.crt -noout -text
```

# HTTPS対応のWebサーバーを構築する

## Apacheの設定

```
sudo apt -y install apache2
# etc/apache2/sites-available/000-default.conf 更新
sudo mv /var/www/html/index.html /var/www/html/index.html.org
echo "<h1>Test Page</h1>" | sudo tee /var/www/html/index.html
```

## HTTPSの設定

- Apache拡張モジュールを有効化
```
sudo a2enmod ssl
```

- 証明書と秘密鍵を/etc/apache2に配置
  - 証明書: 証明書はクライアントにより検証され、なりすましを防止されていないことを証明する
  - 秘密鍵: クライアントは証明書に含まれた公開鍵を使用して通信するので、サーバ側での復号に秘密鍵が必要となる

```
sudo mv server.crt /etc/apache2/
sudo mv server.key /etc/apache2/
```

- 設定を読み込んで、apachectlを再起動

```
sudo a2ensite default-ssl.conf
sudo apachectl restart
```

- 無事にHTTPSを使ってドメインアクセスできた。
