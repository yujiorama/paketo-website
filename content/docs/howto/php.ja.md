---
title: "Paketo Buildpacks で PHP アプリケーションをビルドする"
weight: 328
menu:
  main:
    parent: "howto"
    name: "PHP"
aliases:
  - /docs/buildpacks/language-family-buildpacks/php/
---

{{% howto_exec_summary bp_name="Paketo PHP Buildpack" bp_repo="https://github.com/paketo-buildpacks/php" reference_docs_path="/docs/reference/php-reference" %}}

## サンプルアプリをビルドする
<!-- To build a sample app locally with this CNB using the `pack` CLI, run -->
`pack` コマンドを使って、PHP Paketo Buildpack でサンプルアプリをビルドします。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/php/webserver
pack build my-app --buildpack gcr.io/paketo-buildpacks/php \
  --builder paketobuildpacks/builder:full
{{< /code/copyable >}}

<!-- See [samples](https://github.com/paketo-buildpacks/samples/tree/main/php/webserver)
for how to run the app. -->
アプリケーションの実行方法は [README ファイル](https://github.com/paketo-buildpacks/samples/tree/main/php/webserver) を参照してください。

<!-- **NOTE: The Paketo Full builder is required because PHP relies on operating
system libraries only present in the Full builder.** -->
**注意：PHP Paketo Buildpack は Paketo Full ビルダーが必要です。Full ビルダーにしか含まれないOSのシステムライブラリを使用するからです。**

## PHP のインストールするバージョンを指定する

<!-- The PHP Dist CNB allows you to specify a version of PHP to use during
deployment. This version can be specified in a number of ways, including
through `buildpack.yml` or `composer.json` files. When specifying a
version of PHP, you must choose a version that is available
within the buildpack. -->
PHP Distribution Cloud Native Buildpack では、デプロイするときに使用する PHP のバージョンを、いろいろな方法で変更できます。
例えば、`buildpack.yml` や `composer.json` で指定できます。
PHP Distribution Cloud Native Buildpack で使用できるバージョンは、 [PHP Distribution Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/php/releases) で確認できます。

<!-- The buildpack prioritizes the versions specified in
each possible configuration location with the following precedence, from
highest to lowest: `buildpack.yml`, `composer.json`.
Buildpack  -->
バージョンを指定する場所の優先順位は `buildpack.yml, composer.json` の順になっています。

### `buildpack.yml` を使用する

<!-- To configure the buildpack to use PHP version when deploying your app,
include the values like below in your `buildpack.yml` file. Any valid semver
constraints are acceptable. -->
Buildpack でアプリケーションをデプロイするときに使用する PHP のバージョンを `buildpack.yml` で指定できます。
バージョン番号は semver の形式で記述します。

{{< code/copyable >}}
---
php:
  version: 7.2.*
{{< /code/copyable >}}

### `composer.json` を使用する

<!-- If your apps use `composer`, you can specify the PHP version your apps use
during deployment by configuring the `require` field in the `composer.json`
file. To configure the buildpack to use PHP v7.1 or greater when deploying your
app, include the values below in your `composer.json` file: -->
[composer](https://getcomposer.org/) を使用するアプリケーションでは、`composer.json` の `require` フィールドに使用する PHP のバージョンを指定できます。
例えば、v7.1 より新しいバージョンの PHP を使用するときは次のように記述します。

{{< code/copyable >}}
{
  "require": {
    "php": ">=7.1"
  }
}
{{< /code/copyable >}}

If your app has a `composer.lock` file, the buildpack will use
the php version defined there.
アプリケーションのルートディレクトリに `composer.lock` があるときは、そのファイルに指定されたバージョンの PHP を使用します。

## Composer を使用する

<!-- The following options are configurable in the app's `buildpack.yml` -->
[composer](https://getcomposer.org/) を使用するアプリケーションでは、`buildpack.yml` で次のような設定ができます。

{{< code/copyable >}}
composer:
  # 使用する composer のバージョン番号を semver の形式で記述します
  version: 1.10.x

  # composer install の引数を記述します
  # default: ["--no-dev"]
  install_options: ["--no-dev"]

  # ベンダーディレクトリ名を指定します。初期値は vendor です
  vendor_directory: vendor

  # composer.json を配置したディレクトリを記述します。初期値はルートディレクトリです
  json_path: composer

  # compose global を実行させるときに指定する引数を記述します
  install_global: ["list", "of", "install", "options"]
{{< /code/copyable >}}

## Web サーバーを指定する

<!-- The PHP buildpack supports the use of 3 different web servers:

 - PHP built-in Web Server
 - Apache HTTP Web Server
 - Nginx Web Server -->
PHP Paketo Buildpack では次の3種類の Web サーバーを選択できるようになっています。

 - PHP の組み込み Web サーバー
 - Apache HTTPd
 - Nginx

<!-- You can configure the webserver using `buildpack.yml` as follows: -->
使用する Web サーバーは `buildpack.yml` で指定できます。

{{< code/copyable >}}
php:
  # Web サーバーの選択肢（初期値は php-server）：php-server, httpd, nginx
  webserver: php-server
{{< /code/copyable >}}

<!--  If you're using `httpd` or `nginx`, a suitable `httpd.conf` or `nginx.conf`
 will be generated for you by the buildpack. -->
`httpd` や `nginx` を使用する場合、Buildpack が `httpd.conf` や `nginx.conf` などの適切な設定ファイルを生成します。

<!-- You can also provide additional configurations like follows: -->
それぞれの設定ファイルが使用する設定項目を指定することもできます。

{{< code/copyable >}}
# buildpack.yml
php:
  # アプリケーションコードを配置するディレクトリ名。初期値は htdocs
  webdirectory: htdocs

  # ライブラリコードを配置するディレクトリ名。初期値は lib
  libdirectory: lib

  # 使用する cli スクリプト名。初期値は空
  script:

  # サーバー管理者の電子メール。初期値は admin@localhost
  serveradmin: admin@localhost
{{< /code/copyable >}}

## 外部化した composer のパッケージ

<!-- If your php app that uses `composer` has a valid `vendor` directory, then
the buildpack will not download those packages. It will instead use the the
packages location in the `vendor` directory. -->
composer を使用するアプリケーションに正しい `vendor` ディレクトリがある場合、Buildpack はそこに含まれるパッケージをダウンロードしません。
`vendor` ディレクトリに配置したパッケージをそのまま使用します。

## 独自の .ini ファイルを使用する

<!-- If you like to configure custom .ini files in addition to the `php.ini`
provided by the buildpack, you can create a directory named `.php.ini.d` at the
root of your app and put your custom ini files there. See
[`PHP_INI_SCAN_DIR`](https://paketo.io/docs/buildpacks/language-family-buildpacks/php/#php_ini_scan_dir)
in the Variables section below. -->
Buildpack の提供する `php.ini` とは別に任意の `.ini` ファイルを使用するときは、アプリケーションのルートディレクトリに作成した `.php.ini.d` ディレクトリへ配置します。
詳しくは [`PHP_INI_SCAN_DIR`](https://paketo.io/docs/buildpacks/language-family-buildpacks/php/#php_ini_scan_dir) のセクション「変数」を参照してください。

## 独自の CA 証明書をインストールする
<!-- PHP buildpack users can provide their own CA certificates and have them
included in the container root truststore at build-time and runtime by
following the instructions outlined in the [CA
Certificates](/docs/reference/configuration/#ca-certificates)
section of our configuration docs. -->
PHP Paketo Buildpack では、ビルド時や実行時のどちらでも、[CA 証明書の構成](/docs/reference/configuration/#ca-certificates) 手順に従って、ユーザーが自分で用意した CA 証明書をコンテナのルートトラストストアへ配置できます。

## Buildpack の設定する起動プロセスを変更する
<!-- PHP buildpack users can set custom start processes for their app image by
following the instructions in the
[Procfiles](/docs/reference/configuration/#procfiles) section
of our configuration docs. -->
PHP Paketo Buildpack では、[Procfiles の導入](/docs/reference/configuration/#procfiles) 手順に従って、アプリケーションのコンテナイメージが起動するプロセスを変更できます。

## アプリケーションを起動するときの環境変数を設定する
<!-- PHP buildpack users can embed launch-time environment variables in their
app image by following the documentation for the [Environment Variables
Buildpack](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md). -->
PHP Paketo Buildpack では、[環境変数の構成](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md) 手順に従って、アプリケーションのコンテナイメージを実行するときの環境変数を変更できます。

## アプリケーションのコンテナイメージにラベルを設定する
<!-- PHP buildpack users can add labels to their app image by following the
instructions in the [Applying Custom
Labels](/docs/reference/configuration/#applying-custom-labels)
section of our configuration docs. -->
PHP Paketo Buildpack では、[ラベルの構成](/docs/reference/configuration/#applying-custom-labels) 手順に従って、アプリケーションのコンテナイメージにラベルを指定できます。
