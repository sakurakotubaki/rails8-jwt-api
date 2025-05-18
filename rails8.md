# Rails 8 で発生したJWT認証エラーの解決方法

## 発生したエラー

Rails 8のアプリケーションでJWT認証を実装する際に、以下のエラーが発生しました：

```
{"status":500,"error":"Internal Server Error","exception":"#<NoMethodError: undefined method `secrets' for an instance of JwtAuthApi::Application>","traces":{"Application Trace":[{"exception_object_id":21140,"id":0,"trace":"app/services/json_web_token.rb:2:in `<class:JsonWebToken>'"},...
```

具体的には、`JsonWebToken`クラスの実装で、`Rails.application.secrets.secret_key_base`を使用していた部分でエラーが発生しました。

## 原因

このエラーはRails 8で`secrets`メソッドが非推奨になり、完全に削除されたためです。Rails 5.2以降、`secrets`の代わりに`credentials`システムが導入されており、Rails 8では完全に移行が完了しています。

## 解決方法

### 1. JsonWebToken クラスの修正

`app/services/json_web_token.rb`ファイルを以下のように修正しました：

```ruby
# 変更前
class JsonWebToken
  SECRET_KEY = Rails.application.secrets.secret_key_base.to_s

  def self.encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET_KEY)
  end

  def self.decode(token)
    decoded = JWT.decode(token, SECRET_KEY)[0]
    HashWithIndifferentAccess.new decoded
  end
end

# 変更後
class JsonWebToken
  SECRET_KEY = Rails.application.credentials.secret_key_base || Rails.application.config.secret_key_base

  def self.encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET_KEY)
  end

  def self.decode(token)
    decoded = JWT.decode(token, SECRET_KEY)[0]
    HashWithIndifferentAccess.new decoded
  end
end
```

主な変更点：
- `Rails.application.secrets.secret_key_base.to_s` から `Rails.application.credentials.secret_key_base || Rails.application.config.secret_key_base` に変更
- フォールバックとして `config.secret_key_base` も利用可能にしました（開発環境向け）

### 2. JWT gemのインストール

JWT認証に必要なgemがインストールされていなかったため、以下のコマンドを実行してインストールしました：

```shell
docker exec -it rails8-jwt-api-web-1 bash -c "cd jwt_auth_api && echo \"gem 'jwt'\" >> Gemfile && bundle install"
```

### 3. サーバーの再起動

変更を適用するためにRailsサーバーを再起動しました：

```shell
docker exec -it rails8-jwt-api-web-1 bash -c "cd jwt_auth_api && rails restart"
```

## 動作確認

修正後、以下のcurlコマンドでサインアップとログインをテストし、正常に動作することを確認しました：

### サインアップ

```shell
curl -X POST http://localhost:3000/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'
```

### ログイン

```shell
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

レスポンスとして正常にJWTトークンが返されました：

```json
{
  "token":"eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE3NDc2Mzg4MDV9.OFIB9Hi3SqC85Zekk-37gNxAT9GY0IcdXNZWEcD1Brc",
  "exp":"05-19-2025 07:13",
  "email":"user@example.com"
}
```

## まとめ

Rails 8への移行で、`secrets`から`credentials`への変更に伴うエラーが発生しましたが、適切にコードを修正することで解決しました。Rails 5.2以降のプロジェクトでは、秘密情報の管理に`credentials`を使用することがベストプラクティスとなっています。また、JWT認証を実装する際には必ず`jwt` gemをインストールすることも重要です。
