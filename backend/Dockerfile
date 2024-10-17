# ビルド用環境
# ----------------------------------------------

# Golang 1.19.2 と Debian Bullseye ベースのイメージを使用して、ビルド環境を構築
FROM golang:1.19.2-bullseye AS deploy-builder 

# 作業ディレクトリを /app に指定
WORKDIR /app

# Go モジュールの依存関係を定義したファイル（go.mod、go.sum）を作業ディレクトリにコピー
COPY go.mod go.sum ./

# Go の依存パッケージをダウンロード
RUN go mod download

# アプリケーション全体のソースコードをコンテナにコピー
COPY . .

# Go アプリをビルド、
# -trimpath でソースパスを除去、
# -ldflags "-w -s"でデバッグ情報を削除し、
# "app"という実行可能ファイルを出力
RUN go build -trimpath -ldflags "-w -s" -o app

# 本番環境
# ----------------------------------------------
# 本番用の軽量な Debian Bullseye ベースのイメージを使用
FROM debian:bullseye-slim AS deploy

# X509: Certificate Signed by Unknown Authorityエラーを回避する
# パッケージリストを更新
# SSL証明書とHTTP通信のための必要パッケージをインストール
# キャッシュを削除してディスクスペースを節約
# 不要なパッケージを削除する
# キャッシュリストを削除してイメージサイズを削減
RUN apt-get update \ 
    && apt-get install -y --force-yes --no-install-recommends apt-transport-https curl ca-certificates \
    && apt-get clean \
    && apt-get autoremove \
    && rm -rf /var/lib/apt/lists/*

# ビルドした実行ファイルを本番環境にコピー
COPY --from=deploy-builder /app/app .

# アプリケーションを起動
CMD ["./app"]


# 開発環境
# ----------------------------------------------
# 開発用に軽量な Alpine Linux ベースの Golang イメージを使用
# バージョン変わった
FROM golang:1.23-alpine AS dev

# 作業ディレクトリを /app に設定
WORKDIR /app

# 必要なパッケージ（Alpine SDKとJSONパーサーのjq）をインストール
RUN apk update && apk add alpine-sdk && apk add jq

# コードのホットリロードツール "air" をインストール
# バージョン変わった！
RUN go install github.com/air-verse/air@latest 

# MySQL のスキーマ定義ツール "mysqldef" をインストール
# バージョン変わった！
RUN go install github.com/sqldef/sqldef/cmd/mysqldef@latest

CMD ["air"]