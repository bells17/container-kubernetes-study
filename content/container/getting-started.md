---
title: "Dockerを触ってみる"
date: 2022-12-07T08:09:14Z
draft: false
weight: 2
---

DockerのインストールとDocker Hubへの登録が完了したので、実際にDockerでコンテナの実行を行ってみましょう。

{{< toc >}}

## コンテナを実行する

それでは下記のコマンドを実行して、nginxのコンテナを実行してみてください。

```Shell
docker run -p 8080:80 nginx
```

`docker run` コマンドはコンテナの起動とコマンドの実行を同時に行うコマンドです。

https://docs.docker.com/engine/reference/commandline/run/

今回の例で言えばnginxイメージのコンテナ起動とnginxの起動処理を同時に行ってくれます。

また `-p <host port>:<container pod>` オプションをつけると、コンテナ側の `<container port>`で指定したポートをホスト側の `<host port>`で指定したポートに割り当てることができます。
今回は、実行したnginxコンテナのnginxサーバーにリクエストできかをチェックしたいので、ホスト側からリクエストを送れるようにするために設定しています。

実行中に別のターミナルで `docker ps` というコマンドを実行すると

```
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
62c2cd21f16c   nginx     "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->80/tcp   competent_curie
```

のように出力されてnginxのイメージが実行されているのが確認できます。

`docker ps` はコンテナの一覧を表示してくれるコマンドとなります。
特にオプションを付けないと実行中のコンテナのみを一覧表示するので、 `docker ps` コマンドを実行することで、実行したコンテナが現在も実際に実行中かどうかを確認することができます。

https://docs.docker.com/engine/reference/commandline/ps/

また、この状態で `curl -I localhost:8080` コマンドを実行すると

```
$ curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.23.2
Date: Fri, 09 Dec 2022 07:19:27 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 19 Oct 2022 07:56:21 GMT
Connection: keep-alive
ETag: "634fada5-267"
Accept-Ranges: bytes
```

のようにnginxからレスポンスが返ってくることが確認できるかと思います。

一方で `docker run` を実行した方のターミナルを見ると

```
"HEAD / HTTP/1.1" 200 0 "-" "curl/7.79.1" "-"
```

のようにnginxのアクセスログが出力されていることが確認できると思います。
docker ではコンテナの標準出力/標準エラー出力がログとして表示されるようになっているため、標準出力/標準エラー出力にログを出力されるように設定されているnginxコンテナのログが `docker run` コマンドを実行しているターミナルに出力されています。

ここまでで

- `docker run` コマンドでコンテナの実行が行えること
- `docker ps` コマンドで実行中のコンテナ一覧が確認できること
- nginxのコンテナを起動してcurlでコンテナ上で起動したnginxに対してリクエストが送れること

が確認できましたので、 `docker run` を実行していたターミナルで `Ctrl + C` を押して、コンテナの実行を停止してください。

## コンテナイメージの取得元

先程は `docker run -p 8080:80 nginx` のようにしてnginxのコンテナイメージを実行しましたが、このイメージはどこから取得したものでしょうか？

このnginxのイメージは https://hub.docker.com/_/nginx/ から取得しており、パスなどを指定せずにイメージ名のみを指定すると[Official Images](https://hub.docker.com/search?image_filter=official&q=) からイメージが検索されます。

Official Images以外のイメージを利用する場合は `<ユーザー名>/<リポジトリ名>` のように指定すればDocker Hubにある指定のイメージを利用することができます。
例えば `docker run -p 8080:80 ubuntu/nginx` を実行すると https://hub.docker.com/r/ubuntu/nginx にあるイメージを実行することができます。

また、Docker Hub以外のレジストリを利用する場合は `<レジストリ名>(<ユーザー名>/)/<リポジトリ名>` といった形で指定することが可能で、例えば `docker run k8s.gcr.io/pause` のように利用することができます。


