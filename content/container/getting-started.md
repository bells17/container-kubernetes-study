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

実行中に別のターミナルで `curl localhost:8080` のコマンドを実行すると

```
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

のようにレスポンスが返ってくることが確認できるかと思います。

一方で `docker run` を実行した方のターミナルを見ると

```
172.17.0.1 - - [07/Dec/2022:18:28:32 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36" "-"
```

のようにnginxのアクセスログが出力されていることが確認できると思います。

ここまでで

- nginxのコンテナを起動して
- curlでコンテナ上で起動したnginxに対してリクエストが送れる

ということが確認できました。

このように `docker run` コマンドを使うとコンテナの起動ができることがわかりました。

これで `docker run` の動作が簡単に確認できたので、 `docker run` を実行していたターミナルで Ctrl + C を押して、コンテナの実行を停止してください。

## コンテナイメージの取得元

先程は `docker run -p 8080:80 nginx` のようにしてnginxのコンテナイメージを実行しましたが、このイメージはどこから取得したものでしょうか？

このnginxのイメージは https://hub.docker.com/_/nginx/ から取得しており、パスなどを指定せずにイメージ名のみを指定すると[Official Images](https://hub.docker.com/search?image_filter=official&q=) からイメージが検索されます。

Official Images以外のイメージを利用する場合は `<ユーザー名>/<リポジトリ名>` のように指定すればDocker Hubにある指定のイメージを利用することができます。
例えば `docker run -p 8080:80 ubuntu/nginx` を実行すると https://hub.docker.com/r/ubuntu/nginx にあるイメージを実行することができます。

また、Docker Hub以外のレジストリを利用する場合は `<レジストリ名>(<ユーザー名>/)/<リポジトリ名>` といった形で指定することが可能で、例えば `docker run k8s.gcr.io/pause` のように利用することができます。


