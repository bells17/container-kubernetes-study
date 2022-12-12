---
title: "独自のコンテナイメージを作る"
date: 2022-12-07T08:09:14Z
draft: false
weight: 4
---

`docker run` でコマンドを実行する基本的な方法については紹介したので、次は自分たちで独自のコンテナイメージを構築する方法について学んで行きましょう。

{{< toc >}}

## Dockerfileを書いてみる

独自のコンテナイメージを作成するには `Dockerfile` というファイルを作成して、それを `docker build` コマンドで実際にコンテナイメージをビルドするというステップが必要です。

早速下記のような `Dockerfile` を書いてみましょう。

```Dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:18.04
ARG MESSAGE=Dockerfile
WORKDIR /workspace
COPY ./Dockerfile /Dockerfile
RUN groupadd --gid 1000 node \
    && useradd --uid 1000 --gid node --shell /bin/bash --create-home node \
    && echo "Hello $MESSAGE" > /hello-dockerfile
USER node
CMD cat /hello-dockerfile
```

`Dockerfile` を作成したら、`Dockerfile` を設置したディレクトリで `docker build .` コマンドを実行してみましょう

```Shell
$ docker build -t hello-dockerfile .
$ docker run hello-dockerfile
Hello Dockerfile
```

上記のように `docker run` でビルドしたイメージを実行すると `Hello Dockerfile` と出力されていることから、作成したDockerfileを元にイメージが作成されていることがわかります。

## ビルドしたDockerfileについて

先程

```Dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:18.04
RUN echo "Hello Dockerfile" > /hello-dockerfile
CMD cat /hello-dockerfile
```

という内容の `Dockerfile` をビルドしましたが、この中身について説明していきます。

### # syntax=docker/dockerfile:1

先頭の `# syntax=docker/dockerfile:1` はシンタックスの指定です。
基本的には設定しなくても問題ありませんが、詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/build/buildkit/dockerfile-frontend/

### FROM ubuntu:18.04

`FROM` にはベースとなるコンテナイメージを指定します。
コンテナイメージを作成するにはベースとなるコンテナイメージが必要になりますので、何かしらのベースイメージが必要になります。

ベースとするコンテナイメージは[Docker Hubで公開されているDocker公式イメージ](https://hub.docker.com/search?q=&image_filter=official)を利用することが多いため、特に理由がなければこの中からベースイメージを選択すると良いかと思います。

この公式イメージについては、下記のリンクからそれぞれの公式イメージのDockerfileをたどることが可能ですので、どのようにイメージが構築されているのか気になる方は見てみてください。

https://github.com/docker-library/docs

また、[distrolessというbashなどのツールが一切入っていないイメージ](https://github.com/GoogleContainerTools/distroless)があるため、セキュリティを高めたい本番環境で利用するイメージではこちらをベースイメージとして利用するケースも多いです。

`FROM ubuntu:18.04` という部分の `:18.04` という部分はubuntuイメージの中のタグを指定しています。
コンテナイメージにはそれぞれタグを設定することが可能で、バージョンなどをこのタグを使って設定するために利用されています。

`:18.04` では `18.04` のバージョンのubuntuのイメージを指定していることになります。
タグの指定を行っていない場合は `latest` というタグが自動的に指定されることになるので、これまでの例で使ってきた `ubuntu` や `nginx` のイメージは `latest` タグを利用していることになります。
`latest` タグは最新バージョンのイメージを設定するのが一般的です。

また、ubuntuのイメージのタグがどのようになっているかについては下記のリンクで確認することができます。

https://hub.docker.com/_/ubuntu/tags

これまでの例では説明の簡素化のためタグを指定してきませんでしたが、基本的にはタグを指定して利用するイメージのリビジョンを指定して利用することになります。

コンテナイメージはタグ以外にもイメージIDが設定されていて、これを設定してコンテナイメージを指定することも可能です。
例えば下記のように `@sha256:` の後にイメージIDをつけてコンテナイメージを指定することができます。

```Shell
$ docker run ubuntu@sha256:965fbcae990b0467ed5657caceaec165018ef44a4d2d46c7cdea80a9dff0d1ea echo "hello digest"
hello digest
```

イメージIDは下記リンクのようなイメージの一覧の `DIGEST` というフィールドから探すことができます。

https://hub.docker.com/_/ubuntu/tags

また、ローカルにあるイメージのイメージIDは下記のように `docker images` コマンドで確認することが可能です

```
$ docker images
REPOSITORY                             TAG               IMAGE ID       CREATED        SIZE
ubuntu                                 latest            6b7dfa7e8fdb   8 hours ago    77.8MB
ubuntu                                 18.04             71cb16d32be4   2 months ago   63.1MB
```

`FROM` についてより詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/engine/reference/builder/#from

### ARG MESSAGE=Dockerfile

`ARG` を使用すると `docker build` 時に `--build-arg` オプションで任意の値を渡すことが可能です。
例えば下記のように実行することで `MESSAGE` 変数に指定の値を設定することができます。

```Shell
$ docker build -t hello-dockerfile --build-arg MESSAGE=build-arg .
$ docker run hello-dockerfile
Hello build-arg
```

この `MESSAGE` 変数は `RUN` の箇所で `echo "Hello $MESSAGE" > /hello-dockerfile` のように `$MESSAGE` のように利用することが可能です。
変数は `RUN` に限らず利用することが可能です。

`ARG` についてより詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/engine/reference/builder/#arg

### WORKDIR /workspace

`WORKDIR` を利用するとコンテナ内のデフォルトディレクトリを設定することが可能です。
実際に下記のように `pwd` コマンドをコンテナで実行してみると `/workspace` がデフォルトのディレクトリに設定されているのが確認できます。

```Shell
$ docker run hello-dockerfile pwd
/workspace
```

`WORKDIR` についてより詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/engine/reference/builder/#workdir

### COPY ./Dockerfile /Dockerfile

`COPY` を利用するとホスト側のファイルやフォルダをコンテナにコピーすることが可能です。
ビルドしたアプリケーションのバイナリや設定ファイルなどをコンテナに持ち込む際に利用します。

シンタックスは下記のようになっています

`COPY <host file or dir> <container file or dir>`

`COPY` についてより詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/engine/reference/builder/#copy

### RUN groupadd --gid 1000 node ~

`RUN` を利用するとコンテナイメージ内でコマンドを実行できます。

例えば

- 必要なパッケージのインストール
- アプリケーションのビルド

などを行い必要なコンテナイメージの構築に必要な処理を記述していきます。

ここでは `/hello-dockerfile` ファイルの作成に加えて後の `USER` で設定するユーザーとグループの作成を行っています。

`RUN` についてより詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/engine/reference/builder/#run

### USER node

`USER` を利用するとコンテナ内でコマンドを実行するユーザー(+グループ)を指定できます。

デフォルトでは `root` ユーザーが利用されますが、 `root` ユーザーでコンテナを実行するとホスト側でも `root` ユーザーでこのコンテナプロセスを実行することになり、コンテナが乗っ取られた際などのセキュリティリスク増加につながるため、コンテナのユーザーを指定することが一般的です。

今回のDockerfile では `USER` を `RUN` の後にもってきていますが、これは `USER` を設定するとそれ以降の `RUN` などが指定したユーザーで実行されるため、事前にユーザーとグループの作成行っておくためです。

実際にユーザーが設定されているかは下記のように確認することができます。

```
$ docker run hello-dockerfile whoami
node
```

`USER` についてより詳しく知りたい方は下記リンクを参考にしてください。

https://docs.docker.com/engine/reference/builder/#user

### CMD cat /hello-dockerfile

`CMD` を設定すると、そのコンテナイメージがデフォルトで実行するコマンドを設定することができます。

ビルドの例で `docker run hello-dockerfile` を実行すると `Hello Dockerfile` と出力されたのは、この `CMD` で設定されたコマンドが実行された結果となります。
`CMD` は `docker run <任意のコマンド>` で実行するコマンドを上書きすることが可能です。

```Shell
$  docker run hello-dockerfile echo "Hello Custom Command"
Hello Custom Command
```

## docker buildコマンドについて

先程の例では `Dockerfile` のあるディレクトリで `docker build -t hello-dockerfile .` というコマンドを実行しました。

上記コマンドの `-t hello-dockerfile` というのはイメージ名(+タグ)の設定になります。
コンテナイメージにはイメージ名が必要になりますので、任意のイメージ名を設定する必要があります。

また `docker build -t hello-dockerfile:test .` のようにタグを付けることも可能です。
このコマンドでビルドしたイメージも `docker run hello-dockerfile:test` のようにタグを指定することで実行することが可能です。

ビルドと実行の例は下記のようになります。

```Shell
$ docker build -t hello-dockerfile:test .
$ docker run hello-dockerfile:test
Hello Dockerfile
```

## まとめ

このページでは

- 具体的な `Dockerfile` の書き方
- `docker build` による作成した `Dockerfile` を元にしたコンテナイメージのビルド方法
- `docker run` によるビルドしたイメージの実行方法
- 作成した `Dockerfile` を元に基本的なシンタックスの解説

を行いました。

上記の例を通して最低限知っておくと良い `Dockerfile` の基本的なシンタックスについては紹介できたと考えていますが、その他のシンタックスについては下記を参考にしてください。

https://docs.docker.com/engine/reference/builder

また下記ページに `Dockerfile` のベストプラクティスをDocker社がまとめているので、実際に `Dockerfile` を書いていく際は目を通しておくと良いと思います。

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
