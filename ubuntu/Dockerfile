# ベースイメージとしてUbuntuのlatestを使用
FROM ubuntu:latest

# aptのパッケージリストを更新し、telnetをインストール
RUN apt-get update && apt-get install -y telnet

# コンテナが起動したらシェルを使用できるようにする
CMD ["bash"]
