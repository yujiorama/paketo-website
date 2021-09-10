---
title: "Paketo Buildpacks で Java アプリケーションをビルドする"
weight: 316
menu:
  main:
    parent: "howto"
    name: "Java"
aliases:
  - /docs/buildpacks/language-family-buildpacks/java/
  - /docs/buildpacks/language-family-buildpacks/java-native-image/

---

<!-- This documentation explains how to use the Paketo buildpacks
to build Java applications for several common use-cases. For more in-depth
description of the buildpacks' behavior and configuration see the [Paketo Java Buildpack][reference/java] and [Paketo Java Native Image Buildpack][reference/java-native-image] reference documentation. -->
このドキュメントでは Paketo Buildpack を使用して Java アプリケーションのコンテナイメージを作成する方法を説明します。
Buildpack の振る舞いや設定方法を詳しく知りたいときは、[Paketo Java Buildpack][reference/java] や [Paketo Java Native Image Buildpack][reference/java-native-image] のリファレンスを参照してください。

## サンプルアプリをビルドする

<!-- All Java Buildpack examples will use the Paketo [sample applications][samples]. -->
このドキュメントでは [Paketo Buildpacs サンプルアプリ][samples] を使用します。

<!-- Examples assume that the root of this repository is the working directory: -->
それぞれの例では、リポジトリ直下を作業ディレクトリとしています。

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples
{{< /code/copyable >}}

<!-- The [pack CLI][pack] is used throughout the examples. `pack` is just one of several Cloud Native Buildpack [platforms][platforms] than can execute builds with the Java Buildpacks. For example, Spring Boot developers may want to explore the [Spring Boot Maven Plugin][spring boot maven plugin] or [Spring Boot Gradle Plugin][spring boot gradle plugin] . -->
[pack コマンド][pack] も使用します。
`pack` は Java Buildpack でビルドを実行できる Cloud Native Buildpack [platform][platform] の1つです。
例えば、Spring Boot アプリケーションの開発者なら [Spring Boot Maven Plugin][spring boot maven plugin] や [Spring Boot Gradle Plugin][spring boot gradle plugin] を使用できます。

<!-- Examples assume that the [Paketo Base builder][base builder] is the default builder: -->
それぞれの例では、 [Paketo Base ビルダー][base builder] を初期値にしています。

{{< code/copyable >}}
pack config default-builder paketobuildpacks/builder:base
{{< /code/copyable >}}

<!-- All java example images should return `{"status":"UP"}` from the [actuator health endpoint][spring boot actuator endpoints]. -->
Java アプリケーションのサンプルは、どれも [actuator health endpoint][spring boot actuator endpoints] で `{"status":"UP"}` を返すようになっています。

{{< code/copyable >}}
docker run --rm --tty --publish 8080:8080 samples/java
curl -s http://localhost:8080/actuator/health | jq .
{{< /code/copyable >}}

## ソースコードからビルドする

<!-- The Java Buildpack can build from source using any of the following build tools: -->
Java Buildpack は次のようなビルドツールでソースコードをビルドできるようになっています。

* [Gradle][gradle] - [Gradle Buildpack][bp/gradle]
* [Leiningen][leiningen] - [Leiningen Buildpack][bp/leiningen]
* [Maven][maven] - [Maven Buildpack][bp/maven]
* [SBT][sbt] - [SBT Buildpack][bp/sbt]

<!-- The correct build tool to use will be detected based on the contents of the
application directory. -->
使用するビルドツールはアプリケーションディレクトリの内容に基づいて検出します。

<!-- The build should produce one the of [supported artifact
formats][build-from-compiled-artifact]. After building, the buildpack
will replace provided application source code with the exploded archive. The
build will proceed as described in [Building from a Compiled
Artifact][build-from-compiled-artifact]. -->
ビルド処理は [対応しているいずれかの形式のアーティファクト][build-from-compiled-artifact] を生成しなければなりません。
完了すると、buildpack は展開したアーカイブの内容でアプリケーションのソースコードを置き換えます。
その後の処理については [コンパイル済みアーティファクトからのビルド][build-from-compiled-artifact] を参照してください。

<!-- **Example**: Building with Maven -->
**例**：Maven でビルドする

<!-- The following command creates an image from source with `maven`. -->
次のコマンドは Maven プロジェクトのソースコードからコンテナイメージを作成します。

{{< code/copyable >}}
pack build samples/java \
  --path java/maven
{{< /code/copyable >}}

### ビルドツールを設定する

<!-- **Note**: The following set of configuration options are not comprehensive, see the homepage for the relevant component buildpacks for a full-set of configuration options. -->
**注意：以降の内容は全ての設定項目を網羅したものではありません。設定項目について、詳しくはビルドツールに対応する Buildpack のリファレンスを参照してください。**

#### モジュールやアーティファクトを選択する

<!-- For a given build `<TOOL>`, where `<TOOL>` is one of `MAVEN`, `GRADLE`, `LEIN` or `SBT`, the selected artifact can be configured with one of the following environment variable at build-time: -->
以降の説明に登場する `<TOOL>` には `MAVEN` や `GRADLE` や `LEIN` や `SBT` のいずれかで置き換えることになります。
モジュールやアーティファクトを選択するには、次のような環境変数をビルド時に指定します。

* `BP_<TOOL>_BUILT_MODULE`
  * **初期値** はルートモジュールです。
  * マルチモジュールプロジェクトにおいて、Buildpack がアーティファクトをビルドするモジュールを選択します。
  * **例**：`BP_MAVEN_BUILT_MODULE=api` と指定すると、Paketo Maven Buildpack は `target/api/*.[jw]ar` に対応するファイル名のアーティファクトを選択します。
<!--   * *Defaults* to the root module.
  * Configures the module in a multi-module build from which the buildpack will select the application artifact.
  * *Example*: Given `BP_MAVEN_BUILT_MODULE=api`, Paketo Maven Buildpack will look for the application artifact with the file pattern `target/api/*.[jw]ar`. -->
* `BP_<TOOL>_BUILT_ARTIFACT`
  * 初期値はビルドツールによって異なります（Maven なら `target/*.[jw]ar`、Gradle なら `build/libs/*.[jw]ar`）。ビルドツールに対応する Buildpack のリファレンスを参照してください。
  * アーティファクトを探索するパス名のパターンを [Bashのパターンマッチ](bash pattern matching) で指定します。
  * `BP_<TOOL>_BUILT_MODULE` に初期値以外が設定されているとしても、こちらの設定を優先します。
  * **例**：`BP_MAVEN_BUILT_ARTIFACT=out/api-*.jar` と指定すると、Paketo Maven Buildpack は `out/api-1.0.0.jar` を発見します。
<!--   * Defaults to a tool-specific pattern (e.g. `target/*.[jw]ar` for Maven, `build/libs/*.[jw]ar` for gradle). See component buildpack homepage for details.
  * Configures the built application artifact path, using [Bash Pattern Matching][bash pattern matching].
  * Supercedes `BP_<TOOL>_BUILT_MODULE` if set to a non-default value.
  * *Example*: Given `BP_MAVEN_BUILT_ARTIFACT=out/api-*.jar`, the Paketo Maven Buildpack will select a file with name `out/api-1.0.0.jar`. -->

#### ビルドコマンドを指定する

<!-- For a given build `<TOOL>`, where `<TOOL>` is one of `MAVEN`, `GRADLE`, `LEIN` or `SBT`, the build command can be configured with the following environment variable at build-time: -->
以降の説明に登場する `<TOOL>` には `MAVEN` や `GRADLE` や `LEIN` や `SBT` のいずれかで置き換えることになります。
モジュールやアーティファクトを選択するには、次のような環境変数をビルド時に指定します。

* `BP_<TOOL>_BUILD_ARGUMENTS`
<!--   * *Defaults* to a tool-specific value (e.g. `-Dmaven.test.skip=true package` for Maven, `--no-daemon assemble` for Gradle). See component buildpack homepage for details.
  * Configures the arguments to pass to the build tool.
  * *Example*: Given `BP_GRADLE_BUILD_ARGUMENTS=war`, the Paketo Gradle Buildpack will execute `./gradlew war` or `gradle war` (depending on the presence of the gradle wrapper). -->
  * **初期値**：ビルドツールによって異なります（Maven なら `-Dmaven.test.skip=true package`、Gradle なら `--no-daemon assemble`）。ビルドツールに対応する Buildpack のリファレンスを参照してください。
  * 指定した値がビルドツールのコマンドの引数になります。
  * **例**：`BP_GRADLE_BUILD_ARGUMENTS=war` と指定すると、Paketo Gradle Buildpack は `./gradlew war` か `gradle war` を実行します（Gradle Wrapper の有無によって変化します）。

#### プライベート Maven リポジトリを使用する

<!-- A [binding][bindings] with type `maven` and key `settings.xml` can be used to provide custom [Maven settings][maven settings]. -->
`maven` [バインディング](bindings) のキーに `settings.xml` を指定すると、独自の [Maven settings][maven settings] を使用できます。

```plain
<binding-name>
├── settings.xml
└── type
```

<!-- The value of `settings.xml` file may contain the credentials needed to connect to a private Maven repository. -->
`settings.xml` ファイルには、プライベート Maven リポジトリへ接続するための資格情報を記述できます。

<!-- **Example**: Providing Maven Settings -->
**例**: 独自の Maven Settings を構成する

<!-- The following steps demonstrate how to use a `settings.xml` file from your workstation with `pack`. -->
作業 PC の `settings.xml` を使用して、`pack` コマンドでアプリケーションをビルドする手順を説明します。

<!-- 1. Create a directory to contain the binding. -->
1. バインディングリソースを格納するディレクトリを作成します。
{{< code/copyable >}}
mkdir java/maven/binding
{{< /code/copyable >}}

<!-- 2. Indicate that the binding is of type `maven` with a file called `type` inside the binding, containing the value `maven`. -->
2. 作成したディレクトリに、バインディング種類 `maven` を含む `type` ファイルを作成します。
{{< code/copyable >}}
echo -n "maven" > java/maven/binding/type
{{< /code/copyable >}}

<!-- 3. Copy the `settings.xml` file from the workstation to the binding. -->
3. 作成したディレクトリに、作業PCの `settings.xml` を複製します。
{{< code/copyable >}}
cp ~/.m2/settings.xml java/maven/binding/settings.xml
{{< /code/copyable >}}

<!-- 4. Provide the binding to `pack build`. -->
4. 作成したディレクトリを、`pack build` コマンドの `--volume` フラグに指定します。
{{< code/copyable >}}
pack build samples/java \
   --path java/maven \
   --volume $(pwd)/java/maven/binding:/platform/bindings/my-maven-settings
{{< /code/copyable >}}

## コンパイル済みアーティファクトからビルドする

<!-- An application developer may build an image from following archive formats: -->
Buildpack を利用するアプリケーション開発者は、次のようなアーカイブ（の形式）からコンテナイメージをビルドできます。

* [実行可能 jar ファイル][executable jar] - [Executable Jar Buildpack][bp/executable-jar]
* [war ファイル][war] - [Apache Tomcat Buildpack][bp/apache-tomcat]
* [配布用 zip アーカイブ][dist-zip] - [DistZip Buildpack][bp/dist-zip]

<!-- The Java Buildpack expects the application directory to contain the extracted contents of the archive (e.g. an exploded JAR). Most platforms will automatically extract any provided archives. -->
Java Buildpack はアーカイブを展開した場所をアプリケーションディレクトリとして扱います。
指定されたアーカイブが何であれ、ほとんどのプラットフォームでは自動的に展開します。

<!-- If a WAR is detect the Java Buildpack will install [Apache Tomcat][apache tomcat]. For exact set of supported Tomcat versions can be found in the Java Buildpack [releases notes][bp/java/releases]. For tomcat configuration options see the [Apache Tomcat Buildpack][bp/apache-tomcat]. -->
Java Buildpack は war ファイルを検出すると [Apache Tomcat](apache tomcat) をインストールします。
Java Buildpack の使用する Tomcat のバージョンについては、[Paketo Java Buildpack のリリースノート][bp/java/releases] を参照してください。
Tomcat の設定項目については [Apache Tomcat Buildpack][bp/apache-tomcat] を参照してください。

<!-- The component buildpack for the provided artifact format will contribute a start command to the image. -->
指定したアーティファクトの形式に対応する Buildpack は、作成するコンテナイメージの起動コマンドを指定してくれます。

<!-- *Note*: All three of the [Apache Tomcat Buildpack][bp/apache-tomcat],
[Executable Jar Buildpack][bp/executable-jar], and [DistZip
Buildpack][bp/dist-zip] may opt-in during detection. However, only one of these
buildpacks will actually contribute to the final image. This happens because
the artifact type may be unknown during detection, if for example a previous
buildpack [compiles the artifact][building-from-source]. -->
**注意**：アーティファクトの形式に応じて、 [Apache Tomcat Buildpack][bp/apache-tomcat] 、[Executable Jar Buildpack][bp/executable-jar] 、 [Dist Zip Buildpack][bp/dist-zip] の3種類から1つ以上の Buildpack を選択します。しかし、最終的なコンテナイメージを作成するのはどれか1つの Buildpack です。[ソースコードからビルドした場合][building-from-source] 等、アーティファクトの形式を検出できない場合があります。

<!-- **Example**: Building from an Executable JAR -->
**例**：実行可能 jar ファイルをビルドする

<!-- The following command uses Maven to compile an executable JAR and then uses `pack` to build an image from the JAR. -->
Maven でビルドした実行可能 jar ファイルを使用して、`pack` コマンドでアプリケーションをビルドするには、次のように実行します。

{{< code/copyable >}}
cd java/maven
./mvnw package
pack build samples/java \
   --path /target/demo-0.0.1-SNAPSHOT.jar
{{< /code/copyable >}}

<!-- The resulting application image will be identical to that built in the Building with Maven example. -->
作成したコンテナイメージは、「Maven でビルドする」でビルドしたイメージと同じになっているはずです。

## JVM のバージョンを確認する

<!-- The exact JRE version that was contributed to a given image can be read from the Bill-of-Materials. -->
作成したコンテナイメージに含まれる正確な JRE のバージョンは、BOM（Bill-of-Materials） から読み取ることができます。

<!-- **Example** Inspecting the JRE Version -->
**例** JRE のバージョンを確認する

<!-- Given an image named `samples/java` built from one of examples above, the following command should print the exact version of the installed JRE. -->
サンプルプロジェクトから作成したコンテナイメージ `samples/java` について、インストールされている JRE のバージョンを表示するには、次のように実行します。

{{< code/copyable >}}
pack inspect-image samples/app --bom | jq '.local[] | select(.name=="jre") | .metadata.version'
{{< /code/copyable >}}

## JVM のインストールするバージョンを指定する

<!-- The following environment variable configures the JVM version at build-time. -->
JVM のバージョンを指定するときは、ビルド時に次の環境変数を指定します。

* `BP_JVM_VERSION`
  * 初期値は、Buildpack をリリースした時点で最新の LTS バージョンです。
  * JDK あるいは JRE のバージョンを指定します。
  * **例**：`BP_JVM_VERSION=8` あるいは `BP_JVM_VERSION=8.*` とした場合、Buildpack は Java 8 の JDK あるいは JRE の、最新のパッチリリースをインストールします。
<!--   * Defaults to the latest LTS version at the time of release.
  * Configures a specific JDK or JRE version.
  * *Example*: Given `BP_JVM_VERSION=8` or `BP_JVM_VERSION=8.*` the buildpack will install the latest patch releases of the Java 8 JDK and JRE. -->

## JVM の動作を変更する

<!-- The Java Buildpack configures the JVM by setting `JAVA_TOOL_OPTIONS` in the JVM environment. -->
Java Buildpack では環境変数 `JAVA_TOOL_OPTIONS` で JVM の動作を調整するようになっています。

<!-- The runtime JVM can be configured in two ways: -->
JVM の動作を調整する方法は2種類あります。

<!-- 1. Buildpack-provided runtime components including the Memory Calculator accept semantically named environment variables which are then used to derive `JAVA_TOOL_OPTIONS` flags. Examples include:
    * `BPL_JVM_HEAD_ROOM`
    * `BPL_JVM_LOADED_CLASS_COUNT`
    * `BPL_JVM_THREAD_COUNT`
2. Flags can be set directly at runtime with the `JAVA_TOOL_OPTIONS` environment variable. User-provided flags will be appended to buildpack-provided flags. If the user and a buildpack set the same flag, user-provided flags take precedence. -->
1. Buildpack の提供する実行時コンポーネントのメモリー設定計算機が使用する、次のような環境変数を指定します。これらの値は最終的に `JAVA_TOOL_OPTIONS` へ設定する内容を算出するために使用されます。
    * `BPL_JVM_HEAD_ROOM`
    * `BPL_JVM_LOADED_CLASS_COUNT`
    * `BPL_JVM_THREAD_COUNT`
2. 直接 `JAVA_TOOL_OPTIONS` を指定します。ユーザーの指定した内容は Buildpack の構成した内容の最後に追加されます。同じ項目を指定した場合、ユーザーの指定した値を優先します。

<!-- See the [homepage][bp/bellsoft-liberica] for the Bellsoft Liberica Buildpack for a full set of configuration options. -->
全ての設定項目については [Bellsoft Liberica Buildpack のリファレンス][bp/bellsoft-liberica] を参照してください。

## 別の JVM を使用する

<!-- By default, the [Paketo Java buildpack][bp/java] will use the Liberica JVM. The following Paketo JVM buildpacks may be used to substitute alternate JVM implemenations in place of Liberica's JVM. -->
初期設定の [Paketo Java Buildpack][bp/java] は Liberica JVM を使用します。
Liberica の代わりに使用できる JVM （と対応する Buildpack） は次のとおりです。

| JVM                                                         | Buildpack                                                            |
| ----------------------------------------------------------- | -------------------------------------------------------------------- |
| [Alibaba Dragonwell](http://dragonwell-jdk.io/)             | [Paketo Alibaba Dragonwell Buildpack][bp/dragonwell]                 |
| [Amazon Corretto](https://aws.amazon.com/corretto/)         | [Paketo Amazon Corretto Buildpack][bp/amazon-corretto]               |
| [Azul Zulu](https://www.azul.com/downloads/zulu-community/) | [Paketo Azul Zulu Buildpack][bp/azul-zulu]                           |
| [BellSoft Liberica](https://bell-sw.com/pages/libericajdk/) | [Paketo BellSoft Liberica Buildpack - Default][bp/bellsoft-liberica] |
| [Eclipse OpenJ9](https://www.eclipse.org/openj9/)           | [Paketo Eclipse OpenJ9 Buildpack][bp/eclipse-openj9]                 |
| [GraalVM](https://www.graalvm.org/)                         | [Paketo GraalVM Buildpack][bp/graalvm]                               |
| [Microsoft OpenJDK](https://www.microsoft.com/openjdk)      | [Paketo Microsoft OpenJDK Buildpack][bp/microsoft]                   |
| [SapMachine](https://sap.github.io/SapMachine/)             | [Paketo SapMachine Buildpack][bp/sap-machine]                        |

<!-- To use an alternative JVM, you will need to set two `--buildpack` arguments to `pack build`, one for the alternative JVM buildpack you'd like to use and one for the Paketo Java buildpack (in that order). This works because while you end up with two JVM buildpacks, the first one, the one you're specifying will claim the build plan entries so the second one will end up being a noop and doing nothing. -->
別の JVM を使用するときは、`pack build` コマンドに2つの `--buildpack` フラグを指定しなければなりません。
最初に JVM Buildpack を指定し、次に Paketo Java Buildpack を指定します（順番が重要です）。
そうすると、`pack build` は2つの JVM Buildpack を使用することになるのですが、最初に指定した Buildpack がビルドプランの要素として登録され、後に指定した Buildpack は何もしない状態になります。

<!-- This example will switch in the Azul Zulu buildpack: -->
次の例は Azul Zulu Buikdpack を使用しています。

{{< code/copyable >}}
pack build samples/jar \
    --buildpack gcr.io/paketo-buildpacks/azul-zulu \
    --buildpack paketo-buildpacks/java
{{< /code/copyable >}}

<!-- There is one drawback to this approach. When using the method above to specify an alternative JVM vendor buildpack, this alternate buildpack ends up running before the CA certs buildpack and therefore traffic from the alternate JVM vendor buildpack won’t trust any additional CA certs. This is not expected to impact many users because JVM buildpacks should reach out to URLs that have a cert signed by a known authority with a CA in the default system truststore. -->
この方法には1つ欠点があります。
CA Certs Buildpack より先に JVM Buildpack を取得するので、JVM ベンダーの提供するコンテンツを取得するとき、CA 証明局は最新化されておらず、信頼できないコンテンツを取得してしまう場合があるのです。
ですが、ほとんどのユーザーには影響しないものだと考えています。
ほとんどの JVM Buildpack はデフォルトのシステムトラストストアに含まれている CA が書名した証明書を使用するURL（ドメイン）で提供されているはずだからです。

<!-- If you have customized your JVM buildpack to download the JVM from a URL that uses a certificate not signed by a well-known CA, you can workaround this by specifying the CA certs buildpack to run first. This works because while you will end up with the CA certificates buildpack specified twice, the lifecycle is smart enough to drop the second one. -->
カスタマイズした JVM Buildpack を、既存の CA 認証局が署名してない証明書を使用する URL からダウンロードさせるときは、最初に CA Certs Buildpack を実行するといいでしょう。
CA Cert Buildpack を重複して実行することになりますが、ライフサイクルは2回目の実行を無視できるくらい簡潔なままです。

<!-- For example: -->
具体的には次のように実行します。

{{< code/copyable >}}
pack build samples/jar \
    --buildpack paketo-buildpacks/ca-certificates \
    --buildpack gcr.io/paketo-buildpacks/azul-zulu \
    --buildpack paketo-buildpacks/java
{{< /code/copyable >}}

<!-- It does not hurt to use this command for all situations, it is just more verbose and most users can get away without specifying the CA certificates buildpack to be first. -->
常にこのような使い方をしていても、メッセージや処理が冗長になるだけで特に問題はありません。
また、ほとんどのユーザーは CA Cert Buildpack を先頭にしなくても問題になりません。

## 別の Native Image Toolkit を使用する

<!-- By default, the [Paketo Java Native Image buildpack][bp/java-native-image] will use the GraalVM Native Image Toolkit. The following Paketo JVM buildpacks may be used to substitute alternate Native Image Toolkit implemenations in place of the default. -->
初期設定の [Paketo Java Native Image Buildpack][bp/java-ntive-image] は GraalVM Native Image Toolkit を使用します。
代わりに使用できる Native Image Toolkit （と対応する Buildpack） は次のとおりです。

| JVM                                                                       | Buildpack                                                  |
| ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| [Bellsoft Liberica](https://bell-sw.com/pages/liberica-native-image-kit/) | [Paketo Bellsoft Liberica Buildpack][bp/bellsoft-liberica] |

<!-- To use an alternative Java Native Image Toolkit, you will need to set two `--buildpack` arguments to `pack build`, one for the alternative Java Native Image Toolkit buildpack you'd like to use and one for the Paketo Java Native Image buildpack (in that order). This works because while you end up with two Java Native Image Toolkit buildpacks, the first one, the one you're specifying will claim the build plan entries so the second one will end up being a noop and doing nothing. -->
別の Java Native Image Toolkit を使用するときは、`pack build` コマンドに2つの `--buildpack` フラグを指定しなければなりません。
最初に Java Native Image Toolkit Buildpack を指定し、次に Paketo Java Native Image Buildpack を指定します（順番が重要です）。
そうすると、`pack build` は2つの Native Image Toolkit Buildpack を使用することになるのですが、最初に指定した Buildpack がビルドプランの要素として登録され、後に指定した Buildpack は何もしない状態になります。

<!-- This example will switch in the Bellsoft Liberica buildpack: -->
Bellsoft Liberica Buildpack を使う場合は次のように実行します。

{{< code/copyable >}}
pack build samples/native-image \
    --buildpack paketo-buildpacks/bellsoft-liberica \
    --buildpack paketo-buildpacks/java-native-image
{{< /code/copyable >}}

<!-- There is one drawback to this approach. When using the method above to specify an alternative Java Native Image Toolkit vendor buildpack, this alternate buildpack ends up running before the CA certs buildpack and therefore traffic from the alternate Java Native Image Toolkit vendor buildpack won’t trust any additional CA certs. This is not expected to impact many users because Java Native Image Toolkit buildpacks should reach out to URLs that have a cert signed by a known authority with a CA in the default system truststore.

If you have customized your Java Native Image Toolkit buildpack to download the Java Native Image Toolkit from a URL that uses a certificate not signed by a well-known CA, you can workaround this by specifying the CA certs buildpack to run first. This works because while you will end up with the CA certificates buildpack specified twice, the lifecycle is smart enough to drop the second one.

For example:

{{< code/copyable >}}
pack build samples/jar --buildpack paketo-buildpacks/ca-certificates --buildpack paketo-buildpacks/bellsoft-liberica --buildpack paketo-buildpacks/java-native-image
{{< /code/copyable >}}

It does not hurt to use this command for all situations, it is just more verbose and most users can get away without specifying the CA certificates buildpack to be first. -->

## Spring Boot アプリケーションをビルドする

### Spring Boot アプリケーションの依存ライブラリを確認する

<!-- The following command uses `pack` to list every dependency of a sample application. -->
次のコマンドを実行すると、サンプルアプリケーションで作成したコンテナイメージの依存ライブラリを確認できます。

{{< code/copyable >}}
pack inspect-image samples/java --bom | \
jq '.local[] | select(.name=="dependencies") | .metadata.dependencies[].name'
{{< /code/copyable >}}

### Spring Boot の自動構成を無効にする

<!-- The Spring Boot Buildpack adds [Spring Cloud Bindings][spring cloud bindings] to the application class path. Spring Cloud Bindings will auto-configure the application to connect to an external service when a binding of a supported type provides credentials and connection information at runtime. Runtime auto-configuration is enabled by default but can be disabled with the `BPL_SPRING_CLOUD_BINDINGS_ENABLED` environment variable. -->
Spring Boot Buildpack はクラスパスに [Spring Cloud Bindings][spring cloud bindings] を追加します。
Spring CloudBindings は、バインディングに配置した資格情報と接続設定に基づいて、外部サービスへ接続するための設定を、実行時に自動的に構成します。
初期設定で自動構成機能は有効になっていますが、環境変数 `BPL_SPRING_CLOUD_BINDINGS_ENABLED` を指定すると無効にできます。

## APM へ接続する

<!-- The Java Buildpack supports the following [APM][apm] integrations: -->
Java Buildpack は次のような [APM][apm] と連携できるようになっています。

* [Azure Application Insights][azure application insights] - [Azure Application Insights Buildpack][bp/azure-application-insights]
* [Google Stackdriver][google stackdriver] - [Google Stackdriver Buildpack][bp/google-stackdriver]

<!-- APM integrations are enabled with [bindings][bindings]. If a binding of the correct `type` is provided at build-time the corresponding java agent will be contributed to the application image. Connection credentials will be read from the binding at runtime. -->
APM 連携機能は [バインディング][bindings] で有効にします。
ビルド時に適切なバインディング種類が存在する場合、対応する Java エージェントをコンテナイメージに埋め込みます。
APM へ接続するための資格情報は、実行時にバインディングから読み取ります。

<!-- **Example**: Connecting to Azure Application Insights -->
**例**：Azure Application Insights に接続する

<!-- The following command builds an image with the Azure Application Insights Java Agent -->
次のコマンドは、Azure Application Insights の Java エージェントをコンテナイメージに埋め込みます。

{{< code/copyable >}}
pack build samples/java \
    --volume "$(pwd)/java/application-insights/binding:/platform/bindings/application-insights"
{{< /code/copyable >}}

<!-- To connect to Azure Applicaiton Insights at runtime a valid [Instrumentation Key][azure application insights instrumentation key] is required. -->
アプリケーションが Azure Application Insights へ接続するには正式な [Instrumentation Key][azure application insights instrumentation key] が必要です。

{{< code/copyable >}}
echo "<Instrumentation Key>" > java/application-insights/binding/InstrumentationKey
docker run --rm --tty \
  --env SERVICE_BINDING_ROOT=/bindings \
  --volume "$(pwd)/java/application-insights/binding:/bindings/app-insights" \
  samples/java
{{< /code/copyable >}}

## リモートデバッグを有効にする

<!-- If `BP_DEBUG_ENABLED` is set at build-time and `BPL_DEBUG_ENABLED` is set at runtime the [Debug Buildpack][bp/debug] will configure the application to accept debugger connections. The debug port defaults to `8000` and can be configured with `BPL_DEBUG_PORT` at runtime. If `BPL_DEBUG_SUSPEND` is set at runtime, the JVM will suspend execution until a debugger has attached. -->
ビルド時に環境変数 `BP_DEBUG_ENABLED` を指定すると、 [Debug Buildpack](bp/debug) により、Java アプリケーションがデバッグ接続を待ち受けるように設定します。
そして、実行時に環境変数 `BPL_DEBUG_ENABLED` を指定すると、Java アプリケーションがデバッグ接続を待ち受けるようになります。
デバッグ接続用の TCP ポート番号の初期値は `8000` ですが、実行時に環境変数 `BPL_DEBUG_PORT` で変更できます。
実行時に環境変数で `BPL_DEBUG_SUSPEND` を指定すると、デバッガがアタッチするまで JVM プロセスは実行を中断するようになります。

<!-- **Example**: Remote Debugging -->
**例**：リモートデバッグ

<!-- The following commands builds a debug-enabled image. -->
リモートデバッグできるコンテナイメージを作成するには次のように実行します。

{{< code/copyable >}}
pack build samples/java \
  --path java/jar \
  --env BP_DEBUG_ENABLED=true
{{< /code/copyable >}}

<!-- To run the image with the debug port published: -->
デバッグ接続用の TCP ポートを公開するには次のようにコンテナイメージを実行します。

{{< code/copyable >}}
docker run --env BPL_DEBUG_ENABLED=true --publish 8000:8000 samples/java
{{< /code/copyable >}}

<!-- Connect your IDE debugger to connect to the published port. -->
IDE のデバッガで公開した TCP ポートに接続できます。

![Eclipse Remote Debug Configuration](/images/debug-eclipse.png)

## JMX を有効にする

<!-- If `BP_JMX_ENABLED` is set at build-time and `BPL_JMX_ENABLED` is set at runtime, the [JMX Buildpack][bp/jmx] will enable [JMX][jmx]. The JMX connector will listen on port `5000` by default. The port can be configured with the `BPL_JMX_PORT` environment variable at runtime. -->
ビルド時に環境変数 `BP_JMX_ENABLED` を指定すると、 [JMX Buildpack][bp/jmx] が [JMX][jmx] を有効にします。
そして、実行時に環境変数 `BPL_JMX_ENABLED` を指定すると、Java アプリケーションへ JMX で接続できるようになります。
JMX 接続に使用する TCP ポート番号の初期値は `5000` ですが、環境変数 `BPL_JMX_PORT` で変更できます。

<!-- **Example**: Enabling JMX -->
**例**: JMX を有効にする

<!-- The following commands builds a JMX enabled image. -->
JMX 接続できるコンテナイメージを作成するには次のように実行します。

{{< code/copyable >}}
pack build samples/java \
  --path java/jar \
  --env BP_JMX_ENABLED=true
{{< /code/copyable >}}

<!-- To run the image with the JMX port published: -->
JMX 接続用の TCP ポートを公開するには次のようにコンテナイメージを実行します。

{{< code/copyable >}}
docker run --env BPL_JMX_ENABLED=true --publish 5000:5000 samples/java
{{< /code/copyable >}}

<!-- Connect [JConsole][jconsole] to the published port. -->
[JConsole][jconsole] で公開した TCP ポートに接続できます。

![JConsole](/images/jconsole.png)

## コンテナイメージの起動コマンドに引数を追加する
<!-- Additional arguments can be provided to the application using the container [`CMD`][oci config]. In Kubernetes set `CMD` using the `args` field on the [container][kubernetes container resource] resource. -->
コンテナイメージの [`CMD`][oci config] に、起動コマンドの引数を追加できます。
Kubernetes では [container][kubernetes container resource] リソースの `args` フィールドへ指定します。

<!-- **Example**: Setting the Server Port -->
**例**: サーバーポート番号を指定する

<!-- Execute the following command passes an additional argument to application start command, setting the port to `8081`. -->
アプリケーションの起動コマンドの引数で、待ち受けポート番号として `8081` を指定するには次のように実行します。
{{< code/copyable >}}
docker run --rm --publish 8081:8081 samples/java --server.port=8081
curl -s http://localhost:8081/actuator/health
{{< /code/copyable >}}

## コンテナイメージの起動コマンドを指定する

<!-- To override the buildpack-provided start command with a custom command, set the container [`ENTRYPOINT`][oci config] -->
Buildpack の構成した起動コマンドは、コンテナイメージの [`ENTRYPOINT`][oci config] で変更できます。

<!-- **Example**: Starting an Interactive Shell -->
**例**: 対話形式のシェルを実行する

<!-- The following command runs Bash interactively: -->
対話形式で Bash を起動するには次のように実行します。
{{< code/copyable >}}
docker run --rm --entrypoint bash samples/java
{{< /code/copyable >}}

### Buildpack の構成した環境で指定したコマンドを実行する

<!-- Every buildpack-generated image contains an executable called the `launcher` which can be used to execute a custom command in an environment containing buildpack-provided environment variables. The `launcher` will execute any buildpack provided profile scripts before running to provided command, in order to set environment variables with values that should be calculated dynamically at runtime. -->
Buildpack で作成したコンテナイメージには、実行可能ファイル `launcher` が必ず存在します。
このファイルは、Buildpack の構成する環境変数を設定した状態で指定したコマンドを実行するために使用できます。
`launcher` は指定したコマンドを実行する前に、Buildpack の配置した全てのプロファイルスクリプトを実行します。
環境変数の値を実行時に計算するためです。

<!-- To run a custom start command in the buildpack-provided environment set the `ENTRYPOINT` to `launcher` and provide the command using the container `CMD`. -->
コンテナイメージの `ENTRYPOINT` に `launcher` を指定し、`CMD` に実行したいコマンドを指定すれば、Buildpack の構成した環境で指定したコマンドを実行できます。

<!-- **Example**: Inspecting the Buildpack-Provided `JAVA_TOOL_OPTIONS` -->
**例**: Buildpack の構成した `JAVA_TOOL_OPTIONS` を確認する

<!-- The following command will print value of `$JAVA_TOOL_OPTIONS` set by the buildpack: -->
Buildpack の構成した `JAVA_TOOL_OPTIONS` の値を表示するには次のように実行します。
{{< code/copyable >}}
docker run --rm --entrypoint launcher samples/java echo 'JAVA_TOOL_OPTIONS: $JAVA_TOOL_OPTIONS'
{{< /code/copyable >}}

<!-- Each argument provided to the launcher will be evaluated by the shell prior to execution and the original tokenization will be preserved. Note that, in the example above `'JAVA_TOOL_OPTIONS: $JAVA_TOOL_OPTIONS'` is single quoted so that `$JAVA_TOOL_OPTIONS` is evaluated in the container, rather than by the host shell. -->
`launcher` に指定したコマンドは実行前にシェルで評価しますが、元のトークン区切りを保持します。
例えば、`'JAVA_TOOL_OPTIONS: $JAVA_TOOL_OPTIONS'` という文字列はシングルクォートで囲まれているため、`$JAVA_TOOL_OPTIONS` の部分はホストのシェルではなく、コンテナ上で評価されます。

## アプリケーションを GraalVM のネイティブイメージとしてビルドする
<!-- The [Paketo Java Native Image Buildpack][bp/java-native-image] allows users to create an image containing a [GraalVM][graalvm] [native image][graalvm native image] application. -->
[Paketo Java Native Image Buildpack][bp/java-native-image] は、[GraalVM][graalvm] の [native image][graalvm native image] でビルドしたアプリケーションを含む、コンテナイメージを作成します。

<!-- The Java Native Buildpack is a [composite buildpack][composite buildpack] and
each step in a build is handled by one of its [components][components]. The
following docs describe common build configurations. For a full set of
configuration options and capabilities see the homepages of the component
buildpacks. e-->
Java Native Buildpack は [合成 Buildpack][composite buildpack] です。
`build` コマンドのそれぞれの段階は、それぞれの [コンポーネント][components] が制御するようになっています。
以降の説明では、基本的な設定項目を説明します。
全ての設定項目や、利用できる機能については対応するコンポーネント（Buildpack）のリファレンスを参照してください。

### ソースコードからビルドする

<!-- The Java Native Image Buildpack supports the same [build tools and
configuration options][java/building from source] as the [Java
Buildpack][bp/java]. The build must produce an [executable jar][executable
jar]. -->
Java Native Image Buildpack では [Java Buildpack][bp/java] と同じ [ビルドツールや設定項目][java/building from source] を使用できます。
ただし、[実行可能形式のjarファイル][executable jar] を生成しなければなりません。

<!-- After compiling and packaging, the buildpack will replace provided application
source code with the exploded JAR and proceed as described in [Building from an
Executable Jar][building-from-an-executable-jar]. -->
Buildpack はコンパイルとパッケージングを完了すると、作成した jar ファイルを展開してアプリケーションのソースコードを置き換えます。
そして、 [実行可能形式の jar ファイルからビルド][building-from-an-executable-jar] します。

<!-- **Example**: Building a Native image with Maven -->
**例**： Maven で Native Image をビルドする

<!-- The following command creates an image from source with `maven`. -->
次のコマンドは Maven プロジェクトのソースコードからコンテナイメージを作成します。

{{< code/copyable >}}
pack build samples/java-native \
  --env BP_NATIVE_IMAGE=true \
  --path java/native-image/java-native-image-sample
{{< /code/copyable >}}

### 実行可能形式の jar ファイルからビルドする

<!-- An application developer may build an image from an exploded [executable JAR][executable jar]. Most platforms will automatically extract provided archives. -->
[実行可能形式の jar ファイル][executable jar] を展開する方法でコンテナイメージを作成できます。
ほとんどのプラットフォームは指定されたファイルのアーカイブ形式を自動的に判断し、展開できます。

<!-- **Example**: Building a Native image from an Executable JAR -->
**例**：実行可能形式の jar ファイルから Native Image をビルドする

<!-- The following command uses Maven directly to compile an executable JAR and then uses the `pack` CLI to build an image from the JAR. -->
Maven プロジェクトのソースコードからビルドした実行可能形式の jar ファイルを `pack` コマンドに指定して、コンテナイメージを作成するときは次のように実行します。

{{< code/copyable >}}
cd samples/java/native-image
./mvnw package
pack build samples/java-native \
  --env BP_NATIVE_IMAGE=true \
  --path java/native-image/java-native-image-sample/target/demo-0.0.1-SNAPSHOT.jar
{{< /code/copyable >}}

<!-- The resulting application image will be identical to that built in the "Building a Native image with Maven" example. -->
作成したコンテナイメージは「Maven で Native Image をビルドする」でビルドしたイメージと同じになっているはずです。

### JVM のバージョンを確認する

<!-- The exact substrate VM version that was contributed to a given image can be read from the Bill-of-Materials. -->
作成したコンテナイメージに含まれる正確な Substrate VM のバージョンは、BOM（Bill-of-Materials） から読み取ることができます。

<!-- **Example** Inspecting the JRE Version -->
**例** JRE のバージョンを確認する

<!-- Given an image named `samples/java-native` built from one of examples above, the following command will print the exact version of the installed substrate VM. -->
サンプルプロジェクトから作成したコンテナイメージ `samples/java-native` について、インストールされている Substrate VM のバージョンを表示するには、次のように実行します。

{{< code/copyable >}}
pack inspect-image samples/java-native --bom | \
jq '.local[] | select(.name=="native-image-svm") | .metadata.version'
{{< /code/copyable >}}

### GraalVM のバージョンを指定する

<!-- Because GraalVM is evolving rapidly you may on occasion need to, for compatibility reasons, select a sepecific version of the GraalVM and associated tools to use when building an image. This is not a directly configurable option like the JVM version, however, you can pick a specific version by changing the version of the Java Native Image Buildpack you use. -->
GraalVM は急速に進歩しているので、互換性を保証しなければならない場合は、コンテナイメージをビルドする GraalVM や関連するツールのバージョンを指定するといいでしょう。
JVM のバージョンのように直接指定することはできないのですが、代わりに Java Native Image Buildpack のバージョンを指定すれば、対応する GraalVM のバージョンを指定できます。

<!-- The following table documents the versions available. -->
GraalVM と Java Native Image Buildpack のバージョンの対応関係は次のとおりです。

| GraalVM Version | Java Native Image Buildpack Version |
| --------------- | ----------------------------------- |
| 21.2            | 5.5.0                               |
| 21.1            | 5.4.0                               |
| 21.0            | 5.3.0                               |

<!-- For example, to select GraalVM 21.1: -->
GraalVM 21.1 を使用するには次のように実行します。

{{< code/copyable >}}
pack build samples/native \
    -e BP_NATIVE_IMAGE=true \
    --buildpack gcr.io/paketo-buildpacks/ca-certificates \
    --buildpack gcr.io/paketo-buildpacks/java-native-image:5.4.0
{{< /code/copyable >}}

<!-- buildpacks -->
[bp/amazon-corretto]:https://github.com/paketo-buildpacks/amazon-corretto
[bp/apache-tomcat]:https://github.com/paketo-buildpacks/apache-tomcat
[bp/azul-zulu]:https://github.com/paketo-buildpacks/azul-zulu
[bp/azure-application-insights]:https://github.com/paketo-buildpacks/azure-application-insights
[bp/bellsoft-liberica]:https://github.com/paketo-buildpacks/bellsoft-liberica
[bp/ca-certificates]:https://github.com/paketo-buildpacks/ca-certificates
[bp/debug]:https://github.com/paketo-buildpacks/debug
[bp/dist-zip]:https://github.com/paketo-buildpacks/dist-zip
[bp/dragonwell]:https://github.com/paketo-buildpacks/alibaba-dragonwell
[bp/eclipse-openj9]:https://github.com/paketo-buildpacks/eclipse-openj9
[bp/environment-variables]:https://github.com/paketo-buildpacks/environment-variables
[bp/environment-variables]:https://github.com/paketo-buildpacks/environment-variables
[bp/executable-jar]:https://github.com/paketo-buildpacks/executable-jar
[bp/executable-jar]:https://github.com/paketo-buildpacks/executable-jar
[bp/google-stackdriver]:https://github.com/paketo-buildpacks/google-stackdriver
[bp/graalvm]:https://github.com/paketo-buildpacks/graalvm
[bp/graalvm]:https://github.com/paketo-buildpacks/graalvm
[bp/gradle]:https://github.com/paketo-buildpacks/gradle
[bp/gradle]:https://github.com/paketo-buildpacks/gradle
[bp/image-labels]:https://github.com/paketo-buildpacks/image-labels
[bp/image-labels]:https://github.com/paketo-buildpacks/image-labels
[bp/java-native-image]:https://github.com/paketo-buildpacks/java-native-image
[bp/java]:https://github.com/paketo-buildpacks/java
[bp/java]:https://github.com/paketo-buildpacks/java
[bp/jmx]:https://github.com/paketo-buildpacks/jmx
[bp/leiningen]:https://github.com/paketo-buildpacks/leiningen
[bp/leiningen]:https://github.com/paketo-buildpacks/leiningen
[bp/maven]:https://github.com/paketo-buildpacks/maven
[bp/maven]:https://github.com/paketo-buildpacks/maven
[bp/microsoft]:https://github.com/paketo-buildpacks/microsoft-openjdk
[bp/native-image]:https://github.com/paketo-buildpacks/spring-boot-native-image
[bp/procfile]:https://github.com/paketo-buildpacks/procfile
[bp/procfile]:https://github.com/paketo-buildpacks/procfile
[bp/sap-machine]:https://github.com/paketo-buildpacks/sap-machine
[bp/sbt]:https://github.com/paketo-buildpacks/sbt
[bp/sbt]:https://github.com/paketo-buildpacks/sbt
[bp/spring-boot]:https://github.com/paketo-buildpacks/spring-boot
[bp/spring-boot]:https://github.com/paketo-buildpacks/spring-boot

<!-- paketo references -->
[bp/java/releases]:https://github.com/paketo-buildpacks/java/releases
[samples]:https://github.com/paketo-buildpacks/samples

<!-- paketo docs references -->
[base builder]:{{< ref "/docs/concepts/builders#base" >}}
[bindings]:{{< ref "/docs/reference/configuration#bindings" >}}
[build-from-compiled-artifact]:{{< relref "#build-from-a-compiled-artifact" >}}
[building-from-source]:{{< relref "#build-from-source" >}}
[components]:{{< ref "/docs/reference/java-native-image-reference#components" >}}
[composite buildpack]:{{< ref "/docs/concepts/buildpacks#composite-buildpacks" >}}
[java/building from source]:{{< ref "/docs/howto/java#building-from-source" >}}
[java/spring boot applications]:{{< ref "/docs/howto/java#spring-boot-applications" >}}
[reference/java-native-image]:{{< ref "/docs/reference/java-native-image-reference" >}}
[reference/java]:{{< ref "/docs/reference/java-reference" >}}

<!-- cnb references -->
[pack]:https://github.com/buildpacks/pack
[platforms]:https://buildpacks.io/docs/concepts/components/platform/

<!-- other references -->
[apache tomcat]:http://tomcat.apache.org
[apm]:https://en.wikipedia.org/wiki/Application_performance_management
[azure application insights instrumentation key]:https://docs.microsoft.com/en-us/azure/azure-monitor/app/create-new-resource#copy-the-instrumentation-key
[azure application insights]:https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview
[bash pattern matching]:https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html
[dist-zip]:https://docs.gradle.org/current/userguide/distribution_plugin.html
[executable jar]:https://en.wikipedia.org/wiki/JAR_(file_format)#Executable_JAR_files
[executable jar]:https://en.wikipedia.org/wiki/JAR_(file_format)#Executable_JAR_files
[google stackdriver]:https://cloud.google.com/products/operations
[graalvm feature]:https://www.graalvm.org/sdk/javadoc/org/graalvm/nativeimage/hosted/Feature.html
[graalvm native image]:https://www.graalvm.org/reference-manual/native-image/
[graalvm substrate vm]:https://www.graalvm.org/reference-manual/native-image/SubstrateVM/
[graalvm]:https://www.graalvm.org/docs/introduction/
[gradle]:https://gradle.org/
[java]:https://github.com/paketo-buildpacks/java
[jconsole]:https://openjdk.java.net/tools/svc/jconsole/
[jmx]:https://en.wikipedia.org/wiki/Java_Management_Extensions#:~:text=Java%20Management%20Extensions%20(JMX)%20is,MBeans%20(for%20Managed%20Bean)
[kubernetes container resource]:https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#container-v1-core
[leiningen]:https://leiningen.org/
[liberica]:https://bell-sw.com/
[maven settings]:http://maven.apache.org/settings.html
[maven]:https://maven.apache.org/
[oci config]:https://github.com/opencontainers/image-spec/blob/master/config.md#properties
[sbt]:https://www.scala-sbt.org/index.html
[spring boot actuator endpoints]:https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints
[spring boot configuration metadata]:https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html
[spring boot gradle plugin]:https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#build-image
[spring boot gradle plugin]:https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#build-image
[spring boot maven plugin]:https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/html/#build-image
[spring boot maven plugin]:https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/html/#build-image
[spring cloud bindings]:https://github.com/spring-cloud/spring-cloud-bindings
[spring native prerequisites]:https://repo.spring.io/milestone/org/springframework/experimental/spring-graalvm-native-docs/0.8.5/spring-graalvm-native-docs-0.8.5.zip!/reference/index.html#_prerequisites
[spring native releases]:https://github.com/spring-projects-experimental/spring-native/releases
[spring native]:https://github.com/spring-projects-experimental/spring-native
[war]:https://en.wikipedia.org/wiki/WAR_(file_format)

