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
cd jwt_auth_api
# データベース作成
rails db:create

# サーバー起動確認
rails s -b 0.0.0.0
# ctrl + c 停止 \
# コンテナ内部から抜ける
exit
```

## ログイン用のデータを作成

### bcrypt のインストール
bcrypt gemをインストールする必要があります。以下のコマンドを実行してください。

```shell
# コンテナに入る
docker exec -it rails8-jwt-api-web-1 bash

# jwt_auth_apiディレクトリに移動
cd jwt_auth_api

# Gemfileに bcrypt を追加
echo "gem 'bcrypt', '~> 3.1.7'" >> Gemfile

# bundle installを実行
bundle install

# サーバーを再起動（バックグラウンドで実行する場合）
rails s -b 0.0.0.0 -d
```

signup
```shell
curl -X POST http://localhost:3000/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'
```

signin
```shell
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

## Gitのコミットするときの注意
どうやら作成されたRailsもGitがあるらしく別々に、GitHubへpushする必要がありそうだ。

https://github.com/sakurakotubaki/jwt_auth_api