---
title: "Docker Composeを触ってみる"
date: 2022-12-07T08:09:14Z
draft: false
weight: 1
---

Docker Composeの概要について説明をしましたので、早速Docker Composeを使って行きましょう。

Docker Composeを試す構成ですが、[公式ドキュメントの「Try Docker Compose」](https://docs.docker.com/compose/gettingstarted/)が説明にちょうど良いので、こちらのアプリケーションを利用して試していきます。

{{< toc >}}

## Docker Composeで実行するためのWebアプリケーション

まずはじめにDocker Composeを動かすためのサンプルアプリケーションを作成していきます。
まず下記コマンドを実行してDocker Composeを試すためのディレクトリを作成します。

```Shell
$ mkdir composetest
$ cd composetest
```

このディレクトリの中で `app.py` というファイルを下記の内容で作成します。

```Python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

更に `requirements.txt` というファイルを下記の内容で作成します。

```Text
flask
redis
```

このアプリケーションのコードや作成手順は下記リンクから引用しています。

https://docs.docker.com/compose/gettingstarted/#step-1-define-the-application-dependencies

## アプリケーション用Dockerfileの作成

次にアプリケーション用のDockerfileを作成していきます。
下記の内容を `Dockerfile` として保存します。

```Dockerfile
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

この `Dockerfile` の内容は下記リンクから引用しています。

https://docs.docker.com/compose/gettingstarted/#step-2-create-a-dockerfile

## docker-compose.ymlを作成する

次に下記の内容を `docker-compose.yml` として保存します。

```YAML
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_DEBUG: True
  redis:
    image: "redis:alpine"
    volumes:
    - redis_data:/data
volumes:
  redis_data:
```

## Docker Composeを実行する

```Shell
docker compose up
```

実行すると下記のようにredisのイメージのpullやDockerfileのビルドが実行され、コンテナが実行されます。

```
$ docker compose up
[+] Running 7/7
 ⠿ redis Pulled 12.9s
...(中略)
[+] Building 61.9s (17/17) FINISHED
 => [internal] load build definition from 
...(中略)
[+] Running 3/3
 ⠿ Network composetest_default    Created 0.0s
 ⠿ Container composetest-web-1    Created 0.1s
 ⠿ Container composetest-redis-1  Created 0.1s
Attaching to composetest-redis-1, composetest-web-1
composetest-redis-1  | 1:C 01 Dec 2022 00:00:00.000 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
...(中略)
composetest-web-1    |  * Running on all addresses (0.0.0.0)
composetest-web-1    |  * Running on http://127.0.0.1:5000
composetest-web-1    |  * Running on http://172.18.0.3:5000
...(中略)
```

別ターミナルで `curl localhost:8000` を打つとレスポンスが返って来て、リクエストのたびに下記のように `seen x times` の `x` の箇所が加算されているのがわかります。

```Shell
$ curl localhost:8000
Hello World! I have been seen 1 times.
$ curl localhost:8000
Hello World! I have been seen 2 times.
$ curl localhost:8000
Hello World! I have been seen 3 times.
```

この数値はアプリケーションのコードを見ると

```Python
return cache.incr('hits')
```

のようにredis側でインクリメントされた数字が返るよう設定されています。
下記が公式ドキュメントとなりますが、 `incr` は名前の通り実行するたびに数値がインクリメントされるコマンドになります。

https://redis.io/commands/incr/

ここまで完了したら、Docker Composeを実行してるターミナルで `Ctrl + C` を入力してDocker Composeの実行を停止します。

## Docker Composeの再実行

さて、それでは再度 `docker compose up` を実行してコンテナを起動しましょう。

別ターミナルで `curl localhost:8000` を打つと先程と同様レスポンスが返って来て、リクエストのたびに下記のように `seen x times` の `x` の箇所が加算されているのがわかります。

```Shell
$ curl localhost:8000
Hello World! I have been seen 4 times.
$ curl localhost:8000
Hello World! I have been seen 5 times.
$ curl localhost:8000
Hello World! I have been seen 6 times.
$ curl localhost:8000
Hello World! I have been seen 7 times.
```

しかし、先程と違って `x` の部分が先程Docker Composeを停止する前にリクエストした回数からインクリメントされているのが確認できるかと思います。
これによって一度停止したDocker Composeのコンテナが停止してもredisのデータが永続化されているのがわかります。

## docker-compose.yml設定内容の解説

```YAML
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_DEBUG: True
  redis:
    image: "redis:alpine"
    volumes:
    - redis_data:/data
volumes:
  redis_data:
```

### version

`version` はこのファイルの設定がDocker Composeのどのバージョンのシンタックスで記載されているものなのかを指定するための設定です。
基本的には記載時点で最新ものを定義しておけば問題ありません。

バージョンについては下記ページで解説されているので興味がある人は読んでみてください。

https://docs.docker.com/compose/compose-file/compose-versioning/

### services

`services` では実行するコンテナの一覧を定義しています。

```YML
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_DEBUG: True
  redis:
    image: "redis:alpine"
    volumes:
    - redis_data:/data
```

今回の `services` の定義ではこのようになっていていますが、それぞれ下記のような内容に設定されています。

- `web` と `redis` の2つのコンテナを起動するよう設定されている
- `web` のコンテナは `build` フィールドが設定されていて、これにより `web` は `docker-compose.yml` のあるディレクトリにあるDockerfileがビルドされたコンテナが実行される
- `redis` のコンテナは `image` で設定されているイメージが実行される
- `web` 側は `ports` でポートフォワーディングが設定されているので、コンテナ側の `5000` ポートがホスト側の `8000` ポートにポートフォワードされる
- `web` 側は `volumes` で`docker-compose.yml` のあるディレクトリがマウントされる
- `web` 側は `environment` で `FLASK_DEBUG=True` に環境変数が設定されてコンテナが実行される
- `redis` 側は `volumes` で `/data` ディレクトリが永続化される(詳しくは後述)

### volumes

`volumes` では作成するボリュームの一覧を定義しています。

```YAML
volumes:
  redis_data:
```

今回の `volumes` では上記のようになっており `docker volume create redis_data` コマンドでボリュームを作成していると考えてもらうと理解しやすいかと思います。
ここで作成したボリュームを `services` の `redis` で利用しています。

今回のアプリケーション例でもredisの `INCR` を利用しているため、Dockerのvolumeの使い方の例と同様、 `INCR` したデータがDocker Composeを再起動しても永続化されているのがここからわかると思います。

