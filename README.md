# Rails8 JWT Auth API
環境構築用ファイルを作成する。

1. database.yml
2. docker-compose.yml
3. Dockerfile
4. Dockerfile.dev
5. Gemfile

```shell
docker-compose up -d
```

コンテナに入る
```shell
# コンテナ名を調べる
docker ps
# コンテナ名を指定して内部に入る
docker exec -it rails8-jwt-api-web-1 bash
```

[Rails入門](https://guides.rubyonrails.org/getting_started.html#adding-authentication)

コンテナ内部で作成する。
```shell
# コンテナ内で
bundle install
# create project
rails new jwt_auth_api --api --force --database=postgresql

# myappへ移動して実行する。
cd store
# データベース作成
rails db:create

# サーバー起動確認
rails s -b 0.0.0.0
# ctrl + c 停止 \
# コンテナ内部から抜ける
exit
```