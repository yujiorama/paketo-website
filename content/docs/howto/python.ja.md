---
title: "Paketo Buildpacks で Python アプリケーションをビルドする"
weight: 324
menu:
  main:
    parent: "howto"
    name: "Python"
aliases:
  - /docs/buildpacks/language-family-buildpacks/python/
---

{{% howto_exec_summary bp_name="Paketo Python Buildpack" bp_repo="https://github.com/paketo-buildpacks/python" reference_docs_path="/docs/reference/python-reference" %}}

## サンプルアプリをビルドする
<!-- To build a sample app locally with this buildpack using the pack CLI, run -->
`pack` コマンドを使って、Paketo Python Buildpack でサンプルアプリをビルドします。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/python
cd samples/python/pip
pack build my-app --buildpack gcr.io/paketo-buildpacks/python \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- See
[samples](https://github.com/paketo-buildpacks/python/tree/main/samples/python/pip)
for how to run the app. -->
アプリケーションの実行方法は [README ファイル](https://github.com/paketo-buildpacks/python/tree/main/samples/python/pip) を参照してください。

<!-- The Paketo Python Buildpack supports several popular configurations for Python apps. -->
Paketo Python Buildpack は、さまざまな種類の Python アプリケーションに対応しています。

## CPython のインストールするバージョンを指定する

<!-- The Python Cloud Native Buildpack allows you to specify a version of CPython 3
(reference implementation of Python 3) to use during deployment. This version
can be specified via the `BP_CPYTHON_VERSION` environment variable during
build. When specifying a version of CPython, you must choose a version that is
available within the buildpack. The supported versions can be found in the
[release notes](https://github.com/paketo-buildpacks/python/releases/latest). -->
Python Cloud Native Buildpack では、デプロイ時に使用する CPython 3 （Python 3 の参照実装）のバージョンを指定できるようになっています。
バージョンは環境変数 `BP_CPYTHON_VERSION` で指定できます。
指定できるバージョンは、[Python Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/python/releases/latest) で確認できます。

<!-- You may set `BP_CPYTHON_VERSION` using a platfrom-specific option, or using
a [`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor)
as shown in the following example: -->
環境変数 `BP_CPYTHON_VERSION` は、Buildpack のプラットフォームが要求する方法や、[`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor) で指定できます。

{{< code/copyable >}}
[build]
  [[build.env]]
    name = "BP_CPYTHON_VERSION"
    value = "3.6.*" # any valid semver constraints (e.g. 3.6.7, 3.*) are acceptable
{{< /code/copyable >}}

<!-- Specifying a version of CPython is not required. In the case this is not
specified, the buildpack will provide the default version, which can be seen in
the `buildpack.toml` file. -->
CPython のバージョンを指定しなかった場合、Buildpack の `buildpack.toml` に記述された初期値を使用します。

## パッケージマネージャを使用する

<!-- With the Python CNB, there are three options available for package management
depending on your application:

* Using [Pip](https://pip.pypa.io)
* Using [Pipenv](https://pypi.org/project/pipenv)
* Using [Miniconda](https://docs.conda.io/en/latest/miniconda.html) -->
Python Cloud Native Buildpack は次のようなパッケージマネージャに対応しています。

* [Pip](https://pip.pypa.io)
* [Pipenv](https://pypi.org/project/pipenv)
* [Miniconda](https://docs.conda.io/en/latest/miniconda.html)

<!-- You can find specific information for each option below. -->
それぞれのパッケージマネージャについて詳しく説明していきます。

### Pip

<!-- Pip is a popular option for managing third-party application dependencies for
Python apps.  Including a valid `requirements.txt` file at the root of your app
source code triggers the pip installation process by the buildpack. The
buildpack will install the application packages and make it available to the
app. -->
Pip は Python アプリケーションの依存ライブラリを管理するツールです。
Buildpack はアプリケーションコードベースのルートディレクトリに配置された `requirements.txt` を発見すると、pip をインストールし、アプリケーションの使用する依存ライブラリをインストールします。

<!-- The buidpack allows you to configure the version of Pip to be used in the
installation process. You can set this using the `$BP_PIP_VERSION` variable
during build. When specifying a version of Pip, you must choose a version that
is available within the buildpack. The supported versions can be found in the
[release notes](https://github.com/paketo-buildpacks/python/releases/latest). -->
環境変数 `BP_PIP_VERSION` により、インストールする Pip のバージョンを指定できます。
指定できるバージョンは、[Python Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/python/releases/latest) で確認できます。

### Pipenv

<!-- Pipenv is another common option for managing dependencies. Including a valid
`Pipfile` file at the root of your app source code triggers the pipenv
installation process by the buildpack. The buildpack will install the
application packages and make it available to the app. -->
Pipenv は Python アプリケーションの依存ライブラリを管理するツールです。
Buildpack はアプリケーションコードベースのルートディレクトリに配置された `Pipfile` を発見すると、pipenv をインストールし、アプリケーションの使用する依存ライブラリをインストールします。

<!-- The buidpack allows you to configure the version of Pipenv to be used in the
installation process. You can set this using the `$BP_PIPENV_VERSION` variable
during build. When specifying a version of Pipenv, you must choose a version
that is available within the buildpack. The supported versions can be found in the
[release notes](https://github.com/paketo-buildpacks/python/releases/latest). -->
環境変数 `BP_PIPENV_VERSION` により、インストールする Pipenv のバージョンを指定できます。
指定できるバージョンは、[Python Cloud Native Buildpack のリリースノート](https://github.com/paketo-buildpacks/python/releases/latest) で確認できます。

<!-- The buildpack also takes into consideration the Python version requirement
specified by `Pipfile.lock`, but `BP_CPYTHON_VERSION` takes precedence over
this as discussed in [this section above]({{< relref "#install-a-specific-cpython-version" >}}). -->
Buildpack は `Pipfile.lock` で指定されたバージョンの CPython を使おうとします。
ただし、環境変数 `BP_CPYTHON_VERSION` が指定されているときは、そちらの内容を優先します。

### Miniconda

<!-- Miniconda is a package management and environment management system supported
by the Python buildpack. The builpack will create or update a conda environment
from an `environment.yml` file or a `package-list.txt` file located at the root
of the app source code. -->
Miniconda は Python アプリケーションの実行環境と依存ライブラリを管理するツールです。
Buildpack はアプリケーションコードベースのルートディレクトリに配置された `environment.yml` あるいは `package-list.txt` を発見すると、conda の環境を作成、および、更新します。

<!-- Configuring a version of miniconda is not supported. -->
miniconda のバージョンは指定できません。

## 独自の CA 証明書をインストールする
<!-- Python Buildpack users can provide their own CA certificates and have them
included in the container root truststore at build-time and runtime by
following the instructions outlined in the [CA
Certificates]({{< ref "docs/reference/configuration#ca-certificates" >}})
section of our configuration docs. -->
Paketo Python Buildpack では、ビルド時や実行時のどちらでも、[CA 証明書の構成](/docs/reference/configuration/#ca-certificates) 手順に従って、ユーザーが自分で用意した CA 証明書をコンテナのルートトラストストアへ配置できます。

## Buildpack の設定する起動プロセスを変更する
<!-- Python Buildpack users can set custom start processes for their app image by
following the instructions in the
[Procfiles]({{< ref "docs/reference/configuration#procfiles" >}}) section
of our configuration docs. -->
Paketo Python Buildpack では、[Procfiles の導入](/docs/reference/configuration/#procfiles) 手順に従って、アプリケーションのコンテナイメージが起動するプロセスを変更できます。

## アプリケーションを起動するときの環境変数を設定する
<!-- Python Buildpack users can embed launch-time environment variables in their
app image by following the documentation for the [Environment Variables
Buildpack](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md). -->
Paketo Python Buildpack では、[環境変数の構成](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md) 手順に従って、アプリケーションのコンテナイメージを実行するときの環境変数を変更できます。

## アプリケーションのコンテナイメージにラベルを設定する
<!-- Python Buildpack users can add labels to their app image by following the
instructions in the [Applying Custom
Labels]({{< ref "docs/reference/configuration#applying-custom-labels" >}})
section of our configuration docs. -->
Paketo Python Buildpack では、[ラベルの構成](/docs/reference/configuration/#applying-custom-labels) 手順に従って、アプリケーションのコンテナイメージにラベルを指定できます。
