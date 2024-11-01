# NginxでHTTPS対応Webサーバーを構築する
## Apacheのアンインストール

- 自動起動をオフに設定
- apacheを停止
- アンインストール
  - 環境設定ごと削除したいのでpurgeコマンドを使用する

```
sudo systemctl disable apache2.service
sudo apachectl stop
sudo apt-get purge apache2 apache2-utils apache2.2-bin apache2-common
sudo apt-get autoremove --purge
```

## Nginx設定

- 同様にNginxでもHTTPSサーバーを設定
- curlコマンドで自己署名に関する警告を無視するコマンド
    ```
    curl -k https://nyuusen.ddns.net/
    ```

# Docker + NginxでHTTPS構築する

- UbuntuServerにDockerをインストールする方法
  - https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
- ざっくり手順としては、以下の通り。
  - Nginxのイメージを取得し、起動する
  - ポートフォワードする
  - 秘密鍵と証明書をコンテナ側にコピーする
  - コンテナ側のサーバー設定ファイルを修正する