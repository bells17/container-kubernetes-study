---
title: "まとめ"
date: 2022-12-07T08:09:14Z
draft: false
weight: 7
---

ここまでで

- `docker run` によるコンテナの起動
- `docker ps` による起動中コンテナ一覧の表示
- `docker rm` によるコンテナの削除
- `Dockerfile` と `docker build` によるコンテナイメージのビルド方法
- `docker push` によるレジストリへのイメージのpush
- `docker tag` によるタグの設定
- `docker save` と `docker load` のよるイメージのexportと読み込み
- `docker rmi` によるイメージの削除
- `docker volume` によるボリュームの利用

とDockerを使ったコンテナイメージの操作や作成に関する基本的な内容を説明してきました。

ここまでの内容でDockerの使い方に関する基本的な内容については解説できたと考えていますので、次章ではコンテナをまとめて実行管理を行うことができるDocker Composeというツールについて解説していきます。
