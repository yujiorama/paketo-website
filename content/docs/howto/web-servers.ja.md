---
title: "Paketo Buildpacks で Web Server をビルドする"
weight: 308
menu:
  main:
    parent: "howto"
    name: "Web Servers"
aliases:
  - /docs/buildpacks/language-family-buildpacks/httpd/
  - /docs/buildpacks/language-family-buildpacks/nginx/
---

<!-- This documentation explains how to use Paketo buildpacks to build applications
that run web servers like HTTPD and NGINX. These docs focus on explaining
common user workflows. For more in-depth
description of the buildpacks' behavior and configuration, see the reference documentation
for each web server buildpack. -->
このドキュメントでは Paketo Buildpack を使用して HTTPD や NGINX で実行するアプリケーションのコンテナイメージを作成する方法を説明します。
一般的な手順を説明しているだけなので、Buildpack の振る舞いや設定方法を詳しく知りたいときは、それぞれの Web Server 用 Buildpack のリファレンスを参照してください。

## HTTPD

{{% howto_exec_summary bp_name="Paketo HTTPD Buildpack" bp_repo="https://github.com/paketo-buildpacks/httpd" reference_docs_path="/docs/reference/httpd-reference" %}}

### サンプルアプリをビルドする
<!-- To build a sample app locally with this CNB using the `pack` CLI, run -->
`pack` コマンドを使うと、Apache HTTP Server Cloud Native Buildpack で次のようにサンプルアプリケーションをビルドできます。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/httpd
pack build my-app --buildpack gcr.io/paketo-buildpacks/httpd \
  --builder paketobuildpacks/builder:full
{{< /code/copyable >}}

<!-- See [samples](https://github.com/paketo-buildpacks/samples/tree/main/httpd)
for how to run the app. -->
アプリケーションの実行方法は [ソースコードリポジトリの README](https://github.com/paketo-buildpacks/samples/tree/main/httpd) を参照してください。

<!-- **NOTE: The Paketo Full builder is required because HTTPD relies on operating
system libraries only present in the Full builder.** -->
**注意：Apache HTTP Server Cloud Native Buildpack は Paketo Full ビルダーが必要です。Full ビルダーにしか含まれないOSのシステムライブラリを使用するからです。**

### インストールするバージョンを指定する

<!-- The HTTPD CNB (Cloud Native Buildpack) allows you to specify a version of the
Apache HTTP Server to use during deployment. This version can be specified
through the `BP_HTTPD_VERSION` environment variable. When specifying a version
of the Apache HTTP Server, you must choose a version that is available within the
buildpack. The supported versions can be found [here](https://github.com/paketo-buildpacks/httpd/releases) -->
Apache HTTPD Server Cloud Native Buildpack では、デプロイする Apache HTTP Server のバージョンを環境変数 `BP_HTTPD_VERSION` で指定できます。
Apache HTTPD Server Cloud Native Buildpack で使用できるバージョンは、 [Apache HTTPD Server Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/httpd/releases) で確認できます。

<!-- Specifying a version of `httpd` is not required. In the case that it is not
specified, the buildpack will provide the default version listed in the release
notes. -->
`httpd` のバージョンは明示的に指定しなくても構いません。
その場合、Apache HTTPD Server Cloud Native Buildpack のリリースノートに記述された既定のバージョンを使用します。

#### 環境変数 BP_HTTPD_VERSION を使用する

<!-- To configure the buildpack to use HTTPD v2.4.46 when deploying your app, set the
following environment variable at build time, either directly (ex. `pack build
my-app --env BP_HTTPD_VERSION=2.4.*`) or through a
[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md)
file: -->
アプリケーションを HTTPD v2.4.46 でデプロイするときは、ビルド時に環境変数 `BP_HTTPD_VERSION` を指定します。
pack コマンドの `--env` フラグでも指定できますし、 [project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md) でも指定できます。

{{< code/copyable >}}
BP_HTTPD_VERSION="2.4.46"
{{< /code/copyable >}}

#### 非推奨：`buildpack.yml` で指定する

<!-- Specifying the HTTP Server version through `buildpack.yml` configuration will
be deprecated in Apache HTTP Server Buildpack v1.0.0. To migrate from using
`buildpack.yml` please set the `$BP_HTTPD_VERSION` environment variable. -->
Apache HTTP Server Buildpack の v1.0.0 から、`buildpack.yml` でバージョンを指定するのは非推奨になりました。
環境変数 `$BP_HTTPD_VERSION` を使用してください。

### コンテナイメージを実行するときに HTTPD Server を起動する

<!-- Include an `httpd.conf` file in your application's source code. The HTTPD Paketo buildpack
will install the Apache HTTP Server binary _and_ configure it to start when the app image
launches. -->
アプリケーションのソースコードに `httpd.conf` を含めるようにします。
Apache HTTPD Server Cloud Native Buildpack は、Apache HTTPD Server の実行可能ファイルをインストールするだけでなく、コンテナイメージを実行するときに起動できるように構成します。

## NGINX

{{% howto_exec_summary bp_name="Paketo NGINX Buildpack" bp_repo="https://github.com/paketo-buildpacks/nginx" reference_docs_path="/docs/reference/nginx-reference" %}}

### サンプルアプリをビルドする
<!-- To build a sample app locally with this CNB using the `pack` CLI, run -->
`pack` コマンドを使うと、NGINX Server Cloud Native Buildpack で次のようにサンプルアプリケーションをビルドできます。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/nginx
pack build my-app --buildpack gcr.io/paketo-buildpacks/nginx \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- See [samples](https://github.com/paketo-buildpacks/samples/tree/main/nginx)
for how to run the app. -->
アプリケーションの実行方法は [ソースコードリポジトリの README](https://github.com/paketo-buildpacks/samples/tree/main/nginx) を参照してください。

<!-- **NOTE: Though the example above uses the Paketo Base builder, this buildpack is
also compatible with the Paketo Full builder.** -->
**注意：この例では Paketo Base ビルダーを使っていますが、NGINX Server Cloud Native Buildpack は Paketo Full ビルダーと互換性があります**

### インストールするバージョンを指定する

<!-- The NGINX CNB (Cloud Native Buildpack) allows you to specify a version of NGINX to use during
deployment. This version can be specified in a number of ways, including
through `buildpack.yml`. When specifying a
version of the NGINX engine, you must choose a version that is available
within the buildpack. -->
NGINX Server Cloud Native Buildpack では、デプロイする NGINX のバージョンを変更できます。
NGINX Server Cloud Native Buildpack で使用できるバージョンは、 [Apache HTTPD Server Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/httpd/releases) で確認できます。

<!-- Specifying a version of `nginx` is not required. In the case that it is not
specified, the buildpack will provide the default version listed in the release
notes. -->
`nginx` のバージョンは明示的に指定しなくても構いません。
その場合、NGINX Server Cloud Native Buildpack のリリースノートに記述された既定のバージョンを使用します。

#### 環境変数 BP_NGINX_VERSION を使用する

<!-- To configure the buildpack to use NGINX v1.19.8 when deploying your app, set the
following environment variable at build time, either directly (ex. `pack build
my-app --env BP_NGINX_VERSION=1.19.8`) or through a
[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md)
file: -->
アプリケーションを NGINX v1.19.8 でデプロイするときは、ビルド時に環境変数 `BP_NGINX_VERSION` を指定します。
pack コマンドの `--env` フラグでも指定できますし、 [project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md) でも指定できます。

{{< code/copyable >}}
BP_NGINX_VERSION="1.19.8"
{{< /code/copyable >}}

#### 非推奨：`buildpack.yml` で指定する

<!-- Specifying the NGINX version through `buildpack.yml` configuration will be
deprecated in NGINX Server Buildpack v1.0.0.  To migrate from using
`buildpack.yml` please set the `$BP_NGINX_VERSION` environment variable. -->
NGINX Server Buildpack の v1.0.0 から、`buildpack.yml` でバージョンを指定するのは非推奨になりました。
環境変数 `$BP_NGINX_VERSION` を使用してください。

### コンテナイメージを実行するときに NGINX Server を起動する

<!-- Include an `nginx.conf` file in your application's source code. The NGINX Paketo
buildpack will install the NGINX binary _and_ configure it to start when the app
image launches. -->
アプリケーションのソースコードに `nginx.conf` を含めるようにします。
NGINX Server Cloud Native Buildpack は、NGINX の実行可能ファイルをインストールするだけでなく、コンテナイメージを実行するときに起動できるように構成します。

### NGINX Server の設定値を起動時に構成する

<!-- The NGINX buildpack supports data driven templates for nginx config. You can
use templated variables like `{{port}}`, `{{env "FOO"}}` and `{{module "ngx_stream_module"}}` in your `nginx.conf` to use values known at launch time. -->
NGINX Server Cloud Native Buildpack ではテンプレート形式の設定ファイルに対応しています。
`nginx.conf` の中で、`{{port}}` や `{{env "FOO"}}` や `{{module "ngx_stream_module}}` のように、起動時に決定できる値を変数として記述できます。

<!-- A usage example can be found in the [`samples` repository under the `nginx`
directory](https://github.com/paketo-buildpacks/samples/tree/main/nginx). -->
具体的な使い方は [サンプルアプリケーション](https://github.com/paketo-buildpacks/samples/tree/main/nginx) を参照してください。

#### ポート番号

<!-- Use `{{port}}` to dynamically set the port at which the server will accepts requests. At launch time, the buildpack will read the value of `$PORT` to set the value of `{{port}}`. -->
設定ファイルに `{{port}}` と記述すると、Buildpack で作成したコンテナイメージを実行するとき、環境変数 `$PORT` を読み取った値に展開します。
サーバーがリクエストを待ち受けするポート番号を動的に指定する方法として利用できます。

<!-- For example, to set an NGINX server to listen on `$PORT`, use the following in your `nginx.conf` file: -->
具体的には次のように記述できます。

{{< code/copyable >}}
server {
  listen {{port}};
}
{{< /code/copyable >}}

<!-- Then run the built image using the `PORT` variable set as follows: -->
コンテナイメージを実行するときに環境変数で `PORT` を指定するときは次のようにします。

{{< code/copyable >}}
docker run --tty --env PORT=8080 --publish 8080:8080 my-nginx-image
{{< /code/copyable >}}

#### 環境変数

<!-- This is a generic case of the `{{port}}` directive described ealier. To use the
value of any environment variable `$FOOVAR` available at launch time, use the
directive `{{env "FOOVAR"}}` in your `nginx.conf`. -->
`{{port}}` は分かりやすい例でした。
プロセスの起動時に指定された環境変数を参照するときは `{{env "FOOVAR"}}` のように記述します。

<!-- For example, include the following in your `nginx.conf` file to enable or
disable gzipping of responses based on the value of `GZIP_DOWNLOADS`: -->
レスポンスの GZIP 圧縮機能を環境変数 `GZIP_DOWNLOADS` で切り替えられるようにするには、次のように記述します。

{{< code/copyable >}}
gzip {{env "GZIP_DOWNLOADS"}};
{{< /code/copyable >}}

<!-- Then run the built image using the `GZIP_DOWNLOADS` variable set as follows: -->
コンテナイメージを実行するときに環境変数で `GZIP_DOWNLOADS` を指定するときは次のようにします。

{{< code/copyable >}}
docker run --tty --env PORT=8080 --env GZIP_DOWNLOADS=off --publish 8080:8080 my-nginx-image
{{< /code/copyable >}}

### NGINX Server の動的モジュールを起動時に構成する

<!-- You can use templates to set the path to a dynamic module using the
`load_module` directive. -->
`load_module` に指定する動的モジュールのパスも、テンプレート変数で指定できます。

<!-- To load a user-provided module named `ngx_foo_module`, provide a
`modules/ngx_foo_module.so` file in your app directory and add the following
to the top of your `nginx.conf` file: -->
アプリケーションディレクトリに `modules/ngx_foo_module.so` というモジュールがあるときは、`nginx.conf` を次のように記述します。

{{< code/copyable >}}
{{module "ngx_foo_module"}}
{{< /code/copyable >}}

<!-- To load a buildpack-provided module like `ngx_stream_module`, add the
following to the top of your `nginx.conf` file. You do not need to provide an
`ngx_stream_module.so` file: -->
Buildpack が準備した `ngx_stream_module` というモジュールがあるときは、`nginx.conf` を次のように記述します。

{{< code/copyable >}}
{{module "ngx_stream_module"}}
{{< /code/copyable >}}

<!-- See the [NGINX docs](https://nginx.org/en/docs/beginners_guide.html#conf_structure) for more
information about how to set up an `nginx.conf` file. -->
`nginx.conf` の詳しい内容については [NGINX のドキュメント](https://nginx.org/en/docs/beginners_guide.html#conf_structure) を参照してください。
