---
title: "Paketo Buildpacks の設定"
weight: 300
menu:
  main:
    parent: reference
    indentifier: configuration
aliases:
  - /docs/buildpacks/configuration/
---

## 設定例について

<!-- Configuration examples will use the Paketo [sample applications][samples]. -->
[サンプルアプリケーション][samples] で設定例を説明します。

<!-- Examples assume that the root of this repository is the working directory: -->
それぞれの例では、サンプルアプリケーションのリポジトリのルートディレクトリを作業ディレクトリとします。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples
{{< /code/copyable >}}

<!-- The [pack CLI][pack] is used throughout the examples. `pack` is just one of several Cloud Native Buildpack [platforms][platforms] than can execute builds with Paketo Buildpacks. -->
全ての例で使用する [pack CLI][pack] は、Paketo Buildpack を使用してコンテナイメージをビルドできる Cloud Native Buildpack [Platform][platforms] の1つです。

<!-- Examples assume that the [Paketo Base builder][base builder] is the default builder: -->
それぞれの例では、既定のビルダーとして [Paketo Base ビルダー][base builder] を使用します。

{{< code/copyable >}}
pack config default-builder paketobuildpacks/builder:base
{{< /code/copyable >}}

## 設定の種類

<!-- Paketo buildpacks can be configured via the following mechanisms:

* [Environment Variables](#environment-variables) - used for generic configuration at both **build-time** and **runtime**.
* [buildpack.yml](#buildpackyml) - used for generic configuration at **build-time**.
* [Bindings](#bindings) - used for **secret** configuration at both **build-time** and **runtime**.
* [Procfiles](#procfiles) - used to provide custom **process types** at **build-time**. -->
Paketo Buildpack は次のような仕組みを通じて設定できます。

* [環境変数](#environment-variables) - **ビルド時点** と **実行時点** の両方で使用できる汎用的な設定方法です
* [buildpack.yml](#buildpackyml) - **ビルド時点** で使用できる汎用的な設定方法です
* [バインディング](#bindings) - **ビルド時点** と **実行時点** の両方で、**秘匿情報** の設定に使用できる設定方法です
* [Procfiles](#procfiles) - **ビルド時点** で独自の **プロセス種類** を指定できる設定方法です

### 環境変数

#### ビルド時に環境変数を指定する

<!-- Users may configure the build by setting variables in the buildpack environment. The names of variables accepted by the Paketo buildpacks at build-time are either prefixed with `BP_` or have well-known conventional meanings outside of Paketo (e.g. `http_proxy`). -->
利用者は、Buildpack の環境で使用する環境変数を指定することでビルドの仕方を変更できます。
ビルド時に使用できるのは名前が `BP_` で始まる変数や、Paketo Buildpack の外部で慣習的に使用されている変数（`http_proxy` など）です。

<!-- The following example uses an environment variable to configure the JVM version installed by the Java Buildpack. -->
Java Buildpack がインストールする JVM のバージョンを環境変数で指定するときは次のように実行します。

{{< code/copyable >}}
pack build samples/java  \
  --path java/jar \
  --env BP_JVM_VERSION=8
{{< /code/copyable >}}

<!-- During the build process, a buildpack may invoke other programs that accept configuration via the environment. Users may configure these tools as they would normally. For example, the command below configures the JVM memory settings for the JVM running Maven using [`MAVEN_OPTS`][maven opts]. -->
ビルド処理の途中、Buildpack は環境変数で指定された設定を引き継ぐようにして外部プログラムを実行します。
利用者は外部プログラムを普通に実行するときと同じように、環境変数を渡すことができるのです。
例えば、[`MAVEN_OPTS`][maven opts] で Maven の使用する JVM のメモリ設定を変更するときは次のように実行します。

{{< code/copyable >}}
pack build samples/java  \
  --path java/maven \
  --env "MAVEN_OPTS=-Xms256m -Xmx512m"
{{< /code/copyable >}}

#### 実行時に環境変数を指定する

<!-- Users may configure runtime features of the app image by setting environment variables in the app container.  The names of variables accepted by buildpack-provided runtime components (e.g. profile scripts and processes types) are prefixed with `BPL_` or have well-known conventional meanings outside of Paketo (e.g `JAVA_TOOL_OPTIONS`). -->
利用者は、コンテナイメージを実行するときの環境変数を通じて、実行時の設定を変更できます。
Buildpack の提供するランタイムコンポーネント（プロファイルスクリプトやプロセス種類）は名前が `BPL_` で始まる変数や、Paketo Buildpack の外部で慣習的に使用されている変数（`JAVA_TOOL_OPTIONS` など）です。

<!-- The following example uses `JAVA_TOOL_OPTIONS` to set the server port of the sample application: -->
Spring Boot のサンプルアプリケーションについて、環境変数 `JAVA_TOOL_OPTIONS` で待ち受けポート番号を指定するには次のように実行します。

{{< code/copyable >}}
docker run --rm --publish 8082:8082 --env "JAVA_TOOL_OPTIONS=-Dserver.port=8082" samples/java
curl -s http://localhost:8082/actuator/health
{{< /code/copyable >}}

<!-- Programs invoked at runtime, including the application itself, will accept environment as they would normally. -->
アプリケーションだけでなく、実行時に起動するプログラムは、普通に実行した場合と同じように環境変数を参照できます。

#### コンテナイメージに環境変数を埋め込む

<!-- Users may embed environment variables into the images created by using the [Environment Variables buildpack][bp/environment-variables]. The Environment Variables buildpack looks for environment variables matching the pattern `$BPE_*`. When detected, the buildpack will modify the launch environment to adjust the specified variables. This is a good way to set non-sensitive configuration values such as defaults or modify environment variables that you do not need users to set. -->
[Environment Variables buildpack][bp/environment-variables] を使うと、コンテナイメージ自体に環境変数を埋め込むことができます。
Environment Variables buildpack は名前が `BPE_*` にマッチする環境変数を探索し、アプリケーションを起動した環境で利用できるよう、起動環境を変更します。
秘匿性の低い設定項目の初期値を設定したり、利用者が設定する必要のない設定項目の環境変数を変更できます。

<!-- The buildpack supports the following actions on environment variables: -->
Environment Variables buildpack は次のような環境変数の操作に対応しています。
<!--
| Environment Variable Name | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `$BPE_<NAME>`             | set `$NAME` to value (same as override)                      |
| `$BPE_APPEND_<NAME>`      | append value to `$NAME`                                      |
| `$BPE_DEFAULT_<NAME>`     | set default value for `$NAME`                                |
| `$BPE_OVERRIDE_<NAME>`    | set `$NAME` to value                                         |
| `$BPE_PREPEND_<NAME>`     | prepend value to `$NAME`                                     |
 -->
| 環境変数名                | 説明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `$BPE_<NAME>`             | 環境変数 `NAME` を設定します。同じ名前の変数は上書きします   |
| `$BPE_APPEND_<NAME>`      | 環境変数 `NAME` の後ろに指定した値を追加します               |
| `$BPE_DEFAULT_<NAME>`     | 環境変数 `NAME` の初期値を設定します                         |
| `$BPE_OVERRIDE_<NAME>`    | 環境変数 `NAME` を設定します。同じ名前の変数は上書きします   |
| `$BPE_PREPEND_<NAME>`     | 環境変数 `NAME` の前に指定した値を追加します                 |

<!-- For more details on actions, you can refer to the [environment variable modification rules from the buildpacks spec](https://github.com/buildpacks/spec/blob/main/buildpack.md#environment-variable-modification-rules). -->
環境変数の操作について詳しくは [CNB の仕様](https://github.com/buildpacks/spec/blob/main/buildpack.md#environment-variable-modification-rules) を参照してください。

<!-- You can also change the delimiter used when appending or prepending by setting `$BPE_DELIM_<NAME>` for a particular variable name. It will default to an empty string (i.e. no delimiter). An example of using this would be to append to PATH or LD_LIBRARY_PATH, which are colon delimited. -->
環境変数 `BPE_DELIM_<NAME>` で、前後に値を追加するときの区切り文字を変更できます。
初期値は空文字列、すなわち、区切り文字なしになっています。
例えば、区切り文字が「コロン（:）」の環境変数 `PATH` や `LD_LIBRARY_PATH` へパス要素を追加するときに使うといいでしょう。

<!-- **DO NOT** embed sensitive credentials or information using the environment variables buildpack. This information is added to the image generated by your build tool, so anyone with access to the image can see what you embed using this buildpack. -->
**絶対にやらないでください**：Environment Variables buildpack で、秘匿性の高い資格情報をコンテナイメージに埋め込むのはやめましょう。
ビルドツールの生成したコンテナイメージに何らかの情報を埋め込むということは、そのコンテナイメージにアクセスできる人なら誰でも埋め込まれた情報を参照できることになるからです。

### buildpack.yml

<!-- Many Paketo buildpacks accept configuration from a `buildpack.yml` file if one is present at the root of the application directory. -->
多くの Paketo Buildpack は、アプリケーションのソースコードリポジトリの直下に置かれた `buildpack.yml` の設定を読み取ることができます。

<!-- For example, to configure the NodeJS version installed by the NodeJS Buildpack, create a file named `buildpack.yml` in the `nodejs/yarn` directory in the [samples repo][samples]. -->
例えば、Node.js Buildpack のインストールする Node.js のバージョンを指定するには、サンプルリポジトリの `nodejs/yarn` ディレクトリへ、次のような内容の `buildpack.yml` を配置します。

{{< code/copyable >}}
nodejs:
  version: 12.12.0
{{< /code/copyable >}}

<!-- Next, execute the build as normal and observe the that the specified version of NodeJs is installed. -->
同じようにビルドすると、指定したバージョンの Node.js をインストールします。

{{< code/copyable >}}
pack build samples/nodejs --path nodejs/yarn
{{< /code/copyable >}}

### バインディング

#### バインディングとは何か

<!-- Some Paketo Buildpacks and components installed by the Paketo Buildpacks accept credentials and other secrets using bindings at build and runtime. Commonly, bindings provide the location and credentials needed to connect to external services. -->
Paketo Buildpack や、Paketo Buildpack がインストールするコンポーネントの中には、ビルド時点と実行時点の両方で、バインディングに配置した資格情報やそれ以外の秘匿情報を使用できるものがあります。
一般的には、外部サービスへ接続するためのアドレス情報や資格情報を提供するためにバインディングを使用します。

<!-- Some categories of external services one might want to bind at build-time include:

* Private artifact repositories.
* SaaS security scanning tools.
 -->
外部サービスの中には、ビルド時点でバインディングを必要とするものがあります。

* プライベートアーティファクトリポジトリ
* SaaS のセキュリティスキャンツール

<!-- For example, the Maven buildpack accepts the location and credentials need to connect to a private Maven repository in a binding. -->
例えば、Maven Buildpack がプライベート Maven リポジトリへ接続するためのアドレス情報と資格情報はバインディングで構成できます。

<!-- Some categories of external services  one might want to bind at runtime include:

* APM servers.
* Data Services.
* OAuth2 providers.
 -->
外部サービスの中には、実行時点でバインディングを必要とするものがあります。

* APM
* データストア
* OAuth2 プロバイダ

<!-- For example, the Spring Boot Buildpack will install [Spring Cloud Bindings][spring cloud bindings] which is capable of auto-configuring Spring Boot application configuration properties to connect the application to a variety of external services, when a binding is provided at runtime. -->
例えば、Spring Boot Buildpack のインストールする [Spring Cloud Bindings][spring cloud bindings] は、バインディングで指定したさまざまな外部サービスの接続情報を、Spring Boot アプリケーションの自動構成プロパティとして使用できるようにします。

#### バインディングには何が入っているのか

<!-- A Binding contains: -->
バインディングには次のような内容を含みます。

<!-- 1. A **name**. Indentifies a particular binding. The name typically does not affect build or runtime behavior but may be used to reference a specific binding in output such as log messages.
1. A **type** or **kind**. Indicates what type of credentials the binding contains. For example, a binding of type `ApplicationInsights` contains the credentials needed to connect to Azure Application Insights.
1. An _optional_ **provider**. Indicates the source of the binding. For example, in a PaaS context, a specific service broker might provide the binding.
1. key-value pairs. These contain the configuration data. For example, an `ApplicationInsights` binding may contain a key-value pair with key `InstrumentationKey`. -->
1. **name（名前）**：バインディングを識別する名前です。基本的にビルド時点や実行時点の振る舞いには影響しませんが、ログ出力などでバインディングを参照するときの情報として使われる場合があります
2. **type（種類）あるいは kind（種類）**：バインディングに格納した資格情報の種類を表します。例えば、`ApplicationInsights` なら Azure Application Insights へ接続するための資格情報を含むことになります
3. **provider（プロバイダー）**：任意要素です。バインディングの提供元を表します。例えば、PaaS ではバインディングを提供するサービスブローカーを使う場合があります
4. キー・値の対：設定項目と値です。例えば `ApplicationInsights` のバインディングならキー `InstrumentationKey` とその値からなる対を含めることになります

<!-- Bindings must be presented to buildpacks as directories (typically volume mounted) on the container filesystem. The name of the directory provides the name of the binding. The contents of a binding can be provided using one of two specifications. -->
Buildpack にはコンテナファイルシステムのディレクトリとしてバインディングを連携します（ボリュームマウントを利用する場合が多いでしょう）。
ディレクトリ名がバインディングの名前を表します。
バインディングの（ディレクトリの）要素は次のいずれかの形式に従うようにします。

<!-- * [Service Binding Specification for Kubernetes][k8s service bindings]. This specification should be preferred over the CNB Bindings Specification when supported by the platform.
* [Cloud Native Buildpacks Bindings Specification][cnb bindings]. The original buildpacks bindings specification; this specification is en route to [deprecation][cnb bindings deprecation] -->
* [Kubernetes サービスバインディング仕様][k8s service bindings]：実行環境が対応しているなら CNB バインディング仕様よりこちらの仕様に従うことをお勧めします
* [Cloud Native Buildpacks バインディング仕様][cnb bindings]：伝統的な Buildpack のバインディング仕様ですが [廃止予定] です

<!-- Paketo Buildpacks will look for bindings in the `/platform/bindings` directory at build-time and in `$SERVICE_BINDING_ROOT` or `$CNB_BINDINGS` directory at runtime. -->
Paketo Buildpack はビルド時に参照するバインディングを `/platform/bindings` ディレクトリから探索します。
また、実行時に参照するバインディングは、環境変数 `SERVICE_BINDING_ROOT` あるいは `CNB_BINDINGS` の値が指すディレクトリから探索します。

<!-- For example, the Java Buildpack accepts a binding with `type` equal to `maven` containing a key named `settings.xml` containing [Maven settings][maven settings]. In the build container, the Maven Buildpack will use `settings.xml` if it finds either -->
例えば、Maven Buildpack は `type` が `maven` 、キーが `settings.xml` で値が [Maven settings][maven settings] になっているバインディングを受け付けます。
ビルド用コンテナでは、次のいずれかの場所で見つけた `settings.xml` を使用するのです。

{{< code/copyable >}}
/platform
└── bindings
    └── <name>
        ├── settings.xml
        └── type
{{< /code/copyable >}}

{{< code/copyable >}}
/platform
└── bindings
    └──<name>
        └── metadata
        |   └── kind
        └── secret
            └── settings.xml
{{< /code/copyable >}}
<!-- on the filesystem, where either the `type` or `kind` file contains the string `maven`. -->

#### バインディングの使い方

<!-- The workflow for creating a binding and providing it to a build will depend on the chosen platform. For example, `pack` users should use the `--volume` flag to mount a binding directory into the build or app containers. Users of the `kpack` platform should store key value pairs in a Kubernetes Secret and provide that secret and associated metadata to an Image as described in the [kpack documentation][kpack service bindings]. -->
バインディングの作成方法や、ビルド時に指定する方法は、プラットフォームによって異なります。
例えば、pack コマンドなら、 `--volume` フラグでボリュームマウントさせたディレクトリを使用します。
kpack コマンドなら、キーと値の対を Kubernetes の Secret リソースへ登録し、コンテナスペックのメタデータで secret フィールドに指定します。詳しくは [kpack でサービスバインディングを使う方法][kpack service bindings] を参照してください。

<!-- **Example:**  Providing a Binding to `pack build` -->
**例**：`pack build` コマンドにバインディングを指定する

<!-- Given a directory containing a build-time binding, `pack` users can provide this binding to a Paketo buildpack using the `--volume` flag. -->
あるディレクトリをビルド時に使用するバインディングとして使用する場合、`--volume` フラグで指定します。

{{< code/copyable >}}
pack build --volume <absolute-path-to-binding>:/platform/bindings/<binding-name> <image-name>
{{< /code/copyable >}}

<!-- **Example:**  Providing a Binding to `docker run` -->
**例**：`docker run` にバインディングを指定する

<!-- Given a directory containing a runtime binding, `docker` users can provide the binding to the app image using the `--volume` and `--env` flags -->
あるディレクトリを実行時に使用するバインディングとして使用する場合、`--volume` フラグと `--env` フラグで指定します。

{{< code/copyable >}}
docker run --env SERVICE_BINDING_ROOT=/bindings --volume <absolute-path-to-binding>:/bindings/<binding-name> <image-name>
{{< /code/copyable >}}

### Procfiles

<!-- Paketo users may override buildpack-provided types or augment the app-image with additional process types using a `Procfile`.  `Procfile` support is provided by the [Paketo Procfile Buildpack][bp/procfile]. The Procfile Buildpack will search for a file named `Procfile` at the root of the application. `Procfiles` should adhere to the following schema: -->
利用者は `Procfile` で Paketo Buildpack の提供する type（プロセス種類）を上書きしたり、使用できるプロセス種類を追加したりできます。
`Procfile` を処理できるのは [Paketo Procfile Buildpack][bp/procfile] です。
Procfile Buildpack はアプリケーションのルートディレクトリで `Procfile` を探索します。
`Procfile` の形式は次のとおりです。

```plain
<type>: <command>
```

<!-- If a given [language family buildpack][language family buildpacks] does not contain the Procfile Buildpack it can be explicitly appended at runtime. -->
指定された [言語モジュール Buildpack][language family buildpacks] に Procfile Buildpack が含まれていないときは、pack コマンドを実行するとき、明示的に指定します。

<!-- **Example**: A Hello World Procfile -->
**例**：Procfile で Hello World

<!-- The following adds a process with `type` equal to `hello` and makes it the default process. -->
プロセス種類に追加した `hello` を初期値にするときは次のようにします。

{{< code/copyable >}}
echo "hello: echo hello world" > nodejs/yarn/Procfile
pack build samples/nodejs \
  --path nodejs/yarn \
  --buildpack gcr.io/paketo-buildpacks/nodejs \
  --buildpack gcr.io/paketo-buildpacks/procfile \
  --default-process hello
docker run samples/nodejs # should print "hello world"
{{< /code/copyable >}}

## ファイアーウォールの内側でビルドする

### プロキシの設定

<!-- Paketo Buildpacks can be configured to route traffic through a proxy using the `http_proxy`, `https_proxy`, and `no_proxy` environment variables. `pack` will set these environment variables in the build container if they are set in the host environment. -->
Paketo Buildpack は環境変数 `http_proxy` や `https_proxy` や `no_proxy` で http トラフィックを送信するプロキシアドレスを変更できます。
ホストマシンで `pack` コマンドを実行するときにこれらの環境変数を設定すると、ビルドコンテナに継承させることができます。

### 依存関係の対応付け

<!-- Paketo Buildpacks may download dependencies from the internet. For example, the Java Buildpack with download the BellSoft Liberica JRE will from the Liberica [github releases][liberica releases] by default. -->
Paketo Buildpack はさまざまな依存対象をインターネットからダウンロードします。
例えば、Java Buildpack なら BellSoft Liberica JRE を Liberica の [github releases][liberica releases] からダウンロードします。

<!-- If a dependency URI is inaccessible from the build environment, a [binding](#bindings) can be used to map a new URI to a given dependency. This allows organizations to upload a copies of vetted dependencies to an accessible location and provide developers and CI/CD pipelines with configuration pointing the buildpack at the accessible dependencies. -->
依存対象の URI がビルド環境からアクセスできないネットワークにある場合、依存対象の URI として [バインディング](#bindings) と対応付けることができます。
組織としてアクセスを許可する場所へ検査済みの依存対象（の複製）をアップロードすれば、開発者や CI/CD パイプラインはバインディングで指定した場所から、依存対象へアクセスできるようになります。

<!-- The URI mappings can be configured with one or more bindings of `type` `dependency-mapping`. Each key value pair in the binding should map the `sha256` of a dependency to a URI. Information about the dependencies a buildpack may download (including the `sha256` and the current default `uri`) can be found in the `buildpack.toml` of each component buildpack. -->
URI の対応付けをするのは、`type` を `dependency-mapping` にした1つ以上のバインディングです。
それぞれのバインディングには、キー `sha256` の値として依存対象を参照可能な URI を格納します。
Buildpack が依存対象をダウンロードするために必要な情報（`sha256` や元の `uri` を含む）は、それぞれのコンポーネント Buildpack の `buildpack.toml` で定義されています。

<!-- **Example** Mapping the JRE to an internal URI -->
**例**：JRE を内部ネットワークの URI へ対応付ける

<!-- For example, to make the Bellsoft Liberica JRE dependency accessible available to builds in an environment where Github is inaccessible, an operator should: -->
GitHub へアクセスできない環境で、Bellsoft Liberica JRE を依存対象として含むビルドを成功させるには、次のようにします。

<!-- 1. Find the `sha256` and default `uri` for the desired dependency in [buildpack.toml][bp/bellsoft-liberica/descriptor] of the [Bellsoft Liberica buildpack][bp/bellsoft-liberica]. Example values:
    * `sha256`: `b4cb31162ff6d7926dd09e21551fa745fa3ae1758c25148b48dadcf78ab0c24c`
    * `uri`: `https://github.com/bell-sw/Liberica/releases/download/11.0.8+10/bellsoft-jre11.0.8+10-linux-amd64.tar.gz`
2. Download the dependency from the `uri` and upload it to a location on the internal network that is accessible during the build.
3. Create a binding with:
   * `type` equal to `dependency-mapping`
   * A key/value pair where the key is equal to the `sha256` of the dependency and the value is equal to the new URI.
4. Configure all builds with this binding. -->
1. [Bellsoft Liberica buildpack][bp/bellsoft-liberica] の [buildpack.toml][bp/bellsoft-liberica/descriptor] から必要な依存対象の `sha256` と元の `uri` を確認する。以下は具体例です
    * `sha256`: `b4cb31162ff6d7926dd09e21551fa745fa3ae1758c25148b48dadcf78ab0c24c`
    * `uri`: `https://github.com/bell-sw/Liberica/releases/download/11.0.8+10/bellsoft-jre11.0.8+10-linux-amd64.tar.gz`
2. `uri` から依存対象をダウンロードして、ビルド中にアクセスできる内部ネットワークのどこかへ配置する
3. バインディングを作成する
    * `type`: `dependency-mapping`
    * `sha256`: 内部ネットワークに配置した依存対象の URI
4. 作成したバインディングを指定してコンテナイメージをビルドします

## CA 証明書

<!-- Additional CA certificates may be added to the system truststore using the [Paketo CA Certificates Buildpack][bp/ca-certificates]. -->
[Paketo CA Certificates Buildpack][bp/ca-certificates] を使うとシステムのトラストストアへ CA 証明書を追加できます。

<!-- CA certificates can be provided at both build and runtime with a [binding](#bindings) of `type` `ca-certficates`. Each key value pair in the binding should map a certficate name to a single PEM encoded CA Certficates -->
CA 証明書を追加するには `type` が `ca-certficates` の [バインディング](#bindings) を使用します。
キーは証明書の名前、値は PEM 形式にした単独の CA 証明書にします。

```plain
<binding-name>
├── <cert file name>
└── type
```

<!-- If a given [language family buildpack][language family buildpacks] does not contain the Paketo CA Certificates Buildpack it can be explicitly prepended at runtime. -->
指定された [言語モジュール Buildpack][language family buildpacks] に CA Certficates Buildpack が含まれていないときは、pack コマンドを実行するとき、明示的に指定します。

<!-- **Example**: Adding a CA Certificate at Runtime -->
**例**：CA 証明書を追加する

<!-- The samples repository contains a simple Golang application that will make a `HEAD` request to a provided URL. -->
サンプルリポジトリの Go アプリケーションは指定した URL に `HEAD` リクエストを送信します。

<!-- Given a file `<your-ca.pem>` containing a single PEM encoded CA certificate needed to verify a TLS connection to an https URL `<url>`, add the CA certificate to the binding. -->
ファイル `<your-ca.pem>` は、`<url>` への TLS 通信をするための証明書で、PEM 形式にしておきます。
このファイルをバインディングへ追加しましょう。

```bash
cp <your-ca.pem> ca-certificates/binding/
```

<!-- The provided sample contains a simple Golang application that will make a `HEAD` request to a provided URL. Build the application using the CA Certificates buildpack -->
CA Certificates Buildpack を指定して `pack build` コマンドを実行します。

```bash
pack build samples/ca-certificates \
    --path ca-certificates \
    --buildpack paketo-buildpacks/ca-certificates \
    --buildpack paketo-buildpacks/go
```

<!-- Run the sample application, providing the binding, and passing the URL as a positional argument (should print `SUCCESS!`). -->
バインディングを指定して、引数に `<url>` を指定してサンプルアプリケーションを実行します。
成功すれば `SUCCESS!` と表示されます。

```bash
docker run --rm \
  --env SERVICE_BINDING_ROOT=/bindings \
  --volume "$(pwd)/ca-certficates/binding:/bindings/ca-certificates" \
  samples/ca-certificates <url>
```

<!-- **Disabling CA Certificates** -->
**CA 証明書を無効化する**

<!-- If a language family buildpack contains the Paketo CA Certifcates Buildpack,
the CA Certificates Buildpack will always pass detection so that certificates
can be provided dynamically at runtime. -->
言語モジュール Buildpack に Paketo CA Certifcates Buildpack が含まれている場合、コンテナイメージを実行するとき、常に証明書ファイルを探索し、追加するようになります。

<!-- To opt out of this behavior all together, the `BP_ENABLE_RUNTIME_CERT_BINDING`
environment variable can be set to `false` at build-time. This will disable the
ability to set certificates at runtime. The CA Certificates Buildpack will then
only detect if a certificate binding is provided at build-time. -->
この振る舞いを変更するには、ビルド時に環境変数 `BP_ENABLE_RUNTIME_CERT_BINDING` へ `false` を設定します。
そうすると、実行時に証明書を探索する機能は無効化され、CA Certifcates Buildpack はビルド時にバインディングで指定した証明書ファイルだけを探索するようになります。

## 独自ラベルを指定する

<!-- Paketo users may add labels to the application image using the [Image Labels Buildpack][bp/image-labels]. -->
[Image Labels Buildpack][bp/image-labels]を使うと、コンテナイメージにラベルを追加できます。

<!-- Environment variables prefixed with `BP_OCI_` can be used to set [OCI-specific][oci annotation keys]. For example, if `BP_OCI_AUTHORS` is set at build-time, the Image Labels Buildpack will add a label to the image with key `org.opencontainers.image.authors` and value equal to the value of `$BP_OCI_AUTHORS`. -->
名前が `BP_OCI_` で始まる環境変数は [OCI 特有のラベル][oci annotation keys] を設定するために使用できます。
例えば、ビルド時に `BP_OCI_AUTHORS` を指定すると、Image Labels Buildpack はキー `org.opencontainers.image.authors` のラベルを設定します。

<!-- Users may contribute arbitrary labels by providing a collection of space-delimited key-value pairs with the `BP_IMAGE_LABELS` environment variable. Values containing spaces can be quoted. -->
環境変数 `BP_IMAGE_LABELS` の値に、空白文字を区切り文字として並べたキーと値の対を指定すると、複数の任意のラベルを設定できます。
ラベルの値に空白文字を含む場合はクォート文字でエスケープできます。

<!-- If a given [language family buildpack][language family buildpacks] does not contain the Image Labels Buildpack it can be explicitly appended at runtime. -->
[言語モジュール Buildpack][language family buildpacks] に Image Labels Buildpack が含まれていないときは、pack コマンドを実行するとき、明示的に指定します。

<!-- **Example**: Adding Custom Labels -->
**例**：独自のラベルを設定する

{{< code/copyable >}}
pack build samples/nodejs \
  --path nodejs/yarn \
  --buildpack  gcr.io/paketo-buildpacks/nodejs \
  --buildpack  gcr.io/paketo-buildpacks/image-labels \
  --env "BP_OCI_DESCRIPTION=Demo Application" \
  --env 'BP_IMAGE_LABELS=io.packeto.example="Adding Custom Labels"'
docker inspect samples/nodejs | jq '.[].Config.Labels["org.opencontainers.image.description"]' # should print "Demo Application"
docker inspect samples/nodejs | jq '.[].Config.Labels["io.packeto.example"]' # should print "Adding Custom Labels"
{{< /code/copyable >}}

## ロケールを設定する

<!-- By default, an image created using Paketo buildpacks will not have a specific locale set. If you run `locale`, you'll end up with these settings: -->
初期設定では、Paketo Buildpack で作成したコンテナイメージにロケールは設定されていません。
`locale` コマンドを実行すると次のように表示されるでしょう。

```bash
LANG=
LANGUAGE=
LC_CTYPE="POSIX"
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=
```

<!-- If you wish to set a locale, you may do so when you run the image by setting the corresponding environment variable. For example, with Docker one could execute `docker run -e LANG=en_US.utf8 ...` to change the locale. -->
ロケールを設定するには適切な環境変数を指定しなければなりません。
例えば、Docker でコンテナイメージを実行するなら `docker run -e LANG=en_US.utf8 ...` のようにします。

<!-- This isn't always necessary but can impact output from your application. For example if you have an application that writes unicode characters to STDOUT/STDERR and you go to view those, possibly with `docker logs`, they will not display correctly unless you have a locale set that supports unicode, like UTF-8 in the example above. -->
常に必要な設定ではありませんが、アプリケーションの出力に影響する場合があります。
例えば、アプリケーションがユニコード文字列を標準出力と標準エラー出力へ書き込んでも、ユニコードに対応したロケールを設定していなければ、`docker logs` などで正しく表示されない可能性があります。

<!-- buildpacks -->
[bp/ca-certificates]:https://github.com/paketo-buildpacks/ca-certificates
[bp/image-labels]:https://github.com/paketo-buildpacks/image-labels
[bp/procfile]:https://github.com/paketo-buildpacks/procfile
[bp/bellsoft-liberica]:https://github.com/paketo-buildpacks/bellsoft-liberica
[bp/bellsoft-liberica/descriptor]:https://github.com/paketo-buildpacks/bellsoft-liberica/blob/main/buildpack.toml
[bp/environment-variables]:https://github.com/paketo-buildpacks/environment-variables

[samples]:https://github.com/paketo-buildpacks/samples

<!-- paketo docs references -->
[base builder]:{{< ref "/docs/concepts/builders#base" >}}
[language family buildpacks]:{{< ref "/docs/concepts/buildpacks" >}}

<!-- cnb references -->
[pack]:https://github.com/buildpacks/pack
[platforms]:https://buildpacks.io/docs/concepts/components/platform/

<!-- other references -->
[cnb bindings deprecation]:https://github.com/buildpacks/rfcs/blob/main/text/0055-deprecate-service-bindings.md
[cnb bindings]:https://github.com/buildpacks/spec/blob/main/extensions/bindings.md
[k8s service bindings]:https://github.com/k8s-service-bindings/spec
[kpack service bindings]:https://github.com/pivotal/kpack/blob/master/docs/servicebindings.md#service-bindings
[liberica releases]:https://github.com/bell-sw/Liberica/releases
[maven settings]:http://maven.apache.org/settings.html
[maven opts]:https://maven.apache.org/configure.html#maven_opts-environment-variable
[oci annotation keys]:https://github.com/opencontainers/image-spec/blob/master/annotations.md#pre-defined-annotation-keys
[spring cloud bindings]:https://github.com/spring-cloud/spring-cloud-bindings
