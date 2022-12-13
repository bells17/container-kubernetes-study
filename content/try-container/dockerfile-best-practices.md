---
title: "Dockerfileベストプラクティス"
date: 2022-12-07T08:09:14Z
draft: false
weight: 7
---

`Dockerfile` のベストプラクティスについては下記のリンクでDocker社がまとめているので、実際に `Dockerfile` を書いていく際は別途目を通しておくと良いと思います。

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

ただ少し長いので、このページでは `Dockerfile` を書くにあたって、最低限これくらいは気をつけておいた方が良いかなという内容をまとめておきます。

{{< toc >}}

## 秘密情報を埋め込まない

パスワードやAPI Keyなどの情報をコンテナに埋め込まないようにしてください。
コンテナイメージから埋め込んだパスワードやAPI Keyが漏洩するリスクがあります。

## 環境に依存する設定値を埋め込まない

これは `Dockerfile` もコンテナ化するアプリケーションについても言えることですが、「環境に依存する設定値を埋め込まない」ようにするべきです。
例えば開発環境も、本番環境も、Dockerが提供するボリュームのマウントや環境変数の設定機能を利用することで、開発環境や本番環境を切り替えることができるようにするべきです。

これにより開発環境も本番環境も同じ `Dockerfile` で構築したイメージを利用することができるようになり、環境の差異を減らすことに繋がります。
そのため、特に理由がなければ基本的にはどの環境向けにも1つの `Dockerfile` で1つのコンテナイメージで動作できるようにしましょう。

## ログはstdout/stderrに出力する

コンテナのログはstdout/stderrに出力したものを `docker log` コマンドで確認することが可能です。
そのため、ログは基本的にstdout/stderrに出力するように設定しましょう。

これはKubernetesなどのコンテナオーケストレーターを利用しても同様で、基本的にログをstdout/stderrに出力することを前提としているので、この前提に合わせてアプリケーションなどの設定も行うようにしましょう。

## イメージサイズを小さくする

コンテナイメージはリモートにある際はイメージをpull(ダウンロード)してくるため、イメージサイズが大きいほど都度pullするデータ量が増えてしまいます。
そのため、下記のような工夫によってコンテナイメージのサイズは可能な限り小さく保つようにしたほうが良いです。

- サイズの小さなベースイメージを使用する
- [`.dockerignore` ファイルを使った不要ファイルの除外](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#exclude-with-dockerignore)
- [multi-stage buildを使ってアプリケーションのバイナリのみをコンテナに入れる](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#use-multi-stage-builds)
- [`RUN` をまとめてイメージのレイヤーを最小限に抑える](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#minimize-the-number-of-layers)

## PID 1 問題を回避する

コンテナが実行するアプリケーションなどのプロセスは基本的にPID 1で実行されます。
PID 1のプロセスはLinuxカーネルに特別扱いされていて、そのプロセス自身が明示的に送信されたシグナルをハンドリングしていない場合それを無視するため

- 明示的にシグナルをハンドリングするようにする
- PID 1 で実行されないようにする

のいずれかの対応が必要になります。

こちらについて詳しくは下記の記事を参考にしてください。

https://text.superbrothers.dev/200328-how-to-avoid-pid-1-problem-in-kubernetes/

## 実行ユーザーをroot以外にする

デフォルトでは `root` ユーザーが利用されますが、 `root` ユーザーでコンテナを実行するとホスト側でも `root` ユーザーでこのコンテナプロセスを実行することになり、コンテナが乗っ取られた際などのセキュリティリスク増加につながるため、コンテナのユーザーを指定することが一般的です。

`root` で動作させる必要の無いコンテナについては、基本的には `root` 以外のユーザーで動作させるように設定しましょう。

## distrolessをベースイメージとして使用する

[distrolessというbashなどのツールが一切入っていないイメージ](https://github.com/GoogleContainerTools/distroless)があるため、セキュリティを高めたい本番環境で利用するイメージではこちらをベースイメージとして利用しましょう。

bashなどが入っておらず使いづらいなどdistroless以外のイメージをベースイメージとして利用する場合は、[Docker Hubの公式イメージ](https://hub.docker.com/search?q=&image_filter=official)など信頼できるイメージをベースイメージとして利用しましょう。

distroless以外によく使われるベースイメージとして[ubuntu](https://hub.docker.com/_/ubuntu)、サイズが軽量なイメージとして[busybox](https://hub.docker.com/_/busybox)や[alpine](https://hub.docker.com/_/alpine)といったものが使われることが多い印象です。

## ベースイメージのタグを指定する

`Dockerfile` で使用するベースイメージについても下記のようにタグを指定するようにしましょう。

```Dockerfile
FROM ubuntu:20.04
```

タグの指定が無いと `latest` タグのイメージが自動で使用されるため、ビルドするタイミングや環境によって利用するベースイメージが変わってしまいます。

## その他の参考記事

Docker社とは別にsysdigが出しているベストプラクティスもあるので目を通しておくと良いと思います。

https://sysdig.jp/blog/dockerfile-best-practices/

