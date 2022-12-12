---
title: "基本的なdocker runコマンドを試してみる"
date: 2022-12-07T08:09:14Z
draft: false
weight: 3
---

前のページで `docker run` コマンドでコンテナの実行ができることを確認できたので、次は開発などを行っていく際によく使用する基本的な `docker run` コマンドのオプションなどを試して行きましょう。

{{< toc >}}

## コンテナのバックグラウンド実行

前のページではnginxのコンテナをフォアグラウンドで実行していたため、 `curl` でリクエストを送ったりする際に別ターミナルを立ち上げていましたが、 `docker run -d` のように `-d` オプションを付けることでコンテナをバックグラウンドで実行することができます。

では早速試していきましょう。  
下記のコマンドを実行してみてください。

```Shell
docker run -d -p 8080:80 nginx
```

実行すると下記のようにハッシュ値のようなものが出力されます。

```Shell
$ docker run -d -p 8080:80 nginx
9e8b652a5ca924c9693422e5931efdb3ae883e4cce9c9c136756fc4dbe84bf31
```

このハッシュ値のようなものがバックグラウンドで実行したコンテナのIDとなります。

### コンテナIDによるコンテナのフィルタリング

```Shell
docker ps  --filter "id=<container ID>"
```

のように `<container ID>` のところに出力されたコンテナIDを入力して実行すると

```Shell
$ docker ps --filter "id=9e8b652a5ca924c9693422e5931efdb3ae883e4cce9c9c136756fc4dbe84bf31"
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
9e8b652a5ca9   nginx     "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   0.0.0.0:8080->80/tcp   competent_morse
```

といった感じに `docker ps` 　で出力されるコンテナをフィルタリングすることができます。

ちなみに、上記の例だとコンテナIDが `9e8b652a5ca9` のように短縮されていますが

```Shell
$ docker ps --filter "id=9e8b652a5ca9"
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
9e8b652a5ca9   nginx     "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp   competent_morse
```

のように、短縮されたIDでもフィルタリングすることが可能です。

### バックグラウンド実行コンテナの確認とログ表示

実行したコンテナが本当に動作しているのか確認するために `curl -I localhost:8080` を実行してみましょう。

```Shell
$ curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.23.2
Date: Fri, 09 Dec 2022 07:58:17 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 19 Oct 2022 07:56:21 GMT
Connection: keep-alive
ETag: "634fada5-267"
Accept-Ranges: bytes
```

実行するとフォアグラウンドで実行したときと同様に上記のようなレスポンスを得ることができます。
これでバックグラウンドでもnginxのコンテナが動作していることが確認できました。

nginx側でリクエストされたアクセスログなどの確認ですが、バックグラウンドで実行しているコンテナのログは `docker logs <container ID>` で表示することができます。

```Shell
$ docker logs 9e8b652a5ca9
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/01/01 00:00:00 [notice] 1#1: using the "epoll" event method
2022/01/01 00:00:00 [notice] 1#1: nginx/1.23.2
2022/01/01 00:00:00 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/01/01 00:00:00 [notice] 1#1: OS: Linux 5.10.124-linuxkit
2022/01/01 00:00:00 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/01/01 00:00:00 [notice] 1#1: start worker processes
2022/01/01 00:00:00 [notice] 1#1: start worker process 29
2022/01/01 00:00:00 [notice] 1#1: start worker process 30
2022/01/01 00:00:00 [notice] 1#1: start worker process 31
2022/01/01 00:00:00 [notice] 1#1: start worker process 32
172.17.0.1 - - [01/Dec/2022:00:00:00 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.79.1" "-"
```

このように、バックグラウンドで実行するコンテナのログを確認するにはコンテナIDが必要となりますので、 `docker ps` コマンドを使ってIDの確認などを行うことが度々発生します。

### 実行中コンテナでコマンドを実行する

`docker exec` コマンドを利用すると、実行中コンテナでコマンドを行うことが可能です。

```
$ docker exec -it 9e8b652a5ca9 ls /usr/share/nginx/html
50x.html  index.html
$ docker exec -it 9e8b652a5ca9 sh
# whoami
root
# exit
```

このように `-it` オプションつけることでシェルを利用することも可能なので、デバッグなどが必要なときは `dokcer exec` コマンドでコンテナ内を確認してみましょう。

### バックグラウンドで起動したコンテナの実行を終了する

フォアグラウンドで実行していたコンテナは `Ctrl + C` を押すことで終了させていましたが、バックグラウンドで実行しているコンテナを終了させるには `docker stop <container ID>` コマンドを実行する必要があります。
更にコンテナを終了後に削除するには `docker rm <container ID>` コマンドを実行する必要があります。

そのためまとめると下記のようなコマンドを実行する必要があります。

```Shell
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
9e8b652a5ca9   nginx     "/docker-entrypoint.…"   16 minutes ago   Up 16 minutes   0.0.0.0:8080->80/tcp   competent_morse
$ docker stop 9e8b652a5ca9
9e8b652a5ca9
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$ docker rm 9e8b652a5ca9
9e8b652a5ca9
```

上記のように、はじめに `docker ps` で表示されていたコンテナが `docker stop <container ID>` コマンドの実行後に `docker ps` で再度確認すると表示されてなくなっていることが確認できるかと思います。
これでコンテナを終了させることができたのが確認できました。

その後更に `docker rm` でコンテナの削除も行ったので削除も完了しているはずです。

念の為 `curl -I localhost:8080` でコンテナのnginxにリクエストを送れるかを確認してみます。

```Shell
$ curl -I localhost:8080
curl: (7) Failed to connect to localhost port 8080 after 7 ms: Connection refused
```

すでにコンテナは終了・削除してるので、上記のように接続エラーが発生するのが確認できました。

### まとめ

ここまでで

- `docker run -d` でコンテナのバックグラウンド実行ができること
- `docker run -d` の実行結果としてコンテナIDが得られること
- `docker ps  --filter "id=<container ID>"` でコンテナIDによる実行中コンテナのフィルタリングができること
- `docker logs <container ID>` で指定したコンテナのログが確認できること
- `docker stop <container ID>` で指定したコンテナを終了させることができること
- `docker rm <container ID>` で指定したコンテナを削除させることができること

が確認できたかと思います。

`docker run` コマンドを実際に使う際はバックグラウンドで実行するケースが多くなると思いますので、ここで説明した一通りは覚えておくと便利です。

## コンテナを実行終了時に削除する

コンテナを終了したあとに削除するには `docker rm` コマンドの実行が必要だと記載しましたが、 `docker run --rm` のように `--rm` オプションを付けて実行することで、コンテナ終了時に自動的に削除を行うように設定できます。

```Shell
$ docker run --rm -d -p 8080:80 nginx
0260d72a4f967634a3de6d0b801f8de73baf541823b926db2d94165b75de0071
$ curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.23.2
Date: Mon, 12 Dec 2022 08:10:28 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 19 Oct 2022 07:56:21 GMT
Connection: keep-alive
ETag: "634fada5-267"
Accept-Ranges: bytes

$ docker stop 0260d72a4f967634a3de6d0b801f8de73baf541823b926db2d94165b75de0071
0260d72a4f967634a3de6d0b801f8de73baf541823b926db2d94165b75de0071
$ docker rm 0260d72a4f967634a3de6d0b801f8de73baf541823b926db2d94165b75de0071
Error: No such container: 0260d72a4f967634a3de6d0b801f8de73baf541823b926db2d94165b75de0071
```

上記にように `--rm` オプションをつけて実行すると、`docker stop` で停止したあとにコンテナが自動で削除されるため、`docker rm` で削除するコンテナが見つからないというエラーが出るのが確認できるかと思います。

### TIPS: コンテナのライフサイクル

上記まででコンテナの終了と削除は異なるということがなんとなく伝わったかと思いますが、コンテナのライフサイクルをもう少し細かく別れていて主に

- 作成済
- 実行中
- 終了済
- 削除

となっています。

`docker run` を使うとコンテナの作成~実行まで同時に行っているので作成を意識することがあまり無いのですが、下記のように `docker create` と `docker start` コマンドを使用して `docker run` を置き換えることも可能です。

```Shell
$ docker create -p 8080:80 nginx
a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
$ docker start a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
$ curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.23.2
Date: Mon, 12 Dec 2022 08:23:30 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 19 Oct 2022 07:56:21 GMT
Connection: keep-alive
ETag: "634fada5-267"
Accept-Ranges: bytes

$ docker stop a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
$ docker rm a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
a34c45e8397892341ee92d1bad15250e7da0e30412db9d1d561bd7e77d698978
```

実際のコンテナのステータスは上記のライフサイクルで紹介したもの以外にも存在するのですが、基本的には上記のライフサイクルを理解しておけば問題ないと思います。

ステータスの一覧については下記を参照してください。

https://docs.docker.com/engine/reference/commandline/ps/#status

## コンテナで任意のコマンドを実行する

`docker run` コマンドを実行するとデフォルトで事前に設定されたコマンドが実行されますが、任意のコマンドを実行させることも可能です。
例えば `docker run nginx echo "Hello Docker!"` というコマンドを実行してみると

```Shell
$ docker run nginx echo "Hello Docker!"
Hello Docker!
```

のように `echo` コマンドを実行することができます。

### コンテナ起動時にshellを実行する

`docker run -it nginx sh` コマンドを実行すると

```Shell
$ docker run -it nginx sh
# echo "Hello Docker!"
Hello Docker!
# exit
```

のようにshellを実行することが可能です。  

実行する際は上記のように

- `--interactive`,`-i`: STDIN を開いたままにする
- `--tty`,`-t`: pseudo-TTY割り当て

という2つのオプションをつけて上げる必要があります。

https://docs.docker.com/engine/reference/commandline/run/#assign-name-and-allocate-pseudo-tty---name--it

また、nginxのコンテナイメージの場合 `sh` がコンテナイメージにインストールされているので上記のように `docker run` で `sh` が実行できていますが、 `sh` などがインストールされていないイメージも多く、そういった場合はshellの実行自体ができないということを留意しておくと良いです。

nginxのコンテナイメージの場合は `bash` がインストールされていないため `sh` を利用しましたが、ubuntuイメージの場合は下記のように `bash` などもデフォルトでりようすることが可能ですので、shellを試したい場合はubuntuなどのイメージを利用した方が自由度が高いです。

```Shell
$ docker run -it ubuntu bash
root@501c433d8c50:/#  cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/usr/bin/sh
/bin/dash
/usr/bin/dash
root@501c433d8c50:/# exit
```

## ホストディレクトリをマウントする

`docker run -v <host dir>:<container dir>` のように `-v <host dir>:<container dir>` オプションを付けることで指定のディレクトリをコンテナ側にマウントすることができます。

```Shell
$ mkdir example
$ touch example/hello-docker
$ docker run -it -v $(pwd)/example:/example ubuntu bash
root@81678e0a553e:/# ls -la /example/
total 4
drwxr-xr-x 3 root root   96 Dec  9 08:40 .
drwxr-xr-x 1 root root 4096 Dec  9 08:42 ..
-rw-r--r-- 1 root root    0 Dec  9 08:40 hello-docker
root@81678e0a553e:/# exit
exit
```

上記のように `example/hello-docker` ファイルを作成して、 `-v` オプションで `example` ディレクトリをマウントすることが可能です。
また `<host dir>` は絶対パスでの指定が必要なため、上記の例では `$(pwd)/example` のように指定しています。

## 環境変数を設定する

`docker run -e <env name>:<env value>` のように `-e <env name>:<env value>` オプションを使用することで環境変数をコンテナに設定することができます。
`-e` オプションは複数設定することができるので複数の環境変数を設定することも可能です。

実行例は下記のようになります。

```Shell
$ docker run -e foo=bar -e hoge=piyo -it ubuntu bash
root@f2c0999dcaa5:/# env | grep -e foo -e hoge
foo=bar
hoge=piyo
root@f2c0999dcaa5:/# exit
exit
```

また `--env-file` というオプションで環境変数をファイルでまとめて渡すことも可能です。
このファイルのフォーマットは `<env name>=<env value>` でそれぞれの環境変数を設定できます( `#` がコメントとなります )。

`--env-file` オプションを使った実行例は下記のようになります。

```Shell
$ cat <<EOL > env.list
# This is a comment
VAR1=value1
VAR2=value2
EOL
$ docker run --env-file env.list ubuntu env | grep -e VAR
VAR1=value1
VAR2=value2
```

コンテナを実行する際にコンテナイメージに外部からの設定値を持ち込むには、ボリュームをマウントしてファイルなどでデータを渡すか、もしくは環境変数を通して設定値などを渡すパターンが一般的ですので `-v` によるマウントに加えて `-e` による環境変数の設定方法については覚えておくと良いと思います。

## volumeを利用する

Dockerは `docker volume` コマンドでボリュームを扱うことが可能です。

- `docker volume create` でボリュームの作成
- `docker volume ls` でボリューム一覧の確認

がそれぞれ可能になっており

```Shell
$ docker volume create redis
redis
$ docker volume ls
DRIVER    VOLUME NAME
local     redis
```

のようにボリュームを作成して、作成したボリュームを確認することが可能です。

作成したボリュームをコンテナで利用することが可能で、例えば下記のように利用することが可能です。

### volumeを利用したコンテナの作成

ここではredisのコンテナを利用して作成したボリュームを試して行きます。
まずはバックグラウンドでredisコンテナを実行します。

`docker run` の際に `--name` オプションでコンテナの名前を指定することが可能です。
指定することで後の操作が簡単になるので、今回は `--name` オプションをつけて実行します。

```Shell
$ docker run -d \
  --name redis-test \
  --mount source=redis,target=/data \
  redis:alpine
9cbc65469e5965df1ed27ef7a48fd5703366f51bf89ce0080fde5fa1614f583e
```

### データの書き込みとコンテナの停止・削除

それでは実行したredisコンテナで `INCR` コマンドを使ってデータを作成していきます。
`ICCR` コマンドは引数のキー名の値を0から実行毎に1ずつインクリメントしていくコマンドになります。

https://redis.io/commands/incr/

`INCR` コマンド実行後にコンテナの停止と削除を行います。

```Shell
$ docker exec -it redis-test redis-cli
127.0.0.1:6379> INCR count
(integer) 1
127.0.0.1:6379> INCR count
(integer) 2
127.0.0.1:6379> INCR count
(integer) 3
127.0.0.1:6379> exit
$ docker stop redis-test
redis-test
$ docker rm redis-test
redis-test
```

### 再びredisコンテナを起動して、データが永続化されていることを確認する

先程と同様にredisコンテナを同じボリュームを指定して実行して `INCR` コマンドを実行してみましょう。

```Shell
$ docker run -d \
  --name redis-test \
  --mount source=redis,target=/data \
  redis:alpine
c97d619ecd86c095e19f015c5419987dc475ff2664ad76773a7478e687dddf1a
$ docker exec -it redis-test redis-cli
127.0.0.1:6379> INCR count
(integer) 4
127.0.0.1:6379> INCR count
(integer) 5
127.0.0.1:6379> INCR count
(integer) 6
127.0.0.1:6379> exit
```

実行すると、今度は `INCR` のカウントが `4` から始まっており、先程のデータが引き継がれていることがわかると思います。

### ボリュームデータの確認

データが永続化されていることはわかりましたが、このデータはどこにあるのでしょうか？

`docker volume inspect` コマンドを使うと下記のようにボリュームに関するデータが確認可能です。

```Shell
$ docker volume inspect redis
[
    {
        "CreatedAt": "2022-12-01T00:00:00Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/redis/_data",
        "Name": "redis",
        "Options": null,
        "Scope": "local"
    }
]
```

この `Mountpoint` というのがボリュームのあるパスになります。

{{< tabs "install-docker" >}}
{{< tab "Linux" >}}
Linuxの場合はホスト上で直接Dockerを起動しているはずなので普通に `Mountpoint` を `ls` コマンドで叩けばそこに `dump.rdb` というredisのバックアップファイルがあることが確認できます。

```Shell
$ ls /var/lib/docker/volumes/redis/_data
dump.rdb
```
{{< /tab >}}
{{< tab "macOS" >}}
MacでDocker Desktopを利用している場合、Macは実際にはVM内で起動したDockerを操作しているため下記のように `justincormack/nsenter1` というイメージを使用することでDocker DesktopのVM内でコマンドを実行できます。

ここでは `ls` コマンドで `dump.rdb` というredisのバックアップファイルがあることを確認しています。

```Shell
$ docker run -it --privileged --pid=host justincormack/nsenter1 /bin/ls /var/lib/docker/volumes/redis/_data
dump.rdb
```

参考記事: https://www.docker.com/blog/capturing-logs-in-docker-desktop/
{{< /tab >}}
{{< tab "Windows" >}}
TODO
{{< /tab >}}
{{< /tabs >}}

上記のようにして `Mountpoint` のパスが作成されており、ボリュームのデータが作られていることが確認できたので、 `docker volume` が作成するボリュームデータについても確認することができました。

### クリーンアップ

諸々の確認ができたので、最後にコンテナやボリュームの削除を行います。

```Shell
$ docker stop redis-test
redis-test
$ docker rm redis-test
redis-test
$ docker volume rm redis
redis
```

### volumeまとめ

上記のようにDockerからボリュームを扱うことが可能です。
`docker` コマンドからボリュームを直接扱うケースは比較的少ないかもしれませんが、後から紹介するDocker Composeなどを利用してWebアプリケーションの開発環境を構築したりする際などには利用するケースも多くなると思われるため、ボリュームの利用方法について把握しておくと後々役立つかと思います。

volumeについては下記の記事も参考になるかと思いますので、必要に応じて参照してください。

https://docs.docker.com/storage/volumes/

## その他

下記ページを見るとここでは紹介しなかった `docker run` のオプションとその説明があるので、必要に応じて確認してみましょう。

https://docs.docker.com/engine/reference/commandline/run

また、Dockerではネットワークを手動で作成することも可能です。
手動でネットワーク設定を行う必要のあるケースあまり多くないとは思いますが、興味のある方は下記を参考にしてみてください。

https://docs.docker.com/network/
