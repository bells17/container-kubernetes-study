---
title: "作成したイメージをpushしよう"
date: 2022-12-07T08:09:14Z
draft: false
weight: 5
---

独自のコンテナイメージを作ったので、次は作ったイメージを実際にコンテナレジストリにpushしてみましょう。

{{< toc >}}

## Docker Hubにログインする

この資料ではコンテナレジストリとしてDocker Hubを利用しますので、もしまだアカウントの作成を行っていなければアカウントの作成をお願いします。

[「はじめに」](/try-container/introduction)のページでターミナルの `docker login` まで完了していなければ、 `docker login` でDocker Hubにログインをするところまで実行しておいてください。

## イメージをpushする

先程ビルドしたイメージをpushしたいのですが、ちょっと値を変える必要があるためDocker Hubにpushするためのイメージを改めてビルドしていきましょう。

前のページでは `docker build -t hello-dockerfile .` のようにしていましたが `docker build -t <Docker Hub User ID>/hello-dockerfile .` というふうにDocker Hubで作成したアカウントIDをつけてビルドを実行してみてください。 

ビルドが成功したら `docker push <Docker Hub User ID>/hello-dockerfile` というコマンドを実行してイメージをDocker Hubにpushしてみましょう。

著者の場合はアカウントIDが `bells17` となりますので下記のようになります。

```Shell
$ docker build -t bells17/hello-dockerfile .
$ docker push bells17/hello-dockerfile
```

Pushが成功すると下記のようにリポジトリが作成されているのが確認できます。

https://hub.docker.com/repository/docker/bells17/hello-dockerfile



## 新たなタグをつける

イメージタグについては前のページでも解説しましたが、 `docker tag` コマンドで新たなタグをつけることもできるので試してみましょう。

`docker tag <base image tag> <new image tag>` のようにすることで新たなタグを付けることができます。

タグ付きのイメージをpushするには `docker push <Docker Hub User ID>/hello-dockerfile:<new tag>` のようにします。

実際の実行例は下記のようになります。

```Shell
$ docker tag bells17/hello-dockerfile bells17/hello-dockerfile:foo
$ docker push bells17/hello-dockerfile:foo
```

pushしたら、イメージが下記のリンクのようにDocker Hubで確認できます。

https://hub.docker.com/layers/bells17/hello-dockerfile/foo/images/sha256-414aa55f1e9c084df5cf4768dde085e007463d8d4a6ba62df4a8a91b0d60a4db?context=repo

## コンテナイメージをtar形式で出力する

`docker save` コマンドを使うとイメージをtar形式で出力することが可能です。

また、出力したデータは `docker load` コマンドで読み込むことが可能です。

例えば下記のようにイメージのtarへのエクスポートとロードを行うことができます。

```
$ docker save bells17/hello-dockerfile:foo > hello-dockerfile.tar
$ docker load < hello-dockerfile.tar
```

基本的にはあまりsave/loadでイメージをやり取りすることはありませんが、一部のツールだと `docker save` でエクスポートしたイメージを読み込む必要があることがあるため覚えておくと便利です。

参考:

- https://docs.docker.com/engine/reference/commandline/save/
- https://docs.docker.com/engine/reference/commandline/load/

## イメージを削除する

`docker rmi` コマンドを使用すると手元のコンテナイメージの削除が可能です。

例えば下記のように削除できます。

```
$ docker rmi bells17/hello-dockerfile
Untagged: bells17/hello-dockerfile:latest
Untagged: bells17/hello-dockerfile@sha256:414aa55f1e9c084df5cf4768dde085e007463d8d4a6ba62df4a8a91b0d60a4db
Deleted: sha256:87b69702d32c03b418f9b487ea795ea15b51dee9fa6ca20601a89774e49afaff
```

参考:
- https://docs.docker.com/engine/reference/commandline/rmi/
