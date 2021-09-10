---
title: "Paketo Buildpacks で Go アプリケーションをビルドする"
weight: 304
menu:
  main:
    parent: "howto"
    name: "Go"
aliases:
  - /docs/buildpacks/language-family-buildpacks/go
---

{{% howto_exec_summary bp_name="Paketo Go Buildpack" bp_repo="https://github.com/paketo-buildpacks/go" reference_docs_path="/docs/reference/go-reference" %}}

## サンプルアプリをビルドする

<!-- You can quickly build a sample Go app into a runnable OCI image on your
local machine with Paketo buildpacks. -->
Paketo Buildpacks を使用すると、ローカル PC で Go 言語のサンプルアプリケーションの OCI イメージを簡単にビルドできます。

<!-- *Prerequisites*
- docker CLI
- pack CLI -->
*必要なもの*
- docker コマンド
- pack コマンド

<!-- 1. Clone the Paketo samples and navigate to a Go sample app. -->
1. サンプルプロジェクトのリポジトリをローカル PC に clone して、Go 言語のアプリケーションのソースコードが入っているディレクトリへ移動します。
{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/go/mod
{{< /code/copyable >}}

<!-- 1. Use the pack CLI with the Paketo Go Buildpack to build the sample app. -->
1. 移動したディレクトリで pack コマンドを実行し、Paketo Go Buildpack でアプリケーションのコンテナイメージを作成します。
{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- 1. Run the app using instructions found in its `README`. -->
1. 同じディレクトリにある `README` を参照して、アプリケーションを実行します。

<!-- *Note: Though the example above uses the Paketo Base Builder, this buildpack is
also compatible with the Paketo Full Builder and Paketo Tiny Builder.* -->
**注意：この例では Paketo Base ビルダーを使っていますが、Paketo Go Buildpack は Paketo Full ビルダー、および、Paketo Tiny ビルダーと互換性があります**

## 自動的に検出する Go 言語のバージョンを上書きする

<!-- The Paketo Go buildpack will attempt to automatically detect the correct version
of Go to install based on the version in your app's `go.mod`. It is possible to
override this version by setting the `BP_GO_VERSION` environment variable at build time. -->
Paketo Go Buildpack は、 `go.mod` の内容から Go 処理系の正確なバージョンを自動的に検出します。
ビルド時に使用する Go 処理系のバージョンは、環境変数 `BP_GO_VERSION` で変更できます。

<!-- `BP_GO_VERSION` can be set to any valid semver version or version constraint (e.g. `1.14.1`, `1.14.*`).
For the versions available in the buildpack, see the buildpack's [releases page][bp/releases]. Specifying a version of Go is not required. In the case that is not specified,
the buildpack will provide the default version, which can be seen in the
`buildpack.toml` file. -->
環境変数 `BP_GO_VERSION` には semver 形式のバージョン番号を指定できます（例えば `1.14.1` や `1.14.*` など）。
Paketo Go Buildpack で使用できる Go 処理系のバージョンは、 [Go Distribution Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/go-dist/releases) で確認できます。
Go 処理系のバージョンは明示的に指定しなくても構いません。
その場合、Go Distribution Cloud Native Buildpack に同梱された `buildpack.toml` で定義されたバージョンを使用します。

<!-- #### With pack and a Command-Line Flag -->
#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `BP_GO_VERSION` at build time with the `--env` flag. -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_GO_VERSION` を指定できます。
{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_GO_VERSION="1.14.1"
{{< /code/copyable >}}

<!-- #### With pack and a `project.toml` -->
#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `BP_GO_VERSION` at build time. -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_GO_VERSION` を指定できます。
{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_GO_VERSION"
    value="1.41.1"
{{< /code/copyable >}}

<!-- The pack CLI will automatically detect the project file at build time. -->

#### 非推奨：pack コマンドから `buildpack.yml` を参照する
<!-- Please note that setting the Go version through a buildpack.yml file will be
deprecated in Go Dist Buildpack v1.0.0. -->
Go Distribution Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` で Go 処理系のバージョンを指定するのは非推奨になりました。

## `go build` コマンドを変更する

<!-- The Paketo Go buildpack compiles Go source code with the `go build` command, with certain opinionated flags by default. (See reference [documentation]({{< ref "docs/reference/go-reference" >}}) for information about the default flagset.) It is possible to override or add to these defaults by setting the `BP_GO_BUILD_FLAGS` and `BP_GO_BUILD_LDFLAGS`
environment variables at build time. -->
Paketo Go Buildpack は、ソースコードをコンパイルするとき、`go build` にいくつかのフラグを指定します（詳しくは [Go 言語のリファレンス]({{< ref "docs/reference/go-reference" >}}) を参照してください）。
環境変数 `BP_GO_BUILD_FLAGS` や `BP_GO_BUILD_LDFLAGS` で `go build` に指定するフラグを変更できます。

### `go build` に `-ldflags` を指定する
<!-- The Paketo Go buildpack has a dedicated environment variable for setting the value of `-ldflags`. -->
Paketo Go Buildpack では、`-ldflags` を指定するための独立した環境変数を使用できます。

#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `BP_GO_BUILD_LDFLAGS` at build time with the `--env` flag. For example, to add `-ldflags="-X main.variable=some-value"` to the build flagset, set the environment variable as follows: -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_GO_BUILD_LDFLAGS` を指定できます。
例えば、`-ldflags="-X main.variable=some-value"` を指定するには次のように実行します。
{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_GO_BUILD_LDFLAGS="-X main.variable=some-value"
{{< /code/copyable >}}

#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `BP_GO_BUILD_LDFLAGS` at build time. For example, to add `-ldflags="-X main.variable=some-value"` to the build flagset, set the environment variable as follows: -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_GO_BUILD_LDFLAGS` を指定できます。
例えば、`-ldflags="-X main.variable=some-value"` を指定するには次のように記述します。
{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_GO_BUILD_LDFLAGS"
    value="-X main.variable=some-value"
{{< /code/copyable >}}

<!-- The pack CLI will automatically detect the project file at build time. -->

### `go-build` に `-ldflags` 以外のフラグを指定する
<!-- Setting `BP_GO_BUILD_FLAGS` will add to the Paketo Go buildpack's default flagset. Any value that you set for a given flag will override the value set by the buildpack. See reference [documentation](/docs/reference/go-reference) for information about default configuration. -->
Paketo Go Buildpack では、環境変数 `BP_GO_BUILD_FLAGS` でビルドフラグの初期値を変更できます。
詳しくは [Go 言語のリファレンス]({{< ref "docs/reference/go-reference" >}}) を参照してください。

#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `BP_GO_BUILD_FLAGS` at build time with the `--env` flag. For example, to add `-buildmode=default -tags=paketo` to the build flagset, set the environment variable as follows: -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_GO_BUILD_FLAGS` を指定できます。
例えば、`-buildmode=default -tags=paketo` を指定するには次のように実行します。
{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_GO_BUILD_FLAGS="-buildmode=default -tags=paketo"
{{< /code/copyable >}}

#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `BP_GO_BUILD_FLAGS` at build time. For example, to add `-buildmode=default -tags=paketo` to the build flagset, set the environment variable as follows: -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_GO_BUILD_FLAGS` を指定できます。
例えば、`-buildmode=default -tags=paketo` を指定するには次のように記述します。
{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_GO_BUILD_FLAGS"
    value="-buildmode=default -tags=paketo"
{{< /code/copyable >}}

#### 非推奨：pack コマンドから `buildpack.yml` を参照する
<!-- Specifying the Go Build flags through buildpack.yml configuration will be
deprecated in Go Build Buildpack v1.0.0. To migrate from using buildpack.yml
please set the `$BP_GO_BUILD_FLAGS` environment variable. -->
Go Build Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` でフラグセットを指定するのは非推奨になりました。
環境変数 `$BP_GO_BUILD_FLAGS` を使用してください。

<!-- The pack CLI will automatically detect the project file at build time. -->

## デフォルト以外のパッケージをビルドする
<!-- The Paketo Go Buildpack will compile the package in the app's root directory
by default. It is possible to build a non-default package (or packages) by
setting the `BP_GO_TARGETS` environment variable at build time. -->
初期設定の Paketo Go Buildpack は、コードベースのルートディレクトリのパッケージをコンパイルします。
他の（複数の）パッケージをコンパイルするには、環境変数 `BP_GO_TARGETS` を指定します。

<!-- The following examples will build the `second` package in an app source directory with the structure: -->
次のようなディレクトリ構成で、`second` というサブディレクトリにビルドしたいパッケージが存在する場合を考えてみましょう。

```bash
app-directory
├── first
│  └── main.go
├── second
│  └── main.go
└── third
    └── main.go
```

#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `BP_GO_TARGETS` at build time with the `--env` flag.  -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_GO_TARGETS` を指定できます。

{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_GO_TARGETS="./second"
{{< /code/copyable >}}

#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `BP_GO_TARGETS` at build time. -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_GO_TARGETS` を指定できます。

{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_GO_TARGETS"
    value="./second"
{{< /code/copyable >}}

<!-- The pack CLI will automatically detect the project file at build time. -->

#### 非推奨：pack コマンドから `buildpack.yml` を参照する
<!-- Specifying the Go Build targets through buildpack.yml configuration will be
deprecated in Go Build Buildpack v1.0.0. To migrate from using buildpack.yml
please set the `$BP_GO_TARGETS` environment variable. -->
Go Build Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` でコンパイルターゲットを指定するのは非推奨になりました。
環境変数 `$BP_GO_TARGETS` を使用してください。

### 複数のパッケージをビルドしてコンテナイメージにする

<!-- The `BP_GO_TARGETS` evironment variable can accept a colon-delimited list of
target packages. Each binary will be set as a [launch process][cnb/launch-process] of the same name in
the app image. The following examples will build _both_ the `first` and `second` packages in the same multi-package app directory as [above]({{< relref "#build-non-default-packages" >}}). -->
環境変数 `BP_GO_TARGETS` には、コロンを区切り文字とする対象パッケージのリストを指定できます。
それぞれのパッケージに対応する実行可能ファイルは、コンテナイメージの [起動プロセス][cnb/launch-process] に指定できます。
[前に説明したディレクトリ構造]({{< relref "#build-non-default-packages" >}}) において、`first` と `second` を _両方とも_ 指定する場合を説明します。

#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `BP_GO_TARGETS` at build time with the `--env` flag.  -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_GO_TARGETS` を指定できます。

{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_GO_TARGETS="./first:./second"
{{< /code/copyable >}}

#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `BP_GO_TARGETS` at build time. -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_GO_TARGETS` を指定できます。

{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_GO_TARGETS"
    value="./first:./second"
{{< /code/copyable >}}

#### 非推奨：pack コマンドから `buildpack.yml` を参照する
<!-- Specifying the Go Build targets through buildpack.yml configuration will be
deprecated in Go Build Buildpack v1.0.0. To migrate from using buildpack.yml
please set the `$BP_GO_TARGETS` environment variable. -->
Go Build Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` でコンパイルターゲットを指定するのは非推奨になりました。
環境変数 `$BP_GO_TARGETS` を使用してください。

## 同じコードベースのサブパッケージをインポートするアプリケーションをビルドする

<!-- If you are building a `$GOPATH` application that imports its own sub-packages,
you will need to specify the import paths for those sub-packages. The Paketo Go Buildpack supports setting these import paths with the `$BP_GO_BUILD_IMPORT_PATH`
environment variable at build time. -->
`$GOPATH` を使用して、同じコードベースのサブパッケージをインポートするアプリケーションをビルドするときは、それぞれのサブパッケージのためのインポートパスを指定しなければなりません。
Paketo Go Buildpack では、環境変数 `BP_GO_BUILD_IMPORT_PATH` でインポートパスを変更できます。

<!-- The following examples will configure the buildpack to build an app whose
directory structure looks like: -->
次のようなディレクトリ構成のコードベースについて考えてみましょう。

```bash
app-directory
├── handlers
│  └── details.go
└── main.go
```

<!-- and whose `main.go` imports: -->
`main.go` の import ブロックには次のように記述されていることにします。

```go
import (
	"github.com/app-developer/app-directory/handlers"
)
```

#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `$BP_GO_BUILD_IMPORT_PATH` at build time with the `--env` flag.  -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_GO_BUILD_IMPORT_PATH` を指定できます。

{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_GO_BUILD_IMPORT_PATH="github.com/app-developer/app-directory"
{{< /code/copyable >}}

#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `$BP_GO_BUILD_IMPORT_PATH` at build time. -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_GO_BUILD_IMPORT_PATH` を指定できます。

{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_GO_BUILD_IMPORT_PATH"
    value="github.com/app-developer/app-directory"
{{< /code/copyable >}}

#### 非推奨：pack コマンドから `buildpack.yml` を参照する
<!-- Specifying the Go Build import path through buildpack.yml configuration will be
deprecated in Go Build Buildpack v1.0.0. To migrate from using buildpack.yml
please set the `$BP_GO_BUILD_IMPORT_PATH` environment variable. -->
Go Build Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` でコンパイルターゲットを指定するのは非推奨になりました。
環境変数 `$BP_GO_BUILD_IMPORT_PATH` を使用してください。

## ビルドしたソースコードファイルを削除せずに残す

<!-- By default, the Paketo Go Buildpack deletes the contents of your app source directory (except for built artifacts). To preserve certain static assets that
are needed in the app image, you can set the `BP_KEEP_FILES` environment
variable at build time. -->
初期設定の Paketo Go Buildpack は、ビルド成果物を除いて、ソース―コードディレクトリから全てのファイルを削除します。
コンテナイメージに静的アセットファイルを残しておきたいときは、環境変数 `BP_KEEP_FILES` を設定してください。

<!-- The following examples configure the buildpack to prevent the `assets/` and
`public/` directories from being removed in the app image. -->
例えば、サブディレクトリ `assets/` と `public/` のファイルをコンテナイメージに残すには次のようにします。

#### `pack build` コマンドのフラグで指定する
<!-- When building with the pack CLI, set `$BP_KEEP_FILES` at build time with the
`--env` flag.  -->
`pack` コマンドでビルドするとき、 `--env` フラグで `BP_KEEP_FILES` を指定できます。

{{< code/copyable >}}
pack build my-app --buildpack gcr.io/paketo-buildpacks/go \
  --env BP_KEEP_FILES="assets/*:public/*"
{{< /code/copyable >}}

#### `project.toml` で指定する
<!-- When building with the pack CLI, create a [project.toml][cnb/project-file] file in your app directory that sets `$BP_KEEP_FILES` at build time. -->
`pack` コマンドでビルドするとき、アプリケーションディレクトリに配置した [project.toml][cnb/project-file] で `BP_KEEP_FILES` を指定できます。

{{< code/copyable >}}
# project.toml
[ build ]
  [[ build.env ]]
    name="BP_KEEP_FILES"
    value="assets/*:public/*"
{{< /code/copyable >}}

## 独自の CA 証明書をインストールする
<!-- Go buildpack users can provide their own CA certificates and have them
included in the container root truststore at build-time and runtime by
following the instructions outlined in the [CA
Certificates]({{< ref "docs/reference/configuration#ca-certificates" >}})
section of our configuration docs. -->
Paketo Go Buildpack では、ビルド時や実行時のどちらでも、[CA 証明書の構成](/docs/reference/configuration/#ca-certificates) 手順に従って、ユーザーが自分で用意した CA 証明書をコンテナのルートトラストストアへ配置できます。

## Buildpack の設定する起動プロセスを変更する
<!-- Go buildpack users can set custom start processes for their app image by
following the instructions in the
[Procfiles]({{< ref "docs/reference/configuration#procfiles" >}}) section
of our configuration docs. -->
Paketo Go Buildpack では、[Procfiles の導入](/docs/reference/configuration/#procfiles) 手順に従って、アプリケーションのコンテナイメージが起動するプロセスを変更できます。

## アプリケーションを起動するときの環境変数を設定する
<!-- Go buildpack users can embed launch-time environment variables in their
app image by following the documentation for the [Environment Variables
Buildpack](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md). -->
Paketo Go Buildpack では、[環境変数の構成](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md) 手順に従って、アプリケーションのコンテナイメージを実行するときの環境変数を変更できます。

## アプリケーションのコンテナイメージにラベルを設定する
<!-- Go buildpack users can add labels to their app image by following the
instructions in the [Applying Custom
Labels]({{< ref "docs/reference/configuration#applying-custom-labels" >}})
section of our configuration docs. -->
Paketo Go Buildpack では、[ラベルの構成](/docs/reference/configuration/#applying-custom-labels) 手順に従って、アプリケーションのコンテナイメージにラベルを指定できます。

<!-- References -->
[cnb/launch-process]:https://buildpacks.io/docs/app-developer-guide/run-an-app/

[cnb/project-file]:https://buildpacks.io/docs/app-developer-guide/using-project-descriptor

[bp/releases]:https://github.com/paketo-buildpacks/go/releases/latest