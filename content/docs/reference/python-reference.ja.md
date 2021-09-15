---
title: "Python Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: python-reference
    name: "Python Buildpack"
---

{{% reference_exec_summary bp_name="Paketo Python Buildpack" bp_repo="https://github.com/paketo-buildpacks/python" howto_docs_path="/docs/howto/python" %}}

## 対応している依存対象
<!-- The Python buildpack supports several versions of CPython, Pip and Pipenv.  For
more details on the specific versions supported in a given buildpack version,
see the [release notes](https://github.com/paketo-buildpacks/python/releases). -->
Python Buildpack は CPython や pip や Pipenv の複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート](https://github.com/paketo-buildpacks/python/releases) で確認してください。

## Buildpack の設定する環境変数

<!-- The Python Buildpack sets a few environment variables during the `build` and
`launch` phases of the app lifecycle. The sections below describe each
environment variable and its impact on your app. -->
Python Buildpack はアプリケーションライフサイクルのビルドフェーズや起動フェーズでいくつかの環境変数を設定します。
このセクションではアプリケーションに影響するであろう環境変数について説明します。

### PYTHONPATH

<!-- The [`PYTHONPATH`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH)
environment variable is used to add directories where python will look for
modules. -->
環境変数 [`PYTHONPATH`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH) には、アプリケーションを実行するときに Python がモジュールを探索するパスを追加します。

* Set by: `CPython`, `Pip` and `Pipenv`
* Phases: `build` and `launch`

<!-- The CPython buildpack sets the `PYTHONPATH` value to its installation location,
and the Pip, Pipenv buildpack prepends their `site-packages` location to it.
`site-packages` is the target directory where packages are installed to. -->
CPython Buildpack は Python をインストールしたディレクトリと、Pip をインストールしたディレクトリのパスを `PYTHONPATH` へ追加します。
Pipenv Buildpack は `site-packages` のパスを `PYTHONPATH` へ追加します。
`site-packages` とはパッケージをインストールするディレクトリのことです。

### PYTHONUSERBASE

<!-- The [`PYTHONUSERBASE`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONUSERBASE)
environment variable is used to set the user base directory. -->
環境変数 [`PYTHONUSERBASE`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONUSERBASE) にはアプリケーションを実行するユーザの既定ディレクトリのパスを指定します。

* Set by: `Pip Install` and `Pipenv Install`
* Phases: `build` and `launch`

<!-- The value of `PYTHONUSERBASE` is set to the location where these buildapcks install
the application packages so that it can be consumed by the app source code. -->
`PYTHONUSERBASE` には Buildpack がアプリケーションの依存パッケージをインストールした場所を設定します。
アプリケーションがそれらのパッケージを参照できるようにするためです。

### 起動コマンド

<!-- The Python Buildpack sets the default start command `python`. This starts the Python
REPL (read-eval-print loop) at launch. -->
Python Buildpack は起動コマンドに `python` を設定します。
つまり、コンテナイメージを実行すると REPL（read-eval-print loop）を開始します。

<!-- The Python Buildpack comes with support for
[`Procfile`]({{< ref "docs/reference/configuration#procfiles" >}})
that lets users set custom start commands easily. -->
Python Buildpack は [`Procfile`]({{< ref "docs/reference/configuration#procfiles" >}}) に対応しているので、利用者は簡単に起動コマンドを変更できます。
