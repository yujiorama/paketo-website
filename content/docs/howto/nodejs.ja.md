---
title: "Paketo Buildpacks で Node.js アプリケーションをビルドする"
weight: 324
menu:
  main:
    parent: "howto"
    name: "Node.js"
aliases:
  - /docs/buildpacks/language-family-buildpacks/nodejs/
---

<!-- This documentation explains how to use the [Paketo Node.js Buildpack](https://github.com/paketo-buildpacks/nodejs)
to build applications for several common use-cases. For more in-depth
description of the buildpack's behavior and configuration see the Node.js
Buildpack Reference [documentation](/docs/reference/nodejs-reference). -->
このドキュメントでは [Paketo Node.js Buildpack](https://github.com/paketo-buildpacks/nodejs) を使用してアプリケーションのコンテナイメージを作成する方法を説明します。
Buildpack の振る舞いや設定方法を詳しく知りたいときは [リファレンス](/docs/reference/nodejs-reference) を参照してください。

## サンプルアプリをビルドする
<!-- To build a sample app locally with this buildpack using the pack CLI, run -->
ローカルPCで `pack` コマンドを実行して、サンプルアプリのコンテナイメージを Buildpack で作成するには次のように実行します。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/nodejs/npm
pack build my-app --buildpack gcr.io/paketo-buildpacks/nodejs \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- See [samples](https://github.com/paketo-buildpacks/samples/tree/main/nodejs/npm)
for how to run the app. -->
アプリケーションの実行方法は [README ファイル](https://github.com/paketo-buildpacks/samples/tree/main/nodejs/npm) を参照してください。

<!-- **NOTE: Though the example above uses the Paketo Base builder, this buildpack is
also compatible with the Paketo Full builder. The Paketo Full builder is
required if your app utilizes common C libraries.** -->
**注意：この例では Paketo Base ビルダーを使っていますが、Paketo Node.js Buildpack は Paketo Full ビルダーと互換性があります。アプリケーションが一般的なCライブラリ関数を呼び出している場合は Full ビルダーが必要です。**

## Node Engine のインストールするバージョンを指定する
<!-- The Node.js buildpack allows you to specify a version of Node.js to use during
deployment. This version can be specified in a number of ways, including
through the `BP_NODE_VERSION` environment variable, a `package.json`, `.nvmrc` or `.node-version` files. When specifying a version of the Node.js engine, you must choose a version that is available
within the buildpack. The supported versions can be found on the Paketo Node Engine
component buildpack's [releases page](https://github.com/paketo-buildpacks/node-engine/releases/latest). -->
Paketo Node.js Buildpack では、デプロイするときに使用する Node.js のバージョンを指定できるようになっています。
バージョンを指定する方法はいろいろあります。
環境変数 `BP_NODE_VERSION` や、コードベースに配置したファイル（`package.json, .nvmrc, .node-version`）で指定できるのです。
指定できるバージョンは、使用する Buildpack の対応しているバージョンだけです。
Paketo Node.js Buildpack の使用できる Node.js Engine のバージョンについては、[Paketo Node.js Buildpack のリリースノート](https://github.com/paketo-buildpacks/node-engine/releases/latest) を参照してください。

<!-- The buildpack prioritizes the versions specified in
each possible configuration location with the following precedence, from
highest to lowest: `BP_NODE_VERSION`, `package.json`, `.nvmrc` and `.node-version`. -->
バージョンを指定する場所の優先順位は `BP_NODE_VERSION, package.json, .nvmrc, .node-version` の順になっています。

### 環境変数 `BP_NODE_VERSION` でバージョンを指定する

<!-- To configure the buildpack to use Node.js v12.12.0 when deploying your app, set the
following environment variable at build time, either directly (ex. `pack build
my-app --env BP_NODE_VERSION=12.12.0`) or through a
[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md)
file: -->
例えば、アプリケーションをデプロイするときに Node.js v12.12.0 を使用するには、環境変数 `BP_NODE_VERSION` へ次のように指定します。
あるいは、`pack build` コマンドの `--env` フラグに `BP_NODE_VERSION=12.12.0` を指定することもできますし、[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md) で指定することもできます。

{{< code/copyable >}}
BP_NODE_VERSION="12.12.0"
{{< /code/copyable >}}

### `package.json` でバージョンを指定する
<!-- If your apps use `npm` or `yarn`, you can specify the Node.js version your apps use
during deployment by configuring the `engines` field in the `package.json`
file. To configure the buildpack to use Node.js v12.12.0 when deploying your
app, include the values below in your `package.json` file: -->
アプリケーションを `npm` や `yarn` で管理しているなら、`package.json` の `engines` フィールドへ使用する Node.js のバージョンを指定できます。

{{< code/copyable >}}
{
  "engines": {
    "node": "12.12.0"
  }
}
{{< /code/copyable >}}

<!-- For more information about the `engines` configuration option in the
`package.json` file, see the
[engines](https://docs.npmjs.com/files/package.json#engines) section of the
_npm-package.json_ topic in the NPM documentation. -->
`engines` フィールドについて詳しくは NPM の package.json に関するドキュメントの [engines](https://docs.npmjs.com/files/package.json#engines) セクションを参照してください。

### `.nvmrc` でバージョンを指定する
<!-- Node Version Manager is a common option for managing the Node.js version an app
uses. To specify the Node.js version your apps use during deployment, include a
`.nvmrc` file with the version number. For more information about the contents
of a `.nvmrc` file, see [.nvmrc](https://github.com/nvm-sh/nvm#nvmrc) in the
Node Version Manager repository on GitHub. -->
[NVM(Node Version Manager)](https://github.com/nvm-sh/nvm) はアプリケーションの使用する Node.js のバージョンを管理するためのツールです。
アプリケーションをデプロイするとき、`.nvmrc` に記述したバージョンの Node.js を使用できます。

### `.node-version` でバージョンを指定する
<!-- `.node-version` is another common option that is compatible with Node.js version managers
such as `asdf` and `nodenv`. You can use a `.node-version` file to set the Node.js version
that your apps use during deployment, according to one of the following formats: -->
`.node-version` は [asdf](https://github.com/asdf-vm/asdf) や [nodenv](https://github.com/nodenv/nodenv) で Node.js のバージョンを指定するために使用するファイルです。
アプリケーションをデプロイするとき、`.node-version` に記述したバージョンの Node.js を使用できます。
バージョン番号はいろいろな形式で記述できます。

{{< code/copyable >}}
12.12.0
{{< /code/copyable >}}

<!-- OR -->

{{< code/copyable >}}
v12.12.0
{{< /code/copyable >}}

<!-- OR -->

{{< code/copyable >}}
12.12
{{< /code/copyable >}}

#### 非推奨：`buildpack.yml` を使用する

<!-- Specifying the Node version through `buildpack.yml` configuration will be deprecated in Node Engine Buildpack v1.0.0.
To migrate from using `buildpack.yml` please set the `$BP_NODE_VERSION` environment variable. -->
Paketo Node.js Buildpack の v1.0.0 から、`buildpack.yml` で Node.js のバージョンを指定するのは非推奨になりました。
環境変数 `BP_NODE_VERSION` で指定するようにしてください。

## ヒープメモリ最適化機能を有効にする
<!-- Node.js limits the total size of all objects on the heap. Enabling the
`optimize-memory` feature sets this value to three-quarters of the total memory
available in the container. For example, if your app is limited to 1&nbsp;GB
at run time, the heap of your Node.js app is limited to 768&nbsp;MB. -->
Node.js はヒープメモリに格納できるオブジェクトサイズの合計値を制限しています。
ヒープメモリ最適化機能を有効にすると、コンテナから利用できるメモリサイズの4分の3までヒープメモリとして使用できるようになります。
例えば、あなたのアプリケーションが実行時に使用できるメモリサイズが 1GB なら、Node.js の使用できるヒープメモリサイズは 768 MB になります。

<!-- You can enable memory optimization through the `BP_NODE_OPTIMIZE_MEMORY` environment variable. -->
環境変数 `BP_NODE_OPTIMIZE_MEMORY` を指定すると有効化できます。

### 環境変数 `BP_NODE_OPTIMIZE_MEMORY` を指定する
<!-- To enable memory optimization through the `BP_NODE_OPTIMIZE_MEMORY` environment
variable, set it to `true`. -->
ヒープメモリ最適化機能を有効にするには環境変数 `BP_NODE_OPTIMIZE_MEMORY` に `true` を指定します。

{{< code/copyable >}}
pack build my-app \
  --buildpack gcr.io/paketo-buildpacks/nodejs \
  --env BP_NODE_OPTIMIZE_MEMORY=true
{{< /code/copyable >}}

#### 非推奨：`buildpack.yml` を使用する

Enabling memory optimization through your `buildpack.yml` file will be
deprecated in Node Engine Buildpack v1.0.0. To migrate from using
`buildpack.yml` please set the `BP_NODE_OPTIMIZE_MEMORY` environment variable
mentioned above.
Paketo Node.js Buildpack の v1.0.0 から、`buildpack.yml` でヒープメモリ最適化機能を有効にするのは非推奨になりました。
環境変数 `BP_NODE_OPTIMIZE_MEMORY` で指定するようにしてください。

## サブディレクトリのソースコードからアプリケーションのコンテナイメージをビルドする

<!-- To specify a subdirectory to be used as the root of the app, please use the
`BP_NODE_PROJECT_PATH` environment variable at build time either directly or
through a [`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor).
This could be useful if your app is a part of a monorepo. -->
環境変数 `BP_NODE_PROJECT_PATH` でサブディレクトリを指定すると、アプリケーションのルートディレクトリとして使用できるようになります。
また、[`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor) でも指定できます。
アプリケーションをモノリポで管理しているときに便利です。

<!-- For example, if your project has the following structure: -->
アプリケーションが次のようなディレクトリ構成になっている場合を考えてみましょう。

```
.
├── go-app
│  ├── go.mod
│  └── main.go
└── node-app
    ├── file.js
    ├── index.js
    └── package.json
```

<!-- you could then set the following at build time. -->
この場合、環境変数 `BP_NODE_PROJECT_PATH` で次のようにサブディレクトリを指定します。

```
BP_NODE_PROJECT_PATH=node-app
```

## ビルドフェーズで実行するスクリプトを指定する
<!-- To specify a script or series of scripts to run during build phase, please use the
`BP_NODE_RUN_SCRIPTS` environment variable at build time either directly or through a 
[`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor). This 
could be useful if your app uses a framework like Angular, React, or Vue where you need to run 
scripts to build your app into a production state. -->
環境変数 `BP_NODE_RUN_SCRIPTS` へ `package.json` の `scripts` フィールドに記述したスクリプトを指定すると、ビルドフェーズで実行できます。
また、[`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor) でも指定できます。
Angular や React や Vue など、ビルド処理が必要なフレームワークを使っているときに便利です。

<!-- For example, if your project's `package.json` has the following scripts: -->
例えば、`package.json` に次のようなスクリプトが記述されているとしましょう。

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "lint": "eslint src/**/*.js src/**/*.jsx"
  }
}
```

<!-- and your environment variable was set: -->
環境変数 `BP_NODE_RUN_SCRIPTS` へ次のように指定するとします。

```
$BP_NODE_RUN_SCRIPTS=lint,build
```

<!-- then the `lint` and then `build` scripts would be run via npm or yarn, during build phase. Note that the
value for `BP_NODE_RUN_SCRIPTS` must be a comma separated list of script names. -->
そうすると、ビルドフェーズで `npm` あるいは `yarn` により、`lint` と `build` スクリプトを順番に実行できます。
指定できるのは、スクリプト名をカンマ区切りで連結した文字列です。

## NPM でアプリケーションをビルドする
<!-- The Node.js buildpack can detect automatically if an app requires `npm`. -->
Paketo Node.js Buildpack はアプリケーションが `npm` を使用しているか自動的に検出します。

### ビルド時に実行する NPM の設定を変更する
<!-- The Node.js buildpack respects native configuration options for NPM. If you would
like to learn more about NPM native configuration please check the NPM
Configuration [documentation](https://docs.npmjs.com/cli/v6/using-npm/config)
and the `.npmrc`
[documentation](https://docs.npmjs.com/cli/v6/configuring-npm/npmrc). -->
Paketo Node.js Buildpack は NPM の本来の設定を尊重するようになっています。
詳しくは [NPM のドキュメント](https://docs.npmjs.com/cli/v6/using-npm/config) と [.npmrc のドキュメント](https://docs.npmjs.com/cli/v6/configuring-npm/npmrc) を参照してください。

## Yarn でアプリケーションをビルドする
<!-- The Node.js buildpack can detect automatically if an app requires `yarn`, by
checking for a `yarn.lock` file. -->
Paketo Node.js Buildpack はアプリケーションが `yarn` を使用しているか自動的に検出します（`yarn.lock` ファイルの有無で判断します）。

### ビルド時に実行する Yarn の設定を変更する
<!-- The Node.js buildpack respects native configuration options for Yarn. If you would
like to learn more about Yarn configuration using `.yarnrc` please visit [the
Yarn documentation](https://classic.yarnpkg.com/en/docs/yarnrc). -->
Paketo Node.js Buildpack は Yarn の本来の設定を尊重するようになっています。
詳しくは [.yarnrc のドキュメント](https://classic.yarnpkg.com/en/docs/yarnrc) を参照してください。

## `node-gyp` でネイティブ拡張をコンパイルする
<!-- If your app requires compilation of native extensions using `node-gyp`, the Node.js buildpack requires that
you use the Paketo Full Builder. This is because `node-gyp` requires `python` which is excluded from the
the Paketo Base Builder's stack, and the module may require other shared objects. -->
アプリケーションが `node-gyp` でコンパイルするネイティブ拡張を使っている場合、Paketo Full ビルダーで実行しなければなりません。
Paketo Base ビルダーには、 `node-gyp` を実行するために必要な `python` や、さまざまな共有ライブラリが含まれていないからです。

### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, specify the latest Paketo Full Builder at build time
with the `--builder` flag. -->
`pack` コマンドで Paketo Full ビルダーを指定するときは次のように実行します。

{{< code/copyable >}}
pack build my-app --builder paketobuildpacks/builder:full
{{< /code/copyable >}}

## パッケージ管理ツールを使わないアプリケーションをビルドする

<!-- The Node.js buildpack supports building apps without `node_modules` or a `package.json`.
It will detect this type of app automatically, by looking for one of these four files in
the root of your application directory:
- `server.js`
- `app.js`
- `main.js`
- `index.js` -->
Paketo Node.js Buildpack は `node_modules` や `package.json` を使用しないアプリケーションをビルドできます。
次のいずれかのファイルが、アプリケーションのルートディレクトリに存在する場合は、そのように判断します。

- `server.js`
- `app.js`
- `main.js`
- `index.js`

### エントリポイントを変更する
<!-- If your app's entrypoint file is not one of the four files named above,
you can specify a different file name (or path) by setting the `BP_LAUNCHPOINT`
environment variable at build time. -->
アプリケーションのエントリポイントが前に列挙したファイルとは違うファイルの場合、環境変数 `BP_LAUNCHPOINT` でファイル名を指定できます。

#### 環境変数 `BP_LAUNCHPOINT` を使用する
<!-- `BP_LAUNCHPOINT` can be set as follows: -->
環境変数 `BP_LAUNCHPOINT` でエントリポイントとなるファイル名を指定できます。

{{< code/copyable >}}
BP_LAUNCHPOINT="./src/launchpoint.js"
{{< /code/copyable >}}

<!-- The image produced by the build will run `node src/launchpoint.js`
as its start command. -->
作成したコンテナイメージは、起動コマンドとして `node src/launchpoint.js` を実行します。

## 独自の CA 証明書をインストールする
## Install a Custom CA Certificate
<!-- Node.js Buildpack users can provide their own CA certificates and have them
included in the container root truststore at build-time and runtime by
following the instructions outlined in the [CA
Certificates](docs/reference/configuration/#ca-certificates)
section of our configuration docs. -->
Paketo Node.js Buildpack では、ビルド時や実行時のどちらでも、[CA 証明書の構成](/docs/reference/configuration/#ca-certificates) 手順に従って、ユーザーが自分で用意した CA 証明書をコンテナのルートトラストストアへ配置できます。

## Buildpack の設定する起動プロセスを変更する
<!-- Node.js Buildpack users can set custom start processes for their app image by
following the instructions in the
[Procfiles](https://paketo.io/docs/reference/configuration/#procfiles) section
of our configuration docs. -->
Paketo Node.js Buildpack では、[Procfiles の導入](/docs/reference/configuration/#procfiles) 手順に従って、アプリケーションのコンテナイメージが起動するプロセスを変更できます。

## アプリケーションを起動するときの環境変数を設定する
<!-- Node.js Buildpack users can embed launch-time environment variables in their
app image by following the documentation for the [Environment Variables
Buildpack](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md). -->
Paketo Node.js Buildpack では、[環境変数の構成](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md) 手順に従って、アプリケーションのコンテナイメージを実行するときの環境変数を変更できます。

## アプリケーションのコンテナイメージにラベルを設定する
<!-- Node.js Buildpack users can add labels to their app image by following the
instructions in the [Applying Custom
Labels](/docs/reference/configuration/#applying-custom-labels)
section of our configuration docs. -->
Paketo Node.js Buildpack では、[ラベルの構成](/docs/reference/configuration/#applying-custom-labels) 手順に従って、アプリケーションのコンテナイメージにラベルを指定できます。

