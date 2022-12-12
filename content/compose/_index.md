---
title: "Docker Compose"
date: 2022-12-07T08:09:14Z
draft: false
weight: 2
---

前の章ではDockerや `Dockerfile` の基本について解説してきましたので、次は実際にコンテナを使った開発を行う際によく使うことになるだろうDocker Composeの使い方について解説していきます。

{{< toc >}}

## Docker Composeとは？

Docker Composeは複数のコンテナの実行をまとめて行えるツールになります。

例えばWebアプリケーションの開発環境をコンテナで構築しようとすると、メインで開発を行うアプリケーションのコンテナ以外にもWebサーバーやデータベースを動かすコンテナなども必要になります。
Docker Composeを使うとこういった複数のコンテナの実行管理を簡単に行うことが可能になります。

Docker Composeについては下記の公式サイトで詳しく解説されていますので、詳細な情報が知りたい方はこちらも確認してみてください。

https://docs.docker.com/compose/

## Docker Composeのインストール

{{< tabs "install-docker" >}}
{{< tab "Linux" >}}
Liuxの場合は下記手順に従ってDocker Composeのインストールを行ってください。

https://docs.docker.com/compose/install/linux/
{{< /tab >}}
{{< tab "macOS" >}}
Macの場合はDocker DesktopにすでにDocker Composeが含まれています。

https://docs.docker.com/desktop/install/mac-install/
{{< /tab >}}
{{< tab "Windows" >}}
Windowsの場合はDocker DesktopにすでにDocker Composeが含まれています。

https://docs.docker.com/desktop/install/windows-install/
{{< /tab >}}
{{< /tabs >}}

## Docker Composeの使い方

[Docker Composeの主な機能とユースケースのページによると](https://docs.docker.com/compose/features-uses/#key-features-of-docker-compose)、Docker Composeの主な機能は下記と書かれています。

- 単一のホスト上に複数の隔離された環境を持つ
- コンテナ作成時のボリュームデータ保持
- 変更されたコンテナのみ再作成する
- 変数をサポートし、環境間で構成を切り替える

まだDocker Composeを実際に試す部分を紹介していないので、あまりイメージが沸かないかもしれませんが、なんとなくこんな機能があるよという感じで見ていただければと思います。

### 単一のホスト上に複数の隔離された環境を持つ

Docker Composeには `プロジェクト` という概念があり、このプロジェクト毎にDocker Composeで実行管理するコンテナグループを管理をしています。
Docker Composeは `Dockerfile` のように `docker-compose.yml` という実行するコンテナを管理する設定ファイルを元にして、コンテナグループの起動などの管理を行うのですが、デフォルトでは実行している `docker-compose.yml` ファイルが置かれているディレクトリがプロジェクト名となります。

各Docker Composeの実行管理はこのプロジェクト毎に管理を行うので、プロジェクトAとプロジェクトBのDocker Composeのコンテナグループをそれぞれ同時に実行するといったことが可能です。
要は何かのアプリケーションのコンテナグループを立ち上げながら別のコンテナグループを相互に影響を与えることなく立ち上げることが可能ということですね。

もし、プロジェクト名を手動で設定したい場合は

- `COMPOSE_PROJECT_NAME` 環境変数を設定する
- `-p, --project-name ` オプションを設定する

ことで任意のプロジェクト名を設定できます。

### コンテナ作成時のボリュームデータ保持

Docker Composeでは、コンテナグループで実行される各コンテナのことを `Service` と呼ぶのですが、この `Service` で使用されるボリュームの保持を行ってくれます。

まだ紹介を行っていなかったのですが、Docker単体でもボリュームを利用することが可能で `docker volume` コマンドでボリュームの操作を行うことが可能です。
`docker volume` を直接使うケースは少ないと思い紹介をしていなかったのですが、Docker ComposeでWebアプリケーションの開発を行う際はデータベースのデータを保持するためにボリューム機能を利用することが多いので、Docker Composeの解説ではこのボリューム機能についても紹介していきます。

`docker volume` について詳しく知りたい場合は下記のリンクを参考にしてください。

https://docs.docker.com/storage/volumes/

### 変更されたコンテナのみ再作成する

Docker Composeでは `docker-compose.yml` ファイルを通して `Dockerfile` をビルドして利用することが可能で、開発を行っているアプリケーションのコンテナでは基本的に `Dockerfile` を書いてアプリケーションの開発をしていくことが多いです。

Docker Composeでは、この `Dockerfile` に変更が行われない限りはDocker Composeの実行ごとに毎回 `Dockerfile` をせず、ビルド済のコンテナを利用するようになっています。
不要なコンテナのビルドが行われなくなることにより、Docker Composeによるプロジェクトの再起動などが高速になります。


### 変数をサポートし、環境間で構成を切り替えが可能である

`docker-compose.yml` ファイルは[実行時に環境変数による設定値の置換が可能](https://docs.docker.com/compose/compose-file/compose-file-v3/#variable-substitution)です。
この機能を利用して、1つの `docker-compose.yml` ファイルだけで実行環境別の構成での環境構築が可能になります。

また、 [`docker-compose.yml` ファイルの `extends` キーワード](https://docs.docker.com/compose/extends/)を利用することにより、 `Service` の設定をすでに定義済の他の `Service` 設定を元に設定が行えたり、他のファイルから設定を読み込むことが可能となっており、これによりより柔軟な `Service` の定義が可能となっています。


