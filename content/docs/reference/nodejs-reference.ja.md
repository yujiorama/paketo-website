---
title: "Node.js Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: nodejs-reference
    name: "Node.js Buildpack"
---

<!-- This reference documentation offers an in-depth description of the behavior
and configuration options of the
[Paketo Node.js Buildpack](https://github.com/paketo-buildpacks/nodejs).
For explanations of how to use the buildpack for several common use-cases, see
the Node.js How To [documentation](/docs/howto/nodejs). -->
このドキュメントでは [Paketo Node.js Buildpack](https://github.com/paketo-buildpacks/nodejs) の振る舞いや設定項目について詳しく説明します。
一般的な使用例については [Node.js のチュートリアル](/docs/howto/nodejs) を参照してください。

## 対応している依存対象
<!-- The Node.js buildpack supports several versions of Node.js.
For more details on the specific versions supported in a given buildpack
version, see the [release notes](https://github.com/paketo-buildpacks/nodejs/releases). -->
Node.js Buildpack は Node.js ランタイムの複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート](https://github.com/paketo-buildpacks/nodejs/releases) で確認してください。

## Buildpack の設定する環境変数
<!-- The Node.js buildpack sets a number of environment variables during the `build` and
`launch` phases of the app lifecycle. The sections below describe each
environment variable and its impact on your app. -->
Node.js Buildpack はアプリケーションライフサイクルのビルドフェーズや起動フェーズでいくつかの環境変数を設定します。
このセクションではアプリケーションに影響するであろう環境変数について説明します。

### MEMORY_AVAILABLE
<!-- The `MEMORY_AVAILABLE` environment variable reports the total amount of memory
available to the app. The Node.js buildpack calculates this value from the limits
specified by the operating system in
`/sys/fs/cgroup/memory/memory.limit_in_bytes`. -->
環境変数 `MEMORY_AVAILABLE` にはアプリケーションから利用できるメモリサイズの合計値を設定します。
Node.js Buildpack は OS の `/sys/fs/cgroup/memory/memory.limit_in_bytes` を参照して値を計算します。

* Set by: `profile.d`
* Phases: `launch`
* Value: non-negative integer

### NODE_ENV
<!-- The `NODE_ENV` environment variable specifies the environment in which the app
runs. -->
環境変数 `NODE_ENV` にはアプリケーションを実行するときの環境種類を設定します。

* Set by: `node-engine` buildpack
* Phases: `build`
* Value: production

### NODE_HOME
<!-- The `NODE_HOME` environment variable sets the path to the `node` installation. -->
環境変数 `NODE_HOME` には Node.js をインストールしたディレクトリのパスを設定します。

* Set by: `node-engine` buildpack
* Phases: `build`
* Value: path to the `node` installation

### NODE_VERBOSE
<!-- The `NODE_VERBOSE` environment variable adjusts the amount of logging output
from NPM during installs. -->
環境変数 `NODE_VERBOSE` には NPM が依存ぱっけー所をインストールするときに出力するログを調整する値を設定します。

* Set by: `node-engine` buildpack
* Phases: `build`
* Value: false

### NPM\_CONFIG\_LOGLEVEL
<!-- The `NPM_CONFIG_LOGLEVEL` environment variable adjusts the level of logging NPM
uses. -->
環境変数 `NPM_CONFIG_LOGLEVEL` には NPM のログ出力レベルを調整する値を設定します。

* Set by: `npm-install` buildpack
* Phases: `build`
* Value: "error"

### NPM\_CONFIG\_PRODUCTION
<!-- The `NPM_CONFIG_PRODUCTION` environment variable installs only production
dependencies if NPM install is used. -->
環境変数 `NPM_CONFIG_PRODUCTION` には NPM が production の依存ライブラリだけをインストールするかどうかを制御する値を設定します。

* Set by: `npm-install` buildpack
* Phases: `build`
* Value: false

### PATH
<!-- The `node_modules/.bin` directory is appended onto the `PATH` environment variable -->
環境変数 `PATH` には `node_modules/.bin` を追加します。

* Set by: `yarn-install` or `npm-install` buildpacks
* Phases: `build`
* Value: path to the `node_modules/.bin` directory

## NPM によるパッケージ管理
<!-- Many Node.js apps require a number of third-party libraries to perform common
tasks and behaviors. NPM is an option for managing these third-party
dependencies that the Node.js buildpack fully supports. Including a `package.json`
file in your app source code triggers the NPM installation process. The sections
below describe the NPM installation process run by the buildpack. -->
多くの Node.js アプリケーションで、一般的なタスクや振る舞いを実現するためにサードパーティのライブラリを使用しています。
NPM は、Node.js Buildpack が完全に対応している、サードパーティのライブラリを管理する方法の1つです。
Buildpack は、アプリケーションのソースコードに `package.json` があれば、NPM によるサードパーティライブラリのインストールを試みます。
このセクションでは Buildpack が NPM によるインストールをどのように実行するのか説明します。

### NPM によるパッケージのインストール
<!-- NPM supports several distinct methods for installing your package dependencies.
Specifically, the Node.js buildpack runs either the [`npm install`](https://docs.npmjs.com/cli-commands/install), [`npm rebuild`](https://docs.npmjs.com/cli-commands/rebuild.html), or [`npm ci`](https://docs.npmjs.com/cli-commands/ci.html) commands to build your app
with the right set of dependencies. When deciding which installation process to
use, the Node.js buildpack consults your app source code, looking for the presence of
specific files or directories. The installation process used also determines
how the Node.js buildpack will reuse layers when rebuilding your app. -->
NPM には依存パッケージをインストールするためのさまざまな方法が用意されています。
Node.js Buildpack は [`npm install`](https://docs.npmjs.com/cli-commands/install) や [`npm rebuild`](https://docs.npmjs.com/cli-commands/rebuild.html) や [`npm ci`](https://docs.npmjs.com/cli-commands/ci.html) を実行します。
実行するコマンドは、ソースコードに含まれる特定のファイルやディレクトリから判断します。
また、実行するコマンドに基づいて、アプリケーションレイヤーを再利用するか、それとも再作成するかを判断します。

<!-- The table below shows the process the Node.js buildpack uses to determine an
installation process for NPM packages. When a combination of the files and
directories listed in the table below are present in your app source code,
the Node.js buildpack uses an installation process that ensures the
correct third-party dependencies are installed during the build process. -->
次の表は Node.js Buildpack が NPM で依存パッケージをインストールする方法を判断する条件です。
アプリケーションのソースコードに存在するファイルやディレクトリに応じて、必要な依存パッケージを正確に再現できるインストール方法を判断します。

| `package-lock.json` | `node_modules` | `npm-cache` | Command |
| ------------------- | -------------- | ----------- | ------- |
| X | X | X | `npm install` |
| X | X | ✓ | `npm install` |
| X | ✓ | X | `npm rebuild` |
| X | ✓ | ✓ | `npm rebuild` |
| ✓ | X | X | `npm ci` |
| ✓ | X | ✓ | `npm ci` |
| ✓ | ✓ | X | `npm rebuild` |
| ✓ | ✓ | ✓ | `npm ci` |

<!-- The following sections give more information about the files listed in the
table above, including how to generate them, if desired. -->
このセクションの残りでは、前の表に並んでいるファイルについて説明し、意図したとおりの結果にする方法を説明します。

#### package-lock.json
<!-- The `package-lock.json` file is generated by running `npm install`.  For
more information, see
[npm-package-lock.json](https://docs.npmjs.com/files/package-lock.json) in the
NPM documentation. -->
`package-lock.json` は `npm install` コマンドを実行すると生成されるファイルです。
詳しくは [Node のドキュメント](https://docs.npmjs.com/files/package-lock.json) を参照してください。

#### node_modules
<!-- The `node_modules` directory contains vendored copies of all the packages
installed by the `npm install` process. For more information, see the [Node
Modules](https://docs.npmjs.com/files/folders.html#node-modules) section of the
_npm-folders_ topic in the NPM documentation. -->

`node_modules` ディレクトリには、 `npm install` でインストールした全てのパッケージの複製が存在します。
詳しくは [Node Mudules](https://docs.npmjs.com/files/folders.html#node-modules) で _rpm-folders_ のセクションを参照してください。

#### npm-cache
<!-- The `npm-cache` directory contains a content-addressable cache that stores all
HTTP-request- and package-related data. Additionally, including a cache ensures
that the app can be built entirely offline. -->
`npm-cache` ディレクトリには、パッケージに関する全ての情報を、コンテンツをアドレス指定できるデータとして格納します。
加えて、キャッシュが利用できる場合、アプリケーションをビルドするために必要な全ての依存パッケージをオフライン環境で利用できることになります。

<!-- To populate an `npm-cache` directory: -->
`npm-cache` ディレクトリへデータを追加するには次のように実行します。
<!--
1. Navigate to your source code directory.
1. Run:
{{< code/copyable >}}
npm ci --cache npm-cache
{{< /code/copyable >}}
 -->
{{< code/copyable >}}
cd <your source code directory>
npm ci --cache npm-cache
{{< /code/copyable >}}

<!-- For more information about the NPM cache, see
[npm-cache](https://docs.npmjs.com/cli/cache) in the NPM documentation. -->
NPM キャッシュについて詳しくは [NPM キャッシュのドキュメント](https://docs.npmjs.com/cli/cache) を参照してください。

### Node モジュールレイヤーを再利用するか判断する
<!-- To improve build times for apps, the Node.js buildpack has a method for reusing the build
results from previous builds. When the CNB determines that a portion of the
build process can be reused from a previous build, the CNB uses the previous
result. Each installation process uses a different method for determining
whether the CNB can reuse a previous build result. -->
アプリケーションのビルド時間を短縮するため、Node.js Buildpack には先に実行したビルドの結果を再利用する方法があります。
依存パッケージのインストール方法に応じて、Buildpack が先に実行したビルドの結果を再利用できるか判断する方法は異なります。

<!-- For `npm install`, the CNB never reuses a `node_modules` directory from previous builds. -->
`npm install` の場合、先に実行したビルドで作成した `node_modules` ディレクトリは再利用できません。

<!-- For `npm rebuild`, the CNB can reuse a `node_modules` directory from a previous
build if the included `node_modules` directory in the app source code has not
changed since the prior build. -->
`npm rebuild` の場合、アプリケーションのソースコードに含まれる `node_modules` ディレクトリが、先に実行したビルドで変更されていなければ再利用できます。

<!-- For `npm ci`, the CNB can reuse a `node_modules` directory from a previous
build if the `package-lock.json` file included in the app source code has not
changed since the prior build. -->
`npm ci` の場合、アプリケーションのソースコードに含まれる `package-lock.json` が、先に実行したビルドで変更されていなければ、`node_modules` ディレクトリを再利用できます。

## Yarn によるパッケージ管理
<!-- Many Node.js apps require a number of third-party libraries to perform common
tasks and behaviors. Yarn is an alternative option to NPM for managing these
third-party dependencies. Including `package.json` and `yarn.lock` files in
your app source code triggers the Yarn installation process. -->
多くの Node.js アプリケーションで、一般的なタスクや振る舞いを実現するためにサードパーティのライブラリを使用しています。
Yarn は、Node.js Buildpack が対応している、サードパーティのライブラリを管理する方法の1つです。
Buildpack は、アプリケーションのソースコードに `package.json` と `yarn.lock` があれば、Yarn によるサードパーティライブラリのインストールを試みます。
このセクションでは Buildpack が Yarn によるインストールをどのように実行するのか説明します。

### Yarn によるパッケージのインストール
<!-- The Node.js buildpack runs `yarn install` and `yarn check` to ensure that third-party
dependencies are properly installed.
The `yarn.lock` file contains a fully resolved set of package dependencies that
Yarn manages. For more information, see
[yarn.lock](https://yarnpkg.com/lang/en/docs/yarn-lock/) in the Yarn
documentation. -->
Node.js Buildpack は `yarn install` あるいは `yarn.check` でサードパーティの依存パッケージがインストールできていることを保証します。
`yarn.lock` ファイルには Yarn の管理する依存パッケージに関する完全な情報が格納されています。
詳しくは [Yarn のドキュメント](https://yarnpkg.com/lang/en/docs/yarn-lock/) を参照してください。

### Yarn でパッケージ管理する場合の起動コマンド
<!-- As part of the build process, the Node.js buildpack determines a start command for
your app. The start command differs depending on which package management
tooling the Node.js buildpack uses. If the Node.js buildpack uses `yarn` to install
packages, the start command is `yarn start`. -->
Node.js Buildpack はビルド処理の一部としてコンテナイメージの起動コマンドを生成します。
起動コマンドはアプリケーションのパッケージ管理の方法により異なります。
Node.js Buildpack が `yarn` を使用するときは、起動コマンドに `yarn start` を設定します。

## 起動コマンドを決定する
<!-- As part of the build process, the Node.js buildpack determines a start command for
your app. The start command differs depending on which package management
tooling the Node.js buildpack uses. If the Node.js buildpack uses `npm` or `yarn` to
install packages, the start command is generated from the contents of
`package.json`. -->
Node.js Buildpack はビルド処理の一部としてコンテナイメージの起動コマンドを生成します。
起動コマンドはアプリケーションのパッケージ管理の方法により異なります。
Node.js Buildpack が `npm` か `yarn` を使用するときは、`package.json` の内容から生成した起動コマンドを設定します。

## パッケージ管理を使わないプロジェクト
<!-- The Node.js buildpack also supports simple apps that do not require third-party packages. -->
Node.js Buildpack はサードパーティパッケージを使わない、単純なアプリケーションにも対応しています。

### 起動コマンド
<!-- If no package manager is detected, the Node.js buildpack will look for four generic entrypoint JavaScript
files, in this order:
- `server.js`
- `app.js`
- `main.js`
- `index.js`
 -->
Node.js Buildpack はパッケージ管理ツールを使っていないアプリケーションについて、一般的なエントリポイントとなるファイルを探索します。

- `server.js`
- `app.js`
- `main.js`
- `index.js`

<!-- When the buildpack finds one of these, it will consider that the app's entrypoint and set the
app image's start command to `node <entrypoint filename>`. -->
Buildpack はこれらのファイルを発見すると、起動コマンドに `node <entrypoint filename>` を設定します。

##  部品表（BOM：Bill of Materials）
<!-- The Node.js buildpack supports the full [bill of materials]({{< ref "docs/concepts/bom" >}}) (BOM). For Node applications, the BOM contains: -->
Node.js Buildpack は [BOM]({{< ref "docs/concepts/bom" >}}) に完全対応しています。
Node.js アプリケーションの BOM には次のような内容が含まれます。

<!-- * [buildpack entries]({{< ref "docs/concepts/bom#buildpack-entries" >}}) for dependencies such as `node engine`,
* [language module entries]({{< ref "docs/concepts/bom#language-module-entries" >}}) for each `node module` that is either vendored in or installed via NPM or Yarn. -->
* [Buildpack 要素]({{< ref "docs/concepts/bom#buildpack-entries" >}}) - `node engine` - Node.js のランタイムに関する情報です
* [言語モジュール要素]({{< ref "docs/concepts/bom#language-module-entries" >}}) - `node module` - `node_modules` に展開した依存パッケージに関する情報です

<!-- The `node module` BOM entries provide a full picture of the packages on the final app image. The `purl` field is especially helpful to locate where on the NPM Registry a module came from. Check out the [Access the Bill of Materials guide]({{< ref "docs/howto/bom" >}}) for more information about how to retrieve the BOM for your Node.js app image. -->
`node module` 要素は最終的なコンテナイメージに含まれる依存パッケージの全体像を提供します。
`purl` フィールドを見れば、どの NPM レジストリから取得できるのか分かります。
詳しくは [部品表（Bill of Materials）へアクセスする]({{< ref "docs/howto/bom" >}}) を参照してください。
