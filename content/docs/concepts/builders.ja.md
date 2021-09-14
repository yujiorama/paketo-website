---
title: "Builders"
weight: 420
menu:
  main:
    parent: "concepts"
aliases:
  - /docs/builders/
---

<!-- In the Getting Started tutorial, you used the `pack` CLI and the base `builder` to create a runnable image with your application. This section explains what `builders` are and what builders are released by the Paketo project. -->
チュートリアルでは Paketo base ビルダーでコンテナイメージを作成しました。
このドキュメントではビルダーと、Paketo プロジェクトのリリースしているビルダーについて説明します。

## ビルダーとは何か
<!-- A `builder` is an image that contains three components:
* a set of `buildpacks`, which provide your app's dependencies
* a `stack`, which provides the OS layer for your app image
* the [CNB lifecycle](https://buildpacks.io/docs/concepts/components/lifecycle/), which puts everything together to produce your final app image -->
`builder` とは、次の3種類のコンポーネントを含むコンテナイメージです。

* アプリケーションに必要な依存対象を提供する Buildpack の集合
* コンテナイメージの OS レイヤーを提供するスタック（Stack）
* 最終的なコンテナイメージを作成するために全てを統合する [CNB ライフサイクル](https://buildpacks.io/docs/concepts/components/lifecycle/)

<!-- For more information about `builders`, see [buildpacks.io](https://buildpacks.io/docs/concepts/components/builder/). -->
Builder について詳しく知りたければ [buildpacks.io](https://buildpacks.io/docs/concepts/components/builder/) を参照してください。

## Paket プロジェクトのリリースしているビルダーについて
<!-- The Paketo project releases several builder images to choose from depending on your application needs. These are: -->
Paketo プロジェクトではアプリケーションに応じて選択できるよう、いくつかのビルダーを公開しています。

### Full
<!-- Builder based off of the `ubuntu:bionic` stack. Consists of buildpacks to build most **PHP, Java, Node.js, Go, .NET Core, Ruby, NGINX,** and **HTTPD** apps _**with**_ common C libraries. To build your app with it locally using `pack`, run: -->
スタックは `ubuntu:bionic` で構成されています。
ほとんどのプログラミング言語やWebサーバー（**PHP, Java, Node.js, Go, .NET Core, Ruby, NGINX, HTTPD**）で実行するアプリケーションをビルドする Buildpack で構成されており、標準的な C ライブラリを _**同梱しています**_。
pack コマンドでアプリケーションのコンテナイメージをビルドするときは次のように実行します。

{{< code/copyable >}}
pack build my-app-image --builder paketobuildpacks/builder:full
{{< /code/copyable >}}

<!-- Paketo Full Builder [Github Repo](https://github.com/paketo-buildpacks/full-builder) -->
[Paketo Full ビルダーの GitHub リポジトリ]((https://github.com/paketo-buildpacks/full-builder))

### Base
<!-- Builder based off of the `ubuntu:bionic` stack. Consists of buildpacks to build most **Java, Node.js, Go, .NET Core, Ruby,** and **NGINX** apps _**without**_ common C libraries. To build your app with it locally using `pack`, run: -->
スタックは `ubuntu:bionic` で構成されています。
ほとんどのプログラミング言語やWebサーバー（**Java, Node.js, Go, Ruby, NGINX**）で実行するアプリケーションをビルドする Buildpack で構成されており、標準的な C ライブラリを _**同梱していません**_。
pack コマンドでアプリケーションのコンテナイメージをビルドするときは次のように実行します。

{{< code/copyable >}}
pack build my-app-image --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- Paketo Base Builder [Github Repo](https://github.com/paketo-buildpacks/base-builder) -->
[Paketo Base ビルダーの GitHub リポジトリ]((https://github.com/paketo-buildpacks/base-builder))

### Tiny
<!-- Builder based off of a Distroless `ubuntu:bionic` stack. Consists of buildpacks to build most **Go** and **Java** [GraalVM Native Image](https://www.graalvm.org/docs/reference-manual/native-image/) apps. To build your app with it locally using `pack`, run: -->
スタックは `ubuntu:bionic` の Distroless で構成されています。
**Go** と、**Java** の [ネイティブイメージ](https://www.graalvm.org/docs/reference-manual/native-image/) 実行するアプリケーションをビルドする Buildpack で構成されています。
pack コマンドでアプリケーションのコンテナイメージをビルドするときは次のように実行します。

{{< code/copyable >}}
pack build my-app-image --builder paketobuildpacks/builder:tiny
{{< /code/copyable >}}

<!-- Paketo Tiny Builder [Github Repo](https://github.com/paketo-buildpacks/tiny-builder) -->
[Paketo Tiny ビルダーの GitHub リポジトリ]((https://github.com/paketo-buildpacks/tiny-builder))
