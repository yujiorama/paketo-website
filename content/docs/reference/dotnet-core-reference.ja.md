---
title: ".NET Core Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: dotnet-core-reference
    name: ".NET Core Buildpack"
---

{{% reference_exec_summary bp_name="Paketo .NET Core Buildpack" bp_repo="https://github.com/paketo-buildpacks/dotnet-core" howto_docs_path="/docs/howto/dotnet-core" %}}

## 対応している依存対象

<!-- The .Net Core Paketo Buildpack supports several versions of the .Net Core Framework.
For more details on the specific versions supported in a given buildpack
version, see the [release notes](https://github.com/paketo-buildpacks/dotnet-core/releases). -->
.Net Core Paketo Buildpack は .Net Core Framework の複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート](https://github.com/paketo-buildpacks/dotnet-core/releases) で確認してください。

### .Net Core Framework のバージョン選択

<!-- When detecting what version of the .NET runtime to install, the .Net Core Buildpack uses the
same version selection policy that Microsoft has put together for .Net Core Framework.
If you would like to know more about the policy please refer to this
[documentation](https://docs.microsoft.com/en-us/dotnet/core/versions/selection)
provided by Microsoft. -->
.Net Core Buildpack はインストールすべき .NET Runtime のバージョンを検出すると、Microsoft が .Net Core Framework に実装したポリシーと同じようにバージョンを選択します。
詳しくは [Microsoft のドキュメント](https://docs.microsoft.com/en-us/dotnet/core/versions/selection) を参照してください。

## アプリケーション種類

<!-- The .Net Core Buildpack supports several types of application source code that
can be built into a container image. Developers can provide raw source code, or
built artifacts like Framework-Dependent Deployments/Executables or
Self-Contained Deployments when building their application. -->
.Net Core Buildpack はさまざまな種類のアプリケーションソースに対応しています。
開発者はソースコードをそのまま使用することもできるし、フレームワーク依存の展開（FDD：Framework-Dependent Deployments）やフレームワーク依存の実行可能ファイル（FDE：Framework-Dependent Executable）、自己完結型の展開（SCD：Self Contained Deployment）としてビルドすることもできます。

### アプリケーションソース

<!-- The .Net Core Build Buildpack is capable of building application source code
into Framework-Dependent Deployments (FDD) or Executables (FDE).  This is
achieved using the `dotnet publish` command. For .Net Core Framework 2.x
applications,
[an FDD is produced](https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli#framework-dependent-deployment)
as the default build artifact, while
[an FDE is produced](https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli#framework-dependent-executable)
when the application source is for .Net Core Framework 3.x. -->
.Net Core Build Buildpack は `dotnet publish` コマンドでソースコードから FDD や FDE を作成できます。
.Net Core Framework 2.x のアプリケーションなら、初期設定では [単独の FDD を生成します](https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli#framework-dependent-deployment)。
.Net Core Framework 3.x のアプリケーションなら、初期設定では [単独の FDE を生成します](https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli#framework-dependent-executable)。

### FDD と FDE

<!-- When building an application that has already been published as a
Framework-Dependent Deployment or Framework-Dependent Executable, the buildpack
will include the required .Net Core Framework dependencies and set the start
command. -->
FDD あるいは FDE として発行済みのアプリケーションをビルドするとき、Buildpack は .Net Core Framework の必要な依存ライブラリを取得し、起動コマンドを設定します。

### 自己完結型の展開

<!-- When building an application as a
[Self-Contained Deployment](https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli#self-contained-deployment) (SCD),
the buildpack will ensure the correct start command will be used to run your
app. No .Net Core Framework dependencies will be included in the built image as
they are already included in the SCD artifact. -->
[SCD](https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli#self-contained-deployment) として発行済みのアプリケーションをビルドするとき、Buildpack は正しい起動コマンドになることを保証します。
必要な依存ライブラリは全て含まれているため、.Net Core Framework の依存ライブラリは取得しません。

## Buildpack の設定する環境変数

### DOTNET_ROOT

<!-- The `DOTNET_ROOT` environment variable specifies the path to the directory where .Net Runtimes and SDKs are installed. -->
環境変数 `DOTNET_ROOT` には .Net Runtime や SDK をインストールしたディレクトリ名を設定します。

* Set by: `dotnet-core-runtime`, `dotnet-core-sdk`, `dotnet-core-aspnet` buildpacks
* Phases: `build` and `launch`
* Value: path to the .Net root directory

### RUNTIME_VERSION

<!-- The `RUNTIME_VERSION` environment variable specifies the version of the .Net Core Runtime installed by the .Net Core Runtime Buildpack. -->
環境変数 `RUNTIME_VERSION` には .Net Core Runtime Buildpack のインストールした .Net Core Runtime のバージョンを設定します。

* Set by: `dotnet-core-runtime`
* Phases: `build`
* Value: installed version of the .Net Core Runtime

### SDK_LOCATION

<!-- The `SDK_LOCATION` environment variable specifies the file path location of the installed .Net Core SDK. -->
環境変数 `SDK_LOCATION` には Buildpack のインストールした .Net core SDK のディレクトリを設定します。

* Set by: `dotnet-core-sdk`
* Phases: `build`
* Value: path to the .Net Core SDK installation

### PATH

<!-- The `PATH` environment variable is modified to enable the `dotnet` CLI to be found during subsequent `build` and `launch` phases. -->
環境変数 `PATH` には後に続く `build` フェーズや `launch` フェーズで `dotnet` コマンドを実行できるよう、`dotnet` コマンドの存在するディレクトリを追加します。

* Set by: `dotnet-core-sdk`
* Phases: `build` and `launch`
* Value: path the directory containing the `dotnet` executable

## 起動プロセス

<!-- The .Net Core Conf Buildpack will ensure that your application image is built
with a valid launch process command. These commands differ slightly depending
upon the type of built artifact produced during the build process. -->
.Net Core Conf Buildpack はコンテナイメージが正しい起動コマンドになっていることを保証します。
起動コマンドはアプリケーションをビルドした形式により少しづつ異なります。

<!-- For more information about which built artifact is produced for a Source
Application, see [this section]({{< relref "#application-types" >}}). -->
どの形式でアプリケーションをビルドするかは [このセクション]({{< relref "#application-types" >}}) を参照してください。

### FDD とアプリケーションソース

<!-- For Framework-Dependent Deployments (FDD), the `dotnet` CLI will be invoked to
start your application. The application will be given configuration to help it
bind to a port inside the container. The default port is 8080, but can be
overridden using the `$PORT` environment variable. -->
FDD 形式のアプリケーションは `dotnet` コマンドで実行できます。
コマンドの引数でアプリケーションの待ち受けアドレスやポート番号を指定できます。
待ち受けポート番号の初期値は 8080 ですが、環境変数 `PORT` で変更できます。

{{< code/copyable >}}
dotnet myapp.dll --urls http://0.0.0.0:${PORT:-8080}
{{< /code/copyable >}}

### SCD と FDE

<!-- For Self-Contained Deployments and Framework-Dependent Executables, the
executable will be invoked directly to start your application. The application
will be given configuration to help it bind to a port inside the container. The
default port is 8080, but can be overridden using the `$PORT` environment
variable. -->
SCD 形式と FDE 形式のアプリケーションは直接実行できます。
コマンドの引数でアプリケーションの待ち受けアドレスやポート番号を指定できます。
待ち受けポート番号の初期値は 8080 ですが、環境変数 `PORT` で変更できます。

{{< code/copyable >}}
./myapp --urls http://0.0.0.0:${PORT:-8080}
{{< /code/copyable >}}
