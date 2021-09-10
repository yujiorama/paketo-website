---
title: "Paketo Buildpacks で .NET Core アプリケーションをビルドする"
weight: 302
menu:
  main:
    parent: "howto"
    name: ".NET Core"
aliases:
  - /docs/buildpacks/language-family-buildpacks/dotnet-core/
---

{{% howto_exec_summary bp_name="Paketo .NET Core Buildpack" bp_repo="https://github.com/paketo-buildpacks/dotnet-core" reference_docs_path="/docs/reference/dotnet-core-reference" %}}

## サンプルアプリをビルドする
<!-- To build your app locally with the buildpack using the `pack` CLI, run -->
`pack` コマンドを使って、Paketo .NET Core Buildpack でサンプルアプリをビルドします。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/dotnet-core/aspnet
pack build my-app --buildpack gcr.io/paketo-buildpacks/dotnet-core \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- See
[samples](https://github.com/paketo-buildpacks/samples/tree/main/dotnet-core/aspnet)
for how to run the app. -->
アプリケーションの実行方法は [README ファイル](https://github.com/paketo-buildpacks/samples/tree/main/dotnet-core/aspnet) を参照してください。

<!-- **NOTE: Though the example above uses the Paketo Base builder, this buildpack is
also compatible with the Paketo Full builder.** -->
**注意：この例では Paketo Base ビルダーを使っていますが、Paketo .NET Core Buildpack は Paketo Full ビルダーと互換性があります**

## .NET Core Runtime と ASP.NET のインストールするバージョンを指定する

<!-- The .Net Core Runtime and .Net Core ASP.Net Buildpacks allow you to specify a
version of the .Net Core Runtime and ASP.Net to use during deployment. This
version can be specified in several ways including through a
`runtimeconfig.json`, MSBuild Project file, or build-time environment
variables. When specifying a version of the .Net Core Runtime and ASP.Net, you
must choose a version that is available within these buildpacks. These versions
can be found in the [.Net Core Runtime release
notes](https://github.com/paketo-buildpacks/dotnet-core-runtime/releases) and
[.Net Core ASP.Net release
notes](https://github.com/paketo-buildpacks/dotnet-core-aspnet/releases). -->
Dotnet Core Runtime Cloud Native Buildpack と ASPNet Cloud Native Buildpack では、デプロイ時に使用する .NET Core Runtime と ASP.NET のバージョンを指定できるようになっています。
`runtimeconig.json` や、MSBuild のプロジェクトファイル、ビルド環境変数等で指定できます。
指定できるバージョンは、[Dotnet Core Runtime Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/dotnet-core-runtime/releases) や [ASPNet Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/dotnet-core-aspnet/releases) で確認できる、それぞれの Buildpack が使用できるバージョンだけです。

<!-- .Net Core ASP.Net will only be included in the build process if your
application declares its Runtime Framework as either `Microsoft.AspNetCore.App`
or `Microsoft.AspNetCore.All`. -->
アプリケーションの使用するランタイムフレームワークとして `Microsoft.AspNetCore.App` や `Microsoft.AspNetCore.All` を宣言している場合、指定したバージョンの .NET Core Runtime と ASP.NET はビルドプロセスでのみ使用します。

### 環境変数 `BP_DOTNET_FRAMEWORK_VERSION` を指定する

<!-- To configure the buildpack to use a certain version of the .Net Core Runtime
and ASP.Net when deploying your app, set the `$BP_DOTNET_FRAMEWORK` environment
variable at build time, either by passing a flag to the
[platform](https://buildpacks.io/docs/concepts/components/platform/) or by
adding it to your `project.toml`. See the Cloud Native Buildpacks
[documentation](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor/)
to learn more about `project.toml` files. -->
デプロイ時に使用する .NET Core Runtime と ASP.NET のバージョンは、環境変数 `$BP_DOTNET_FRAMEWORK` で指定できます。
また、 [プラットフォーム](https://buildpacks.io/docs/concepts/components/platform/) にフラグとして指定することもできますし、プロジェクトの `project.toml` で指定することもできます。
`project.toml` の内容について詳しく知りたいときは、Cloud Native Buildpacs の [ドキュメント](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor/) を参照してください。

<!-- **With a `pack build` flag** -->
**`pack build` コマンドのフラグで指定する**
{{< code/copyable >}}
pack build myapp --env BP_DOTNET_FRAMEWORK_VERSION=5.0.4
{{< /code/copyable >}}

<!-- **In a `project.toml` file** -->
**`project.toml` で指定する**
{{< code/copyable >}}
[[ build.env ]]
  name = 'BP_DOTNET_FRAMEWORK_VERSION'
  value = '5.0.4'
{{< /code/copyable >}}

<!-- **Note**: If you specify a particular version using the above environment
variable, the buildpack **will not** run runtime version roll-forward logic. To
learn more about roll-forward logic, see the [Microsoft .Net Runtime
documentation](https://docs.microsoft.com/en-us/dotnet/core/versions/selection#framework-dependent-apps-roll-forward). -->
**注意**：環境変数等でバージョンを指定しても、Buildpack はロールフォワードロジックを適用しません。ロールフォワードロジックについて詳しくは [Microsoft のドキュメント](https://docs.microsoft.com/en-us/dotnet/core/versions/selection#framework-dependent-apps-roll-forward) を参照してください。

### runtimeconfig.json を使用する

<!-- If you are using a
[`runtimeconfig.json`](https://docs.microsoft.com/en-us/dotnet/core/run-time-config/)
file, you can specify the .Net Core Runtime version within that file. To
configure the buildpack to use .Net Core Runtime v2.1.14 when deploying your
app, include the values below in your `runtimeconfig.json` file: -->
あなたのプロジェクトで [runtimeconfig.json](https://docs.microsoft.com/en-us/dotnet/core/run-time-config/) を使っているなら、そのファイル内で .NET Core Runtime のバージョンを指定できます。
アプリケーションをデプロイするとき、Buildpack に v2.1.14 の .NET Core Runtime を使わせるには次のように記述します。

{{< code/copyable >}}
{
  "runtimeOptions": {
    "framework": {
      "version": "2.1.14"
    }
  }
}
{{< /code/copyable >}}

### プロジェクトファイルを使用する

<!-- If you are using a Project file (eg. `*.csproj`, `*.fsproj`, or `*.vbproj`), you can specify
the .Net Core Runtime version within that file. To configure the buildpack to
use .Net Core Runtime v2.1.14 when deploying your app, include the values below
in your Project file: -->
あなたのプロジェクトが `.csproj` や `.fsproj` や `.vbproj` などのプロジェクトファイル
を使っているなら、そのファイル内で .NET Core Runtime のバージョンを指定できます。
アプリケーションをデプロイするとき、Buildpack に v2.1.14 の .NET Core Runtime を使わせるには次のように記述します。

{{< code/copyable >}}
<Project>
  <PropertyGroup>
    <RuntimeFrameworkVersion>2.1.14</RuntimeFrameworkVersion>
  </PropertyGroup>
</Project>
{{< /code/copyable >}}

<!-- Alternatively, for applications that do not rely upon a specific .Net Core
Runtime patch version, you can specify the Target Framework and the buildpack
will choose the appropriate .Net Core Runtime version. To configure the
buildpack to use a .Net Core Runtime version in the 2.1 .Net Core Target Framework
when deploying your app, include the values below in your Project file: -->
一方、アプリケーションが .NET Core Runtime のパッチバージョンに依存しない場合は、対象フレームワーク（TargetFramework）を指定して、Buildpack に .NET Core Runtime の適切なバージョンを選択させることができます。
Buildpack に .NET Core Runtime 2.1 を使わせるには、対象フレームワークを次のように記述します。

{{< code/copyable >}}
<Project>
  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>
</Project>
{{< /code/copyable >}}

<!-- For more details about specifying a .Net Core version using a Project file,
please review the [Microsoft
documentation](https://docs.microsoft.com/en-us/dotnet/core/versions/selection). -->
プロジェクトファイルに .NET Core Runtime のバージョンを指定する方法について、詳しくは [Microsoft のドキュメント](https://docs.microsoft.com/en-us/dotnet/core/versions/selection) を参照してください。

### 非推奨：buildpack.yml を使用する

<!-- Specifying the .Net Core Framework version through `buildpack.yml`
configuration will be deprecated in .Net Core Runtime and .Net Core ASPNET
Buildpacks v1.0.0.  To migrate from using `buildpack.yml`, please set the
`BP_DOTNET_FRAMEWORK_VERSION` environment variable. -->
Dotnet Core Runtime Cloud Native Buildpack と ASPNet Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` で .NET Core Runtime のバージョンを指定するのは非推奨になりました。
代わりに、環境変数 `BP_DOTNET_FRAMEWORK_VERSION` を指定するようにしてください。

## 特定バージョンの .NET SDK をインストールする

<!-- By default, the .Net Core SDK Buildpack installs the latest available patch
version of the SDK that is compatible with the installed .Net Core runtime.
The available SDK versions for each buildpack release can be found in the
[release notes](https://github.com/paketo-buildpacks/dotnet-core-sdk/releases). -->
初期設定の Dotnet Core SDK Buildpack は、すでにインストールされている .NET Core Runtime と互換性のあるバージョンの SDK の、最新のパッチバージョンをインストールします。
Buildpack のバージョンに対応する利用可能な SDK のバージョンは [Dotnet Core SDK Buildpack のリリースノート](https://github.com/paketo-buildpacks/dotnet-core-sdk/releases) を参照してください。

<!-- However, the .Net Core SDK version can be explicitly set by specifying a
version in a `buildpack.yml` file. -->
ですが、.NET Core SDK のバージョンを `buildpack.yml` へ明示的に指定することもできます。

### 非推奨：buildpack.yml を使用する

<!-- Specifying the .Net Core SDK version through `buildpack.yml` configuration will
be deprecated in .Net Core SDK Buildpack v1.0.0. -->
Dotnet Core SDK Buildpack v1.0.0 から、`buildpack.yml` で .NET Core SDK のバージョンを指定するのは非推奨になりました。

<!-- Because versions of the .NET Core runtime and .NET Core SDK dependencies are so
tightly coupled, most users should instead use the
`BP_DOTNET_FRAMEWORK_VERSION` environment variable to specify which version of
the .NET Core runtime that the .NET Core Runtime Buildpack should install. The
.Net Core SDK buildpack will automatically install an SDK version that is
compatible with the selected .NET Core runtime version. -->
.NET Core Runtime と .NET Core SDK は密結合しているし、ほとんどのユーザーは Dotnet Core Runtime Cloud Native Buildpack のインストールする .NET Core Runtime のバージョンを環境変数 `BP_DOTNET_FRAMEWORK_VERSION` で指定したほうがいいからです。
そうすれば、Dotnet Core SDK Buildpack はインストールされた .NET Core Runtime と互換性のあるバージョンの SDK を自動的にインストールします。

## サブディレクトリのソースコードからアプリケーションのコンテナイメージをビルドする

<!-- By default, the .Net Core Build Buildpack will consider the root directory of
your codebase to be the project directory. This directory should contain a C#,
F#, or Visual Basic Project file. If your project directory is not located at
the root of your source code you will need to set a custom project path. -->
初期設定の .NET Core paketo Buildpack はコードベースのルートディレクトリをプロジェクトディレクトリとして扱うようになっています。
C# や F# や Visual Basic のソースコードが配置されたものとして扱うのです。
ソースコードをサブディレクトリに配置している場合は、明示的にプロジェクトパスを指定しなければなりません。

### 環境変数 `BP_DOTNET_PROJECT_PATH` を指定する

<!-- You can specify a project path by setting the `$BP_DOTNET_PROJECT_PATH`
environment variable at build time, either by passing a flag to the
[platform](https://buildpacks.io/docs/concepts/components/platform/) or by
adding it to your `project.toml`. See the Cloud Native Buildpacks
[documentation](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor/)
to learn more about `project.toml` files. -->
ビルド時に使用するプロジェクトパスは、環境変数 `$BP_DOTNET_PROJECT_PATH` で指定できます。
また、 [プラットフォーム](https://buildpacks.io/docs/concepts/components/platform/) にフラグとして指定することもできますし、プロジェクトの `project.toml` で指定することもできます。
`project.toml` の内容について詳しく知りたいときは、Cloud Native Buildpacs の [ドキュメント](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor/) を参照してください。

<!-- **With a `pack build` flag** -->
**`pack build` コマンドのフラグで指定する**
{{< code/copyable >}}
pack build my-app --env BP_DOTNET_PROJECT_PATH=./src/my-app
{{< /code/copyable >}}

<!-- **In a `project.toml` file** -->
**`project.toml` で指定する**
{{< code/copyable >}}
[[ build.env ]]
  name = 'BP_DOTNET_PROJECT_PATH'
  value = './src/my-app'
{{< /code/copyable >}}

<!-- See the [Cloud Native Buildpacks
documentation](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor/)
to learn more about `project.toml` files. -->

### 非推奨：buildpack.yml を使用する

<!-- Specifying the project path through `buildpack.yml` configuration will be
deprecated in Dotnet Publish Buildpack v1.0.0 & Dotnet Execute Buildpack
v1.0.0. To migrate from using `buildpack.yml`, please set the
`$BP_DOTNET_PROJECT_PATH` environment variable. -->
Dotnet Publish Cloud Native Buildpack と Dotnet Execute Cloud Native Buildpack の v1.0.0 から、`buildpack.yml` でプロジェクトパスを指定するのは非推奨になりました。
代わりに、環境変数 `BP_DOTNET_PROJECT_PATH` を指定するようにしてください。

## 独自の CA 証明書をインストールする
<!-- .Net Core Buildpack users can provide their own CA certificates and have them
included in the container root truststore at build-time and runtime by
following the instructions outlined in the [CA
Certificates](/docs/reference/configuration/#ca-certificates)
section of our configuration docs. -->
Paketo .NET Core Buildpack では、ビルド時や実行時のどちらでも、[CA 証明書の構成](/docs/reference/configuration/#ca-certificates) 手順に従って、ユーザーが自分で用意した CA 証明書をコンテナのルートトラストストアへ配置できます。

## Buildpack の設定する起動プロセスを変更する
<!-- .Net Core Buildpack users can set custom start processes for their app image by
following the instructions in the
[Procfiles](/docs/reference/configuration/#procfiles) section
of our configuration docs. -->
Paketo .NET Core Buildpack では、[Procfiles の導入](/docs/reference/configuration/#procfiles) 手順に従って、アプリケーションのコンテナイメージが起動するプロセスを変更できます。

## アプリケーションを起動するときの環境変数を設定する
<!-- .Net Core Buildpack users can embed launch-time environment variables in their
app image by following the documentation for the [Environment Variables
Buildpack](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md). -->
Paketo .NET Core Buildpack では、[環境変数の構成](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md) 手順に従って、アプリケーションのコンテナイメージを実行するときの環境変数を変更できます。

## アプリケーションのコンテナイメージにラベルを設定する
<!-- .Net Core Buildpack users can add labels to their app image by following the
instructions in the [Applying Custom
Labels](/docs/reference/configuration/#applying-custom-labels)
section of our configuration docs. -->
Paketo .NET Core Buildpack では、[ラベルの構成](/docs/reference/configuration/#applying-custom-labels) 手順に従って、アプリケーションのコンテナイメージにラベルを指定できます。
