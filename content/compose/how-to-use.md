---
title: "Docker Composeの使い方アレコレ"
date: 2022-12-07T08:09:14Z
draft: false
weight: 2
---

`docker-compose.yml` と `docker compose up` の基本的な使い方について解説しましたので、その他実際に使う上で覚えておくと便利なDocker Composeの使い方を紹介します。

{{< toc >}}

## バックグラウンド実行

`docker compose up -d` を使うと `docker run -d` コマンド同様にDocker Composeのコンテナをバックグラウンドで実行可能です。

```Shell
$ docker compose up -d
[+] Running 2/2
 ⠿ Container composetest-redis-1  Started 0.3s
 ⠿ Container composetest-web-1    Started 0.3s
```

実行すると上記のように起動したコンテナが表示されます。

実行中のコンテナは `docker compose ps` コマンドで確認できます。

```Shell
$ docker compose ps
NAME                  COMMAND                  SERVICE             STATUS              PORTS
composetest-redis-1   "docker-entrypoint.s…"   redis               running             6379/tcp
composetest-web-1     "flask run"              web                 running             0.0.0.0:8000->5000/tcp
```

実行中のコンテナのログは `docker compose logs` コマンドで確認できます。

```Shell
$ docker compose logs
composetest-redis-1  | 1:C 1 Dec 2022 00:00:00.00 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
...(中略)
composetest-web-1    |  * Serving Flask app 'app.py'
...(中略)
```

また、`docker exec` のように `docker compose exec` コマンドでコンテナでコマンドの実行が可能です。

```Shell
$ docker compose exec redis sh
/data # whoami
root
/data # exit
```

実行しているコンテナの終了・削除は `docker compose down` コマンドで可能です。

```Shell
$ docker compose down
[+] Running 3/3
 ⠿ Container composetest-redis-1  Removed 0.3s
 ⠿ Container composetest-web-1    Removed 0.3s
 ⠿ Network composetest_default    Removed 0.1s
```

`docker compose down` コマンド実行後に `docker compose ps` コマンドを実行するとコンテナが消えており、終了・削除されたのが確認できます。

```Shell
$ docker compose ps
NAME                COMMAND             SERVICE             STATUS              PORTS
```

## ビルドするDockerfileを指定する

ビルドする `Dockerfile` は下記のように `build` の `context` と `dockerfile` フィールドで指定することが可能です。

```YAML
version: "3.9"
services:
  web2:
    build:
      context: .
      dockerfile: Dockerfile2
    ports:
      - "8001:5000"
    volumes:
      - .:/code
    environment:
      FLASK_DEBUG: True
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

これにより、例えば1つの `docker-compose.yml` で複数のコンテナを独自にビルドして使用する際に、それぞれコンテナの `Dockerfile` を指定して実行させることが可能です。
上記の例では `web2` は `Dockerfile2` を利用してビルド~実行されることになります。

## docker-compose.ymlに定義したコンテナを単体で実行させる

`docker compose run <service>` コマンドを利用すると、 `docker run` のようにDocker Composeのように `docker-compose.yml` に定義したコンテナを単体で実行することが可能です。

例えば下記のように実行することで `web` で設定したコンテナを別途立ち上げることが可能です。

```Shell
$ docker compose up -d
[+] Running 3/3
 ⠿ Network composetest_default    Created 0.1s
 ⠿ Container composetest-redis-1  Started 0.7s
 ⠿ Container composetest-web-1    Started 0.7s
$ docker compose run -d -p 8001:5000 web
93362fe61f73ca0946d41f3da413c44e516a1b1a6ac1c645e6894fe413f50adb
$ curl localhost:8001
Hello World! I have been seen 5 times.
```

例えばbatch jobの動作確認やDBのマイグレーションを手動で行いたいようなケースなど、 `docker compose up` で立ち上げた後に何らかの処理を手動実行したいケースなどに使うことがあるんもで覚えておくと便利です。

`docker compose run` で立ち上げたコンテナは `docker ps` で下記のように確認できます。

```Shell
$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                    NAMES
93362fe61f73   composetest-web   "flask run"              54 seconds ago   Up 52 seconds   0.0.0.0:8001->5000/tcp   composetest_web_run_1efb77381342
8253be967e89   composetest-web   "flask run"              8 minutes ago    Up 8 minutes    0.0.0.0:8000->5000/tcp   composetest-web-1
c957f3075325   redis:alpine      "docker-entrypoint.s…"   8 minutes ago    Up 8 minutes    6379/tcp                 composetest-redis-1
```

上記のようにバックグラウンドで実行され続けるコンテナを `docker compose run` で実行した場合は、コンテナを終了させておかないと `docker compose down` が失敗しますので、利用が終了したら下記のように終了・削除しましょう。

```
$ docker stop 93362fe61f73ca0946d41f3da413c44e516a1b1a6ac1c645e6894fe413f50adb
93362fe61f73ca0946d41f3da413c44e516a1b1a6ac1c645e6894fe413f50adb
$ docker rm 93362fe61f73ca0946d41f3da413c44e516a1b1a6ac1c645e6894fe413f50adb
93362fe61f73ca0946d41f3da413c44e516a1b1a6ac1c645e6894fe413f50adb
```

## depends_on と healthcheck

`docker-compose.yml` には `depends_on` と `healthcheck` というフィールドがあります。

`depends_on` は `services` で立ち上げたコンテナ間の依存関係を設定することが可能で、「コンテナAを実行してからコンテナBを立ち上げるようにする」といった設定が可能です。

一方で `healthcheck` は起動するコンテナが正常状態か判定する機能を提供してくれます。
これによって「ヘルスチェックがパスしてるからコンテナAは正常に起動している」といったチェックが可能です。

上記2つを組み合わせると

- DBの起動を確認してからDBマイグレーション用のコンテナを起動する
- DBマイグレーションの実行が完了してからアプリケーションコンテナを起動する

といったことが可能になります。

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
    depends_on:
      redis:
        condition: service_healthy
      migrate:
        condition: service_completed_successfully
  migrate:
    image: "redis:alpine"
    command:
      - echo
      - "Run some migration command!"
    depends_on:
      redis:
        condition: service_healthy
  redis:
    image: "redis:alpine"
    volumes:
      - redis_data:/data
    healthcheck:
      test: "redis-cli INFO 1>/dev/null"
      interval: 15s
      timeout: 15s
      retries: 0
      start_period: 5s
volumes:
  redis_data:
```

例えば上記のように `docker-compose.yml` を編集して `docker compose up` を実行してみましょう。

```Shell
$ docker compose up
[+] Running 4/3
 ⠿ Network composetest_default      Created 0.0s
 ⠿ Container composetest-redis-1    Created 0.1s
 ⠿ Container composetest-migrate-1  Created 0.0s
 ⠿ Container composetest-web-1      Created 0.0s
Attaching to composetest-migrate-1, composetest-redis-1, composetest-web-1
composetest-redis-1    | 1:C 1 Dec 2022 00:00:00.000 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
..(中略)
composetest-migrate-1  | Run some migration command!
composetest-migrate-1 exited with code 0
composetest-web-1      |  * Serving Flask app 'app.py'
..(中略)
```

のように

- redisの起動完了を待ちmigrationが実行され
- migrationコンテナが完了してからwebコンテナが実行されている

というのが確認できるかと思います。

`depends_on` と `healthcheck` の詳細なシンタックスは下記を参考にしてください。

- https://docs.docker.com/compose/compose-file/#long-syntax-1
- https://docs.docker.com/compose/compose-file/#healthcheck

これらを活用すると開発環境の各種セットアップ処理などを事前に行ってからアプリケーションコンテナを起動する、といったことがDocker Compose1つで完結させることが可能で非常に便利ですので、Docker Composeを使って開発環境などを構築する際は積極的に使っていきましょう。
