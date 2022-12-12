---
title: "コンテナを試す"
date: 2022-12-07T08:09:14Z
draft: false
weight: 1
---

この章ではコンテナについて簡単に解説した後、実際にコマンドを実行しながらコンテナについて学んでいきます。

{{< toc >}}

## コンテナとは？

[CNCFによれば](https://glossary.cncf.io/container/)コンテナとは下記のようなものです。

> コンピューターのOSによって管理されるリソースと機能の制約を持つ実行プロセスです。
コンテナプロセスで使用できるファイルは、コンテナイメージとしてパッケージ化されます。
コンテナは同じマシン上で互いに隣接して実行されますが、通常、OSは、個別のコンテナプロセスが相互に対話することを防ぎます。

コンテナと近い技術として仮想化があります。
仮想化はハイパーバイザーを用いてハードウェアをエミュレートすることにより1つのマシン上で様々なOSを実行できる技術ですが、コンテナの場合は(基本的には)1つのOS上でLinuxカーネルの機能を用いて論理的に空間を隔離することで独立した環境を提供してくれます。

{{< figure class="text-center" src="https://www.redhat.com/cms/managed-files/virtualization-vs-containers.png" caption="https://www.redhat.com/ja/topics/containers/whats-a-linux-container より" >}}

## コンテナの特徴

ローカルマシンで開発を行ったり、本番環境で仮想マシンを利用したアプリケーション運用を行うのに比べると、コンテナは以下のような特徴があります。

### ポータビリティ

コンテナを利用する際は、まずプログラムのランタイムやミドルウェアを含めた環境をパッケージングした「コンテナイメージ」を作成を行います。
そして、一般的には実行するアプリケーションのバイナリやプログラムなども、作成するコンテナイメージに含めることになります。

コンテナイメージは、1度作成してコンテナイメージをホストする「コンテナレジストリ」に登録してしまえば、誰でも利用することが可能です。
また、コンテナイメージを作成すれば、他の開発者や本番環境で(基本的に)完全に同じ環境でアプリケーションを動かすことができます。

そのため、開発者同士の開発環境や本番環境での差異を減らすことが可能です。

{{< figure class="text-center" src="https://docs.docker.com/engine/images/architecture.svg" caption="https://docs.docker.com/get-started/overview/ より" >}}


### 低オーバーヘッド

仮想マシンの場合、ハイパーバイザーがハードウェアのエミュレートを行い、その上で様々なOSを動かしますが、コンテナの場合は(基本的には)1つのOS上の1プロセスとして動作します。

コンテナは(基本的には)cgroupやnamespace、capabilityといった様々なLinuxカーネルの機能の組み合わせにより作られるプロセスでしかないため、仮想マシンと比べるとオーバーヘッドが小さく、起動なども高速です。

### 耐障害性

コンテナは論理的に空間を隔離することで独立した環境を提供します。
そのため1つのコンテナで何か障害が発生しても、基本的に他のコンテナには影響を及ぼしません。

また、コンテナの起動や仮想マシンと比べると高速なため、障害が発生してコンテナが停止した際の再起動までの時間も短いため、アプリケーションの回復力と可用性が向上します。

## コンテナランタイム

実際にコンテナを利用する場合は、先程紹介したOCI準拠のコンテナランタイムが必要になります。

代表的なコンテナランタイムとしては[「Docker Engine」](https://docs.docker.com/engine/)というコンテナランタイムがあり、この資料でも主にDocker Engineをコンテナランタイムとして利用していきます。

また、コンテナランタイムには高レベルランタイムと低レベルランタイムがあります。

{{< figure class="text-center" src="https://storage.googleapis.com/static.ianlewis.org/prod/img/771/runtime-architecture.png" caption="https://www.ianlewis.org/en/container-runtimes-part-3-high-level-runtimes より" >}}

上記の図のように高レベルラインタイムは低レベルランタイムに命令を出し、低レベルランタイムがコンテナの起動などを行う仕組みになっています。

主な高レベルラインタイム:

- [containerd](https://containerd.io/)
- [cri-o](https://cri-o.io/)
- [podman](https://podman.io/)

主な低レベルラインタイム:

- [runc](https://github.com/opencontainers/runc)
- [gVisor](https://gvisor.dev/)
- [Kata Containers](https://katacontainers.io/)

## コンテナの仕様

コンテナについては[Open Container Initiative(OCI)](https://opencontainers.org/という団体)が標準化に関する仕様の策定を行っており

- [OCI Runtime Specification](https://github.com/opencontainers/runtime-spec): コンテナの構成、実行環境、およびライフサイクルに関する仕様
- [OCI Image Format Specification](https://github.com/opencontainers/image-spec): コンテナイメージに関する仕様
- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec): コンテンツの配布を促進および標準化するための APIプロトコルに関する仕様

という3つの仕様が今のところ作られています。

また、[OCI Runtime Specification](https://github.com/opencontainers/runtime-spec)のリポジトリ内には[The 5 principles of Standard Containers](https://github.com/opencontainers/runtime-spec/blob/main/principles.md) というドキュメントが作られており

> 標準コンテナの目標は、ソフトウェアコンポーネントとそのすべての依存関係を自己記述的で移植可能な形式でカプセル化することです。
これにより、準拠するランタイムは、基盤となるマシンやコンテナの内容に関係なく、追加の依存関係なしにそれを実行できます。

というふうにコンテナの標準化の目標が掲げられています。

そして5つの原則として

1. 標準操作
1. コンテンツにとらわれないこと
1. インフラにとらわれないこと
1. 自動化のための設計
1. 産業グレードの配送

ということが書かれています。
これらについて詳しく知りたい場合は、ぜひ[The 5 principles of Standard Containers](https://github.com/opencontainers/runtime-spec/blob/main/principles.md)を読んでみてください。

## コンテナレジストリ

作成したコンテナイメージをホストするには、コンテナレジストリを利用します。

代表的なコンテナレジストリとしては以下があります:

- [Docker Hub](https://hub.docker.com/)
- [Github Package Registry](https://github.com/features/packages)
- [Quay.io](https://quay.io/)

また、コンテナレジストリは自分で構築することも可能で、OSSのコンテナレジストリとしては以下のようなものがあります:

- [distribution(Docker Registry)](https://docs.docker.com/registry/)
- [Harbor](https://goharbor.io/)
- [Quay](https://github.com/quay/quay)

## 参考

- https://www.redhat.com/ja/topics/containers/whats-a-linux-container
- https://docs.docker.com/desktop/
- https://speakerdeck.com/makocchi/about-container-runtimes-japan-container-days-v18-dot-04
- https://speakerdeck.com/oracle4engineer/oracle-cloud-hangout-cafe-kontenarantaimufalsewei-lai?slide=10

