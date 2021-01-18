# 簡単なFlask APIサーバ

Flaskで作成する簡単なAPIサーバの写経です。

---

## 1. v0.1

- RestlessとSqlAlchemyによる簡易APIサーバ
- CORによるCross Origin Resource Sharing許容
- Vue.jsによる簡易なフロントエンド
  - 簡易的にtestフォルダへ配置

---

## 2. v0.2

- 環境変数、インスタンスフォルダ、from_object、from_pyfileを使った構成管理
- waitressを使ってWSGIコンテナからサービスを起動

---

## 3. v0.3

- Dockerfile、.dockerignore
- コンテナ化

---

## 4. v0.4

- フロントエンドを変更
  - Vue.jsを書き直し
  - 完了、削除機能
  - CSSで見た目を改善

---

## 5. v0.5

- waitressの前にnginxを追加
  - 静的ファイルの配信
  - API要求をwaitressへReverse Proxy
- nginxをコンテナ化
- docker-compose対応
  - flask/waitressコンテナとnginxコンテナを管理
  - nginxのTCP 8080番のみ外部公開
  - waitressのTCP 5000番は外部に公開していない

---

## 6. v0.6

- MySQL対応
  - python main.pyしたときはsqlite3を使う
  - dockerから実行したときはMySQLを使う
- named volumeでMySQLデータを永続化
- MySQL、App、Proxyの順で起動するようにdocker-compose.ymlを修正
  - bashが必要だったのでAppのベースをalpineからslim-busterへ変更
- db.create_all()をmain.pyで実施

---

## 7. v0.7
- waitressがアクセスログをコンソールへ出力
  - PasteのTranslogggerを使用
  - docker bridgeがNATするのでクライアントIPは残念ながらwaitressには見えない

---

## その他

### curlによるAPIの呼び出し

```
curl -H "Content-Type: application/json" -X POST -d "{\"title\": \"buy milk\"}" http://localhost:5000/api/todoitems
curl -H "Content-Type: application/json" -X POST -d "{\"title\": \"play game\"}" http://localhost:5000/api/todoitems
curl -H "Content-Type: application/json" -X POST -d "{\"title\": \"write article\"}" http://localhost:5000/api/todoitems
curl -H "Content-type: application/json" -X GET http://localhost:5000/api/todoitems
```

---

### Vue.jsのフロントエンド

- nginxから配信

```
/usr/share/nginx/html
├── index.html
└── main.js
```

---

### インスフォルダ

インスタンスフォルダは.gitignoreでGit管理対象外にしている

構成

```
instance/
└── config.cfg
```

config.cfgの内容

```
TODOAPI_CONFIG2 = "configured in instance folder"
SQLALCHEMY_TRACK_MODIFICATIONS = True
```

---

### sqlite3 DB初期化手順

```
export FLASK_APP=todoapi
flask shell
from todapi import db
db
db.create_all()
exit()
```

---

### flaskの実行

waitress実行前にflaskのみで動作確認をする

```
TODOAPI_ENV=dev
flask run
```

---

### waitressの実行

```
python main.py
```

もしくは

```
waitress-serve --port=5000 todoapi:app
```

---

### docker imageの作成

```
docker build -t todoapi:latest .
```

---

### コンテナ実行

```
docker run -it --rm -p 5000:5000 todoapi:latest
```

---

### コンテナ接続

```
docker exec -it <container id> /bin/sh; exit
```

---

### docker imageの作成（docker-compose）

```
docker-compose build
```

---

### コンテナ実行（docker-compose）

```
docker-compose up -d
```

---

### コンテナ終了（docker-compose）

```
docker-compose down
```

### コンテナ接続（docker-compose）

```
docker-compose exec <サービス名> <シェル>
例：docker-compose exec todoapi-proxy /bin/bash
```

「-it」してなくても接続できる

---

### 諦めたこと

#####  SSL/TLS対応
- SSL/TLS対応を行ったが証明書の管理が煩雑なのでcommitしなかった

##### waitressでクライアントの実IPをログ出力
- docker bridgeが要求をNATする。
- NATするときdockerはX-real−ipやX-forwarded-forにクライアントの実IPは入れない。
- このようにクライントの実IP情報がDocker Networkの入り口で失われてしまう。
- 「ubuntuを使うとクライント実IPを取得できる」というコメントもあった。
- 対策
  - docker bridgeの外にLoadbalancerを配置してX-forwarded-forに追加させる
  - host network（NATなし）を使う
  - ubuntuを使う
- host networkが推奨されているようだがhost networkは使いたくないのでやめた
