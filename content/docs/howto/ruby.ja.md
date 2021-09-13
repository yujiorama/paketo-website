---
title: "Paketo Buildpacks で Ruby アプリケーションをビルドする"
weight: 332
menu:
  main:
    parent: "howto"
    name: "Ruby"
aliases:
  - /docs/buildpacks/language-family-buildpacks/ruby/
---

{{% howto_exec_summary bp_name="Paketo Ruby Buildpack" bp_repo="https://github.com/paketo-buildpacks/ruby" reference_docs_path="/docs/reference/ruby-reference" %}}

## サンプルアプリをビルドする
<!-- To build a sample app locally with this buildpack using the `pack` CLI, run -->
`pack` コマンドを使って、PHP Paketo Buildpack でサンプルアプリをビルドします。

{{< code/copyable >}}
git clone <https://github.com/paketo-buildpacks/samples>
cd samples/ruby/puma
pack build my-app --buildpack gcr.io/paketo-buildpacks/ruby \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- See [samples](https://github.com/paketo-buildpacks/samples/tree/main/ruby/thin)
for how to run the app. -->
アプリケーションの実行方法は [README ファイル](https://github.com/paketo-buildpacks/samples/tree/main/ruby/thin) を参照してください。

<!-- **NOTE: Though the example above uses the Paketo Base builder, this buildpack is
also compatible with the Paketo Full builder.** -->
**注意：この例では Paketo Base ビルダーを使っていますが、Ruby Paketo Buildpack は Paketo Full ビルダーと互換性があります**

## Ruby のインストールするバージョンを指定する

<!-- The Ruby Buildpack allows you to specify a version of Ruby to use during
deployment. This version can be specified via the `BP_MRI_VERSION` environment
variable or a `Gemfile`. When specifying a version of Ruby, you must choose a version that is available
within the buildpack. The supported versions can be found
[here](https://github.com/paketo-buildpacks/mri/releases/latest). -->
Ruby Paketo Buildpack では、デプロイするときに使用する CRuby のバージョンを、環境変数 `BP_MRI_VERSION` や `Gemfile` で指定できます。
指定できるバージョンは、使用する Buildpack の対応しているバージョンだけです。
Ruby Paketo Buildpack の使用できる CRuby のバージョンについては、[Ruby Paketo Buildpack のリリースノート](https://github.com/paketo-buildpacks/mri/releases/latest) を参照してください。

<!-- Please note that setting the Ruby version through a `buildpack.yml` file will
be deprecated in MRI Buildpack v1.0.0. -->
MRI Cloud Native Buildpack v1.0.0 から、`buildpack.yml` で CRuby のバージョンを指定する機能は非推奨になりました。

<!-- The buildpack prioritizes the versions specified in
each possible configuration location with the following precedence, from
highest to lowest: `BP_MRI_VERSION`, `Gemfile`. -->
バージョンを指定する場所の優先順位は `BP_MRI_VERSION, Gemfile` の順になっています。

<!-- Specifying a version of Ruby is not required. In the case that is not
specified, the buildpack will provide the default version, which can be seen in
the [`buildpack.toml`
](https://github.com/paketo-buildpacks/mri/blob/main/buildpack.toml) file. -->
CRuby のバージョンを指定しなかった場合は Buildpack の [`buildpack.toml`](https://github.com/paketo-buildpacks/mri/blob/main/buildpack.toml) に指定された初期値が使われます。

### 環境変数 `BP_MRI_VERSION` を使用する

<!-- To configure the buildpack to use Ruby v2.7.1 when deploying your app, set the
following environment variable at build time, either directly (ex. `pack build
my-app --env BP_MRI_VERSION=2.7.1`) or through a
[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md)
file: -->
アプリケーションをデプロイするときに CRuby v2.7.1 を使うようにするには、`pack build` コマンドのフラグ `--env` で環境変数 `BP_MRI_VERSION` を指定するか、[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md) へ記述します。

{{< code/copyable >}}
BP_MRI_VERSION="2.7.1"
{{< /code/copyable >}}

### Gemfile を使用する

<!-- To configure the buildpack to use Ruby v2.7.1 when deploying your app, include
the values below in your `Gemfile`: -->
アプリケーションをデプロイするときに CRuby v2.7.1 を使うようにするには、`Gemfile` へ次のように記述します。

{{< code/copyable >}}
source 'https://rubygems.org'

ruby '~> 2.7.1'
{{< /code/copyable >}}

### 非推奨：`buildpack.yml` を使用する

<!-- Specifying the Ruby version through `buildpack.yml` configuration will be deprecated in MRI Buildpack v1.0.0.
To migrate from using `buildpack.yml` please set the `$BP_MRI_VERSION` environment variable. -->
MRI Cloud Native Buildpack v1.0.0 から、`buildpack.yml` で CRuby のバージョンを指定する機能は非推奨になりました。
環境変数 `BP_MRI_VERSION` を使用してください。

## Bundler のインストールするバージョンを指定する

<!-- The Ruby Buildpack allows you to specify a version of Bundler to use during
deployment. This version can be specified via the `BP_BUNDLER_VERSION`
environment variable or a `Gemfile.lock` created during dependency vendoring.
When specifying a version of Bundler, you must choose a version that is
available within the buildpack.  The supported versions can be found
[here](https://github.com/paketo-buildpacks/bundler/releases/latest). -->
Ruby Paketo Buildpack ではデプロイするときに使用する Bundler のバージョンを指定できます。
バージョンを指定するには環境変数 `BP_BUNDLER_VERSION` か `Gemfile.lock` を使用します。
指定できるバージョンは、使用する Buildpack の対応しているバージョンだけです。
使用できる Bundler のバージョンは [Bundler Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/bundler/releases/latest) で確認してください。

<!-- Please note that setting the Bundler version through a `buildpack.yml` file
will be deprecated in Bundler Buildpack v1.0.0. -->
Bundler Cloud Native Buildpack v1.0.0 から、`buildpack.yml` で Bundler のバージョンを指定する機能は非推奨になりました。

<!-- The buildpack prioritizes the versions specified in each possible configuration
location with the following precedence, from
highest to lowest: `BP_BUNDLER_VERSION`, `Gemfile.lock`. -->
バージョンを指定する場所の優先順位は `BP_BUNDLER_VERSION, Gemfile.lock` の順になっています。

<!-- Specifying a version of Bundler is not required. In the case that is not
specified, the buildpack will provide the default version, which can be seen in
the [`buildpack.toml`](https://github.com/paketo-buildpacks/bundler/blob/main/buildpack.toml) file. -->
Bundler のバージョンを指定しなかった場合は、Bundler Cloud Native Buildpack の [`buildpack.toml`](https://github.com/paketo-buildpacks/bundler/blob/main/buildpack.toml) に指定された初期値が使われます。

### 環境変数 `BP_BUNDLER_VERSION` を使用する

<!-- To configure the buildpack to use Bundler v2.1.4 when deploying your app, set
the following environment variable at build time, either directly (ex. `pack
build my-app --env BP_BUNDLER_VERSION=2.1.4`) or through a
[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md)
file: -->
Buildpack がアプリケーションをデプロイするときに Bundler v2.1.4 を使うようにするには、`pack build` コマンドのフラグ `--env` で環境変数 `BP_MRI_VERSION` を指定するか、[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md) へ記述します。

{{< code/copyable >}}
BP_BUNDLER_VERSION="2.1.4"
{{< /code/copyable >}}

### `Gemfile.lock` を使用する

<!-- To configure the buildpack to use Bundler v2.1.4 when deploying your app, run
`bundle install` on your application source code using v2.1.4 of Bundler. This
will result in a `Gemfile.lock` that includes the following snippet: -->
アプリケーションをデプロイするときに Bundler v2.1.4 を使うようにするには、Bundler v2.1.4 で `bundle install` を実行して生成された `Gemfile.lock` をソースコードリポジトリに登録します。

{{< code/copyable >}}
BUNDLED WITH
   2.1.4
{{< /code/copyable >}}

### 非推奨：`buildpack.yml` を使用する

<!-- Specifying the Bundler version through `buildpack.yml` configuration will be deprecated in Bundler Buildpack v1.0.0.
To migrate from using `buildpack.yml` please set the `$BP_BUNDLER_VERSION` environment variable. -->
Bundler Cloud Native Buildpack v1.0.0 から、`buildpack.yml` で Bundler のバージョンを指定する機能は非推奨になりました。
環境変数 `BP_BUNDLER_VERSION` を使用してください。

## オフライン環境でアプリケーションをビルドする
<!-- In order to build apps in an offline environment, the app will need to have the
`.gem` files located in the `cache_path`. Bundler will copy the required gems
into this location, typically `vendor/cache` when running the `bundle package`
command. During the `bundle install` process, the buildpack will instruct
Bundler to prefer gems in this cache over those on the RubyGems index by
running `bundle install --local`. -->
オフライン環境でアプリケーションをビルドするには、`cache_path` に必要な `.gem` ファイルを配置しておかなければなりません。
Bundler は `bundle package` コマンドにより、`vendor/cache` のようなディレクトリへ必要な gem ファイルを複製します。
Buildpack は Bundler がオンラインの RubyGems よりキャッシュを優先するよう、`bundle install --local` を実行します。

## Rake タスクを実行するコンテナイメージをビルドする
<!-- The Ruby Buildpack can build images that run a rake task
at launch time. Simply include a valid `Rakefile` in your app source
code. The buildpack will build an image that runs the default rake task
at launch time.
See this Paketo sample [app](https://github.com/paketo-buildpacks/samples/tree/main/ruby/rake)
for a working example. -->
Ruby Paketo Buildpack は起動コマンドとして Rake タスクを実行するコンテナイメージをビルドできます。
ソースコードに `Rakefile` を入れるだけです。
実行するのはデフォルトタスクです。
具体例は [サンプルアプリ](https://github.com/paketo-buildpacks/samples/tree/main/ruby/rake) を参照してください。

### 特定の Rake タスクを実行する
<!-- To configure the app image to run a rake task called `non_default` on launch, use a [Procfile](/docs/reference/configuration/#procfiles) with contents as follows to set the start command: -->
デフォルトタスク以外の Rake タスクを実行したいときは、[Procfile](/docs/reference/configuration/#procfiles) へ次のように起動コマンドを記述します。

{{< code/copyable >}}
web: bundle exec rake non_default
{{< /code/copyable >}}

<!-- To start an app container with a rake task (instead of its default start command), start the app container with  `--entrypoint launcher` and add the desired rake start command at the end: -->
コンテナイメージを実行するときにデフォルトタスク以外の Rake タスクを指定するには、エントリポイントに `launcher` を指定して、実行したい Rake コマンドを指定します。

{{< code/copyable >}}
docker run --entrypoint launcher my-rake-app bundle exec rake non_default
{{< /code/copyable >}}

## Web サーバーの背後で Ruby アプリケーションを実行する
<!-- The Ruby Buildpack can automatically detect that a Ruby app needs to be run with
several common web servers, and will configure the app image accordingly. The buildpack
currently supports the following webservers:
- Passenger
- Puma
- Rackup
- Thin
- Unicorn
 -->
Ruby Paketo Buildpack は Ruby アプリケーションの前面で実行する Web サーバーを自動的に検出し、構成できます。
Buildpack が対応している Web サーバーは次の通りです。

- Passenger
- Puma
- Rackup
- Thin
- Unicorn

<!-- To make the Ruby Buildpack automatically configure your app for a given webserver,
include its gem in your app's `Gemfile`. -->
Buildpack の実行する Web サーバーを指定するには、 `Gemfile` へ記述します。

<!-- For example, to use Rackup, include in your `Gemfile`: -->
Rackup を使用するときは次のように記述します。

{{< code/copyable >}}
gem 'rack'
{{< /code/copyable >}}

## Rails アプリケーションをビルドする
<!-- The Ruby Buildpack supports Rails apps (Rails version >= 5.0) that need
asset precompilation. -->
Ruby Paketo Buildpack はアセットの事前コンパイルが必要な Rails アプリケーションに対応しています。
対応している Rails のバージョンは 5.0 以上です。

<!-- To use this feature of the buildpack,
1. Include an `app/assets` directory in your app source code
1. Add the `rails` gem to your `Gemfile` -->
アセットの事前コンパイル機能を使用するには次のように構成します。
1. アセットファイルを `app/assets` ディレクトリへ配置します
2. `Gemfile` に `rails` gem を追加します
