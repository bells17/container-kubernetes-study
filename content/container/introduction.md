---
title: "はじめに"
date: 2022-12-07T08:09:14Z
draft: false
weight: 1
---

まずはコンテナを実行するための環境を構築していきましょう。

{{< toc >}}

## Dockerのインストール

{{< tabs "install-docker" >}}
{{< tab "Linux" >}}
TODO
{{< /tab >}}
{{< tab "macOS" >}}
Macの場合はDocker Desktopを利用します。  
下記のリンクよりDocker Desktopのダウンロード〜インストールを実行してください。

https://www.docker.com/products/docker-desktop/
{{< /tab >}}
{{< tab "Windows" >}}
Windowsの場合はDocker Desktopを利用します。  
下記のリンクの手順に従ってDocker Desktopのダウンロード〜インストールを実行してください。

https://docs.docker.com/desktop/windows/wsl/
{{< /tab >}}
{{< /tabs >}}

インストールが完了したらコマンドラインで下記のコマンドを実行してみてください。

```Shell
docker ps
```

実行して下記の様に出力されたらインストール成功です。

```Shell
$ docker ps
CONTAINER ID   IMAGE                           COMMAND       CREATED       STATUS       PORTS     NAMES
```

## Docker Hubへの登録

コンテナレジストリとして[Docker Hub](https://hub.docker.com/)を使用しますので、アカウントの登録を行ってください。

アカウント登録が完了したら、セキュリティ強化のために下記URLへとアクセスして二段階認証の設定を行ってください。

https://hub.docker.com/settings/security

二段階認証の設定が完了したら、下記の記事を参考にアクセストークンを発行して `docker login` によるDocker Hubへのログイン処理を行ってください。

https://docs.docker.com/docker-hub/access-tokens/
