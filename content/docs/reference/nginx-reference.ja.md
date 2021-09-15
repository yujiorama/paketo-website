---
title: "NGINX Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: nginx-reference
    name: "NGINX Buildpack"
---

{{% reference_exec_summary bp_name="Paketo NGINX Buildpack" bp_repo="https://github.com/paketo-buildpacks/nginx" howto_docs_path="/docs/howto/web-servers/#nginx" %}}

<!-- The NGINX Paketo Buildpack supports the installation of the NGINX binary distribution onto
the `$PATH` inside a container. This makes it available to subsequent
buildpacks. -->
NGINX Paketo Buildpack はコンテナの `$PATH` に含まれるディレクトリへ Nginx のバイナリアーカイブを展開します。
展開した内容は後に続く Buildpack から利用できます。

## 対応している依存対象

<!-- The NGINX Paketo Buildpack supports several versions of NGINX.
For more details on the specific versions supported in a given buildpack
version, see the [release notes](https://github.com/paketo-buildpacks/nginx/releases). -->
NGINX Paketo Buildpack は NGINX の複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート](https://github.com/paketo-buildpacks/nginx/releases) で確認してください。

## 振る舞い
<!-- When the NGINX Buildpack participates in a build, it will contribute in one of two ways: -->
NGINX Buildpack を使用すると次のいずれかの処理を実行します。

<!-- 1. When an `nginx.conf` file **is present** in your app's source code, the
   buildpack will set up an NGINX server with that config.

1. When the `nginx.conf` **is not present** in the app's source code, the
   buildpack simply provides the NGINX dependency to subsequent buildpacks
   without actually setting up a server. -->
1. アプリケーションのソースコードに `nginx.conf` が **存在する場合** 、Buildpack は NGINX server がその設定ファイルを使うようにします
2. アプリケーションのソースコードに `nginx.conf` が **存在しない場合** 、Buildpack は何もしません
