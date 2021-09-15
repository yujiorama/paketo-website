---
title: "Java Native Image Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: java-native-image-reference
    name: "Java Native Image Buildpack"
---

{{% reference_exec_summary bp_name="Paketo Java Native Image Buildpack" bp_repo="https://github.com/paketo-buildpacks/java-native-image" howto_docs_path="/docs/howto/java/#build-an-app-as-a-graalvm-native-image-application" %}}

<!-- The [Paketo Java Native Image Buildpack][bp/java-native-image] allows users to create an image containing a [GraalVM][graalvm] [native image][graalvm native image] application. The Java Native Image Buildpack supports the same [build tools and configuration options][java/building from source] as the [Java Buildpack][bp/java]. The build must produce an [executable jar][executable jar]. -->
[Paketo Java Native Image Buildpack][bp/java-native-image] は、[GraalVM][graalvm] の [native-image][graalvm native image] でビルドしたアプリケーションのコンテナイメージを作成します。
Java Native Image Buildpack では [Java Buildpack][bp/java] と同じ [ビルドツールおよび設定項目][java/building from source] を使用できます。
ただし、[実行可能形式の jar ファイル][executable jar] をビルドしなければなりません。

## 対応しているアプリケーション

<!-- For all native image builds, it is required that:

* `BP_NATIVE_IMAGE` is set at build time.
 -->
アプリケーションを native-image でビルドするには、ビルド時に環境変数 `BP_NATIVE_IMAGE` へ `true` を設定します。

<!-- For Spring Boot applications, it is required that:

* The application declares a dependency on [Spring Native][spring native].
* The version of [Spring Native][spring native] declared by the application may require a specific version of Spring Boot. See the Spring Native [release notes][spring native releases] for supported Spring Boot versions.
 -->
Spring Boot アプリケーションを native-image でビルドするには次のようにします。

* 依存ライブラリに [Spring Native][spring native] を定義します
* [Spring Native][spring native] のバージョンによって利用できる Spring Boot のバージョンが制限される場合があります。[Spring Native のリリースノート][spring native releases] で確認してください。

## コンポーネント

<!-- The following component buildpacks compose the Paketo Java Native Image Buildpack. -->
Java Native Image Buildpack の使用するコンポーネント Buildpack は次のとおりです。
一覧に並んだ順で実行します。
<!--
| Buildpack                                                          | Required/Optional | Responsibility                                                                                                    |
| ------------------------------------------------------------------ | ----------------- | ----------------------------------------------------------------------------------------------------------------- |
| [Paketo CA Certificates Buildpack][bp/ca-certificates]             | Optional          | Adds CA certificates to the system truststore at build and runtime.                                               |
| [Paketo GraalVM Buildpack][bp/graalvm]                             | **Required**      | Provides the GraalVM JDK and Native Image [Substrate VM](https://www.graalvm.org/reference-manual/native-image/). |
| [Paketo Gradle Buildpack][bp/gradle]                               | Optional          | Builds Gradle-based applications from source.                                                                     |
| [Paketo Leiningen Buildpack][bp/leiningen]                         | Optional          | Builds Leiningen-based applications from source.                                                                  |
| [Paketo Maven Buildpack][bp/maven]                                 | Optional          | Builds Maven-based applications from source.                                                                      |
| [Paketo SBT Buildpack][bp/sbt]                                     | Optional          | Builds SBT-based applications from source.                                                                        |
| [Paketo Executable JAR Buildpack][bp/executable-jar]               | Optional          | Contributes a process Type that launches an executable JAR.                                                       |
| [Paketo Spring Boot Buildpack][bp/spring-boot]                     | Optional          | Contributes configuration and metadata to Spring Boot applications.                                               |
| [Paketo Native Image Buildpack][bp/native-image]                   | **Required**      | Creates a native image from a JVM application.                                                                    |
| [Paketo Procfile Buildpack][bp/procfile]                           | Optional          | Allows the application to define or redefine process types with a [Procfile][procfiles]                           |
| [Paketo Environment Variables Buildpack][bp/environment-variables] | Optional          | Contributes arbitrary user-provided environment variables to the image.                                           |
| [Paketo Image Labels Buildpack][bp/image-labels]                   | Optional          | Contributes OCI-specific and arbitrary user-provided labels to the image.                                         |
 -->
| Buildpack                                                          | 必須/任意 | 役割                                                                                                                      |
| ------------------------------------------------------------------ | --------- | ------------------------------------------------------------------------------------------------------------------------- |
| [Paketo CA Certificates Buildpack][bp/ca-certificates]             | 任意      | ビルド時点と実行時点の両方で、指定した CA 証明書をシステムのトラストストアへ追加します                                    |
| [Paketo GraalVM Buildpack][bp/graalvm]                             | **必須**  | GraalVM JDK と Native Image の [Substrate VM](https://www.graalvm.org/reference-manual/native-image/)をインストールします |
| [Paketo Gradle Buildpack][bp/gradle]                               | 任意      | ソースコードから Gradle アプリケーションをビルドします                                                                    |
| [Paketo Leiningen Buildpack][bp/leiningen]                         | 任意      | ソースコードから Leiningen アプリケーションをビルドします                                                                 |
| [Paketo Maven Buildpack][bp/maven]                                 | 任意      | ソースコードから Maven アプリケーションをビルドします                                                                     |
| [Paketo SBT Buildpack][bp/sbt]                                     | 任意      | ソースコードから SBT アプリケーションをビルドします                                                                       |
| [Paketo Executable JAR Buildpack][bp/executable-jar]               | 任意      | 実行可能形式の jar ファイルを起動するプロセス種類を構成します                                                             |
| [Paketo Spring Boot Buildpack][bp/spring-boot]                     | 任意      | Spring Boot アプリケーションの設定とメタデータを構成します                                                                |
| [Paketo Native Image Buildpack][bp/native-image]                   | **必須**  | JVM アプリケーションのネイティブイメージを作成します                                                                      |
| [Paketo Procfile Buildpack][bp/procfile]                           | 任意      | [Procfile][procfiles] でアプリケーションが任意のプロセス種類を定義できるようにします                                      |
| [Paketo Environment Variables Buildpack][bp/environment-variables] | 任意      | 任意の環境変数をコンテナイメージに埋め込みます                                                                            |
| [Paketo Image Labels Buildpack][bp/image-labels]                   | 任意      | OCI 専用のラベルや、任意のラベルをコンテナイメージに設定します                                                            |

<!-- buildpacks -->
[bp/ca-certificates]:https://github.com/paketo-buildpacks/ca-certificates
[bp/graalvm]:https://github.com/paketo-buildpacks/graalvm
[bp/environment-variables]:https://github.com/paketo-buildpacks/environment-variables
[bp/executable-jar]:https://github.com/paketo-buildpacks/executable-jar
[bp/gradle]:https://github.com/paketo-buildpacks/gradle
[bp/image-labels]:https://github.com/paketo-buildpacks/image-labels
[bp/java]:https://github.com/paketo-buildpacks/java
[bp/leiningen]:https://github.com/paketo-buildpacks/leiningen
[bp/maven]:https://github.com/paketo-buildpacks/maven
[bp/procfile]:https://github.com/paketo-buildpacks/procfile
[bp/sbt]:https://github.com/paketo-buildpacks/sbt
[bp/spring-boot]:https://github.com/paketo-buildpacks/spring-boot
[bp/native-image]:https://github.com/paketo-buildpacks/spring-boot-native-image

[samples]:https://github.com/paketo-buildpacks/samples

<!-- cnb references -->
[platforms]:https://buildpacks.io/docs/concepts/components/platform/

<!-- paketo docs references -->
[procfiles]:{{< ref "/docs/reference/configuration#procfiles" >}}
[java/building from source]:{{< ref "/docs/howto/java#building-from-source" >}}

<!-- other references -->
[graalvm]:https://www.graalvm.org/docs/introduction/
[graalvm native image]:https://www.graalvm.org/reference-manual/native-image/
[executable jar]:https://en.wikipedia.org/wiki/JAR_(file_format)#Executable_JAR_files
[spring native]:https://github.com/spring-projects-experimental/spring-native
[spring native releases]:https://github.com/spring-projects-experimental/spring-native/releases
