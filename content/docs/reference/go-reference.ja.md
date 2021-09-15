---
title: "Go Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: go-reference
    name: "Go Buildpack"
---

{{% reference_exec_summary bp_name="Paketo Go Buildpack" bp_repo="https://github.com/paketo-buildpacks/go" howto_docs_path="/docs/howto/go" %}}

## 対応している依存対象

<!-- The Go Paketo Buildpack supports several versions of Go.
For more details on the specific versions supported in a given buildpack
version, see the [release notes][bp/go/releases]. -->
Go Paketo Buildpack は Go 処理系の複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート][bp/go/releases] で確認してください。

## 振る舞い
<!-- The Paketo Go Buildpack is a [composite buildpack][paketo/composite-buildpack] designed to build applications written in Go. -->
Paketo Go Buildpack は Go アプリケーションをビルドするために設計された [合成 Buildpack][paketo/composite-buildpack] です。

### パッケージ管理

<!-- With the Go CNB, there are three options for package management depending on
your application:
* The built-in [Go modules][golang/modules] feature,
* The [Dep][golang/dep] tool
* No package manager
 -->
Go CNB の対応しているアプリケーションのパッケージ管理方法は次のとおりです。

* Go 処理系の [Go モジュール][golang/modules]
* [dep][golang/dep]
* パッケージ管理しない

<!-- Support for each of these package managers is mutually-exclusive. You can find
specific information for each option below. -->
アプリケーションのパッケージ管理方法は二者択一です。
以降の内容は、それぞれの管理方法に関する説明です。

#### Go モジュールによるパッケージ管理

<!-- The buildpack will vendor dependencies using go modules if the app source
code contains a `go.mod` file. During the build phase, the `go-mod-vendor`
[buildpack][bp/go-mod-vendor] checks to see
if the application requires any external modules and if it does, runs the `go
mod vendor` command for your app. The resulting `vendor` directory will exist
in the app's root directory and will contain all packages required for the build. -->
アプリケーションのソースコードに `go.mod` ファイルがあるとき、Buildpack は Go モジュール機能で依存パッケージを外部化します。
ビルドフェーズでは [`go-mod-vendor` Buildpack][bp/go-mod-vendor] が `go mod vendor` コマンドを実行し、依存パッケージを展開した `vendor` ディレクトリを、アプリケーションのルートディレクトリに作成します。

#### dep によるパッケージ管理

<!-- Dep is an alternative option to Go Modules for package management in Go apps. The buildpack will vendor dependencies using `dep` if the app source code
contains a `Gopkg.toml` file. (For more information about this file, see the `dep`
[documentation][golang/dep/gopkg.toml]). There may be an optional `Gopkg.lock` file that outlines specific versions of the dependencies to be packaged. During its build
phase, the `dep-ensure`
[buildpack][bp/dep-ensure] runs the `dep
ensure` command. The resulting `vendor` directory will exist in
the app's root directory and will contain all the packages required for the build. -->
dep は Go モジュールとは異なる Go アプリケーションのパッケージ管理ツールです。
アプリケーションのソースコードに `Gopkg.toml` ファイルがあるとき、Buildpack は `dep` コマンドで依存パッケージを外部化します（詳しくは [dep のドキュメント][golang/dep/gopkg.toml] を参照してください。）。
`Gopkg.lock` ファイルでパッケージごとのバージョン番号を指定する場合もあります。
ビルドフェーズでは [`dep-ensure` Buildpack][bp/dep-ensure] が `dep ensure` コマンドを実行し、依存パッケージを展開した `vendor` ディレクトリを、アプリケーションのルートディレクトリに作成します。


#### パッケージを管理しない

<!-- The buildpack also supports both self-vendored apps and simpler apps that do not
require third-party packages. In this case there is no vendoring step, and the
`go build` command is run on the app source code as it is provided. -->
Buildpack は独自に依存パッケージを外部化したアプリケーションと、依存パッケージを使用しないアプリケーションの両方に対応しています。
どちらの場合でも外部化処理（vendoring）をしないで、その場にあるソースコードを対象に `go build` コマンドを実行します。

### コンパイル
<!-- The buildpack runs `go build` to compile Go source code into executables. By
default, it sets the flag `-buildmode=pie`. If there is a `go.mod` present in
the app's root directory, it also builds with `mod=vendor`. See the Go tool's [documentation][golang/tool-docs] for details about build configuration. -->
Go CNB は `go build` コマンドでソースコードをコンパイルして実行可能ファイルを作成します。
初期設定では `-buildmode=pie` を指定します。
アプリケーションのルートディレクトリに `go.mod` ファイルがある場合は `-mode=vendor` も追加します。
`go build` について詳しくは [Go コマンドのドキュメント][golang/tool-docs] を参照してください。

## Buildpack の設定する環境変数

<!-- The Go CNB sets a few environment variables during the `build` and `launch`
phases of the app lifecycle. The sections below describe each environment
variable and its impact on your app. -->
Go CNB はアプリケーションライフサイクルのビルドフェーズや起動フェーズでいくつかの環境変数を設定します。
このセクションではアプリケーションに影響するであろう環境変数について説明します。

### `GOPATH`

<!-- The `GOPATH` environment variable tells Go where to look for artifacts such as
source code and binaries. The Go CNB takes care of setting the `GOPATH` for
you, depending on your app and which package management option your app uses. -->
環境変数 `GOPATH` は Go 処理系がソースコードやバイナリファイルなどのアーティファクトを探索する場所を教えるために使用します。
Go CNB はアプリケーションの使用するパッケージ管理方法に応じて `GOPATH` を設定します。

* Set by: `go-mod-vendor`, `dep-ensure` and `go-build`
* Phases: `build`
* Value: path to Go workspace

#### Go モジュール

<!-- When using Go modules, the Go CNB sets the `GOPATH` to a cached module layer in
the image so that between builds of the app, the dependencies don't have to be
redownloaded. Essentially, the `GOPATH` is being used to tell the `go mod
vendor` command where to look for dependencies. It's worth noting that in this
case, the `GOPATH` isn't persisted beyond vendoring the dependencies and gets
overwritten by a subsequent buildpack. -->
Go CNB は Go モジュールを使用するとき、アプリケーションをビルドするためのコンテナイメージに追加するキャッシュモジュールレイヤーのパスを環境変数 `GOPATH` へ設定します。
後に続くビルド処理では依存ライブラリをダウンロードする必要がなくなります。
基本的には `go mod vendor` で依存ライブラリを探索するときに `GOPATH` を参照します。
この時点では関係ありませんが、`GOPATH` は依存ライブラリを外部化（vendoring）した後、そのまま永続化され続けるわけではありません。
後に続く Buildpack が上書きする可能性があります。

#### dep

<!-- When using the Dep tool, the Go CNB sets the `GOPATH` to a temporary directory.
The app source code gets copied into the `GOPATH` location so that the `dep
ensure` command knows where to look for the source code, as well as where to
put the `vendor` directory. The `vendor` directory that is created is then
copied to the original source code directory. The `GOPATH` in this case is used
to run `dep ensure`, but does not persist beyond that step. -->
Go CNB は dep を使用するとき、環境変数 `GOPATH` に一時ディレクトリのパスを設定します。
アプリケーションのソースコードを `GOPATH` のパスへコピーするため、 `dep ensure` コマンドはソースコードを探索する場所と、依存ライブラリを格納する `vendor` ディレクトリの場所を知っている状態になります。
一時ディレクトリの下に作成した `vendor` ディレクトリは、元のソースコードディレクトリへコピーします。
環境変数 `GOPATH` は `dep ensure` を実行するためにだけ変更するため、後に続く処理では元に戻ります。

#### ビルド

<!-- The `go-build` buildpack participates in the Go CNB in every case, regardless
of which package management option is used. The `GOPATH` is set to a temporary
directory which includes the app source code and local sub-packages. The
`GOPATH` is utilized in running `go build` to compile your app. -->
Go CNB では、アプリケーションのパッケージ管理方法に依らず、常に `go-build` Buildpack を使用します。
環境変数 `GOPATH` には、アプリケーションのソースコードとローカルサブパッケージを含む一時ディレクトリのパスを設定します。
環境変数 `GOPATH` はアプリケーションをコンパイルする `go build` コマンドが使用します。

### `GOCACHE`

<!-- The `GOCACHE` variable specifies where build outputs are stored for reuse in
subsequent builds. It gets set to a cached layer in  the image by the
`go-build` buildpack, so that it is persisted between builds. -->
環境変数 `GOCACHE` は、後に続くビルド処理で再利用することになるビルド結果の出力先を設定します。
`go-build` Buildpack がビルド用コンテナイメージのキャッシュレイヤーとして作成するため、ビルド処理をしている間は永続的に利用できます。

* Set by: `go-build`
* Phases: `build`
* Value: Go Cache layer path

### `DEPCACHEDIR`

<!-- `DEPCACHEDIR` specifies where upstream dependency source code is stored for use
by the Dep tool. The `dep-ensure` buildpack sets this variable to the path of a
cache layer in the app image. -->
環境変数 `DEPCACHEDIR` は、dep が使用するために上流の依存対象が提供するソースコードの場所を設定します。
コンテナイメージのキャッシュレイヤーに存在するパスを、`dep-ensure` Buildpack が設定します。

* Set by: `dep-ensure`
* Phases: `build`
* Value: Dep Cache layer path

## コンポーネント
<!--
| Name                                   | Required/Optional | Purpose                                               |
|----------------------------------------|-------------------|-------------------------------------------------------|
| [Paketo CA Certificates Buildpack][bp/ca-certs]       | Optional          | Installs custom CA certificates                       |
| [Paketo Go Dist Buildpack][bp/go-dist]               | Required          | Installs the Golang toolchain                         |
| [Paketo Go Mod Vendor Buildpack][bp/go-mod-vendor]         | Optional          | Installs app Go modules                               |
| [Paketo Dep Buildpack][bp/dep]                   | Optional          | Installs `dep`                                        |
| [Paketo Dep Ensure Buildpack][bp/dep-ensure]            | Optional          | Uses `dep` to install app dependencies                |
| [Paketo Go Build Buildpack][bp/go-build]              | Required          | Compiles source code                                  |
| [Paketo Procfile Buildpack][bp/procfile]              | Optional          | Sets a user-specified start command                   |
| [Paketo Environment Variables Buildpack][bp/env-vars] | Optional          | Sets user-specified launch-time environment variables |
| [Paketo Image Labels Buildpack][bp/image-labels]          | Optional          | Adds user-specified labels to app image metadata      |
 -->
| 名前                                                  | 必須/任意 | 用途                                                              |
|-------------------------------------------------------|-----------|-------------------------------------------------------------------|
| [Paketo CA Certificates Buildpack][bp/ca-certs]       | 任意      | 独自の CA certificates をインストールする                         |
| [Paketo Go Dist Buildpack][bp/go-dist]                | 必須      | Go 処理系のツールチェインをインストールする                       |
| [Paketo Go Mod Vendor Buildpack][bp/go-mod-vendor]    | 任意      | Go モジュールでアプリケーションの依存パッケージをインストールする |
| [Paketo Dep Buildpack][bp/dep]                        | 任意      | `dep` をインストールする                                          |
| [Paketo Dep Ensure Buildpack][bp/dep-ensure]          | 任意      | `dep` でアプリケーションの依存パッケージをインストールする        |
| [Paketo Go Build Buildpack][bp/go-build]              | 必須      | ソースコードをコンパイルする                                      |
| [Paketo Procfile Buildpack][bp/procfile]              | 任意      | 起動コマンドを指定する                                            |
| [Paketo Environment Variables Buildpack][bp/env-vars] | 任意      | 起動時の環境変数を指定する                                        |
| [Paketo Image Labels Buildpack][bp/image-labels]      | 任意      | コンテナイメージにラベルを設定する                                |

<!-- References -->
[golang/tool-docs]:https://pkg.go.dev/cmd/go
[golang/modules]:https://github.com/golang/go/wiki/Modules
[golang/dep]:https://github.com/golang/dep
[golang/dep/gopkg.toml]:https://golang.github.io/dep/docs/Gopkg.toml.html

[paketo/composite-buildpack]:{{< ref "docs/concepts/buildpacks#composite-buildpacks" >}}

[bp/ca-certs]:{{< bp_repo "ca-certificates" >}}
[bp/dep]:{{< bp_repo "dep" >}}
[bp/dep-ensure]:{{< bp_repo "dep-ensure" >}}
[bp/env-vars]:{{< bp_repo "environment-variables" >}}
[bp/go/releases]:{{< bp_repo "go" >}}/releases/latest
[bp/go-build]:{{< bp_repo "go-build" >}}
[bp/go-dist]:{{< bp_repo "go-dist" >}}
[bp/go-mod-vendor]:{{< bp_repo "go-mod-vendor" >}}
[bp/image-labels]:{{< bp_repo "image-labels" >}}
[bp/procfile]:{{< bp_repo "procfile" >}}
