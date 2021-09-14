---
title: "HTTPD Buildpack リファレンス"
weight: 300
menu:
  main:
    parent: reference
    identifier: httpd-reference
    name: "HTTPD Buildpack"
---

{{% reference_exec_summary bp_name="Paketo HTTPD Buildpack" bp_repo="https://github.com/paketo-buildpacks/httpd" howto_docs_path="/docs/howto/web-servers/#httpd" %}}

<!-- The HTTPD Paketo Buildpack supports the installation of the
Apache HTTP Server binary distribution
onto the `$PATH` inside a container. This makes it available to subsequent
buildpacks. -->
HTTPD Paketo Buildpack はコンテナの `$PATH` に含まれるディレクトリへ Apache HTTP Server のバイナリアーカイブを展開します。
展開した内容は後に続く Buildpack から利用できます。

## 対応している依存対象

<!-- The HTTPD Paketo Buildpack supports several versions of Apache HTTP Server.
For more details on the specific versions supported in a given buildpack
version, see the [release notes](https://github.com/paketo-buildpacks/httpd/releases). -->
HTTPD Paketo Buildpack は Apache HTTP Server の複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート](https://github.com/paketo-buildpacks/httpd/releases) で確認してください。

## 振る舞い
<!-- When the HTTPD Buildpack participates in a build, it will contribute in one of two ways: -->
HTTPD Buildpack を使用すると次のいずれかの処理を実行します。

<!-- 1. When an `httpd.conf` file **is present** in your app's source code, the
   buildpack will set up an Apache HTTP server with that config.
1. When the `httpd.conf` **is not present** in the app's source code, the
   buildpack simply provides the Apache HTTP Server dependency to subsequent
   buildpacks without actually setting up a server. -->
1. アプリケーションのソースコードに `httpd.conf` が **存在する場合** 、Buildpack は Apache HTTP Server がその設定ファイルを使うようにします
2. アプリケーションのソースコードに `httpd.conf` が **存在しない場合** 、Buildpack は何もしません
