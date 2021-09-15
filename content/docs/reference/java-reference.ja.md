---
title: "Java Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: java-reference
    name: "Java Buildpack"
---

{{% reference_exec_summary bp_name="Paketo Java Buildpack" bp_repo="https://github.com/paketo-buildpacks/java" howto_docs_path="/docs/howto/java" %}}

<!-- The [Paketo Java Buildpack][bp/java] allows users to create an image containing a JVM application from a precompiled artifact or directly from source. -->
[Paketo Java Buildpack][bp/java] はコンパイル済みのアーティファクトやソースコードから JVM アプリケーションのコンテナイメージを作成します。

<!-- The Java Buildpack is a [composite buildpack][composite buildpack] and each step in a build is handled by one of its [components](#components). For a full set of configuration options and capabilities see the homepages for the component buildpacks. -->
Java Buildpack は [合成 Buildpack][paketo/composite-buildpack] です。
ビルドのそれぞれの処理を対応する [コンポーネント Buildpack](#components) が実行します。
全ての設定項目や機能については、それぞれのコンポーネント Buildpack のホームページを参照してください。

## 使用できる JVM

<!-- The Java Buildpack uses the [BellSoft Liberica][liberica] implementations of the JRE and JDK. JVM installation is handled by the [BellSoft Liberica Buildpack][bp/bellsoft-liberica]. The JDK will be installed in the build container but only the JRE will be contributed to the application image. -->
Java Buildpack は [Bellsoft Liberica][liberica] の JRE および JDK を使用します。
JVM をインストールするのは [BellSoft Liberica Buildpack][bp/bellsoft-liberica] です。
JDK をインストールするのはビルド用のコンテナだけで、最終的なコンテナイメージには JRE をインストールします。

<!-- See the [homepage][bp/bellsoft-liberica] for the Bellsoft Liberica Buildpack for a full set of configuration options. -->
Bellsoft Liberica Buildpack の全ての設定項目は [Bellsoft Liberica Buildpack のドキュメント][bp/bellsoft-liberica] を参照してください。

## メモリ計算機

<!-- The Java Buildpack installs a component called the Memory Calculator which will configure JVM memory based on the resources available to the container at runtime. The calculated flags will be appended to `JAVA_TOOL_OPTIONS`. -->
Java Buildpack は、コンテナイメージを実行するときに使用できるメモリサイズから、JVM の使用するメモリサイズを計算するメモリ計算機（Memory Calculator）というコンポーネントをインストールします。
計算した結果は環境変数 `JAVA_TOOL_OPTIONS` へ設定します。

## Spring Boot アプリケーション

<!-- If the application uses Spring Boot the [Spring Boot Buildpack][bp/spring-boot] will enhance the resulting image by adding additional metadata to the image config,
applying Boot-specific performance optimizations, and enabling runtime auto-configuration. -->
アプリケーションが Spring Boot を使用していることを検出すると、[Spring Boot Buildpack][bp/spring-boot] は最終的なコンテナイメージに Spring Boot 特有の最適化や動的な自動構成プロパティを有効化するための情報をメタデータとして追加します。

### 追加するメタデータ

<!-- The Spring Boot Buildpack adds the following additional image labels: -->
Spring Boot Buildpack は次のようなラベルを追加します。
<!--
* `org.opencontainers.image.title` - set to the value of `Implementation-Title` from  `MANIFEST.MF`.
* `org.opencontainers.image.version` - set to the values of `Implementation-Version` from `MANIFEST.MF`.
* `org.springframework.boot.version` - set to the value of `Spring-Boot-Version` from `MANIFEST.MF`.
* `org.springframework.cloud.dataflow.spring-configuration-metadata.json` - containing [configuration metadata][spring boot configuration metadata].
* `org.springframework.cloud.dataflow.spring-configuration-metadata.json` - containing `dataflow-configuration-metadata.properties`, if present.
 -->
* `org.opencontainers.image.title` - `META-INF/MANIFEST.MF` の `Implementation-Title` を設定します
* `org.opencontainers.image.version` - `META-INF/MANIFEST.MF` の `Implementation-Version` を設定します
* `org.springframework.boot.version` - `META-INF/MANIFEST.MF` の `Spring-Boot-Version` を設定します
* `org.springframework.boot.spring-configuration-metadata.json`
  - `META-INF/dataflow-configuration-metadata.properties` が存在するときに設定します
  - アプリケーションと、`META-INF/MANIFEST.MF` の `Spring-Boot-Lib` で指定したディレクトリに存在するいずれかの jar ファイルについて、`META-INF/spring-configuration-metadata.json` の内容を統合、加工した JSON 形式の文字列を設定します
  - `META-INF/spring-configuration-metadata.json` containing [configuration metadata][spring boot configuration metadata].
* `org.springframework.cloud.dataflow.spring-configuration-metadata.json`
  - `META-INF/dataflow-configuration-metadata.properties` が存在するときに設定します
  - `META-INF/dataflow-configuration-metadata.properties` の内容を加工した JSON 形式の文字列を設定します

<!-- In addition, the buildpack will add an entry with name `dependencies` to the Bill-of-Materials listing the application dependencies. -->
また、アプリケーションの依存ライブラリの情報を BOM の `dependencies` へ設定します。

### 最適化

<!-- The Spring Boot Buildpack can apply domain-specific knowledge to optimize the performance of Spring Boot applications. For example, if the buildpack detects that the application is a reactive web application the thread count will be reduced to `50` from a default of `250`. -->
Spring Boot Buildpack は Spring Boot アプリケーションに特有の最適化を適用できます。
例えば、Buildpack の検出したアプリケーションがリアクティブ Web アプリケーションなら、最大スレッド数を初期値の `250` から `50` へ減らします。

## コンポーネント

<!-- The following component buildpacks compose the Java Buildpack. Buildpacks are listed in the order they are executed. -->
Java Buildpack の使用するコンポーネント Buildpack は次のとおりです。
一覧に並んだ順で実行します。
<!--
| Buildpack                                                                    | Required/Optional | Responsibility                                                                                  |
| ---------------------------------------------------------------------------- | ----------------- | ----------------------------------------------------------------------------------------------- |
| [Paketo CA Certificates Buildpack][bp/ca-certificates]                       | Optional          | Adds CA certificates to the system truststore at build and runtime.                             |
| [Paketo BellSoft Liberica Buildpack][bp/bellsoft-liberica]                   | **Required**      | Provides the JDK and/or JRE.                                                                    |
| [Paketo Gradle Buildpack][bp/gradle]                                         | Optional          | Builds Gradle-based applications from source.                                                   |
| [Paketo Leiningen Buildpack][bp/leiningen]                                   | Optional          | Builds Leiningen-based applications from source.                                                |
| [Paketo Maven Buildpack][bp/maven]                                           | Optional          | Builds Maven-based applications from source.                                                    |
| [Paketo SBT Buildpack][bp/sbt]                                               | Optional          | Builds SBT-based applications from source.                                                      |
| [Paketo Executable JAR Buildpack][bp/executable-jar]                         | Optional          | Contributes a process Type that launches an executable JAR.                                     |
| [Paketo Apache Tomcat Buildpack][bp/apache-tomcat]                           | Optional          | Contributes Apache Tomcat and a process type that launches a WAR with Tomcat.                   |
| [Paketo DistZip Buildpack][bp/dist-zip]                                      | Optional          | Contributes a process type that launches a DistZip-style application.                           |
| [Paketo Spring Boot Buildpack][bp/spring-boot]                               | Optional          | Contributes configuration and metadata to Spring Boot applications.                             |
| [Paketo Procfile Buildpack][bp/procfile]                                     | Optional          | Allows the application to define or redefine process types with a [Procfile][procfiles]         |
| [Paketo Azure Application Insights Buildpack][bp/azure-application-insights] | Optional          | Contributes the Application Insights Agent and configures it to connect to the service.         |
| [Paketo Debug Buildpack][bp/debug]                                           | Optional          | Configures debugging for JVM applications.                                                      |
| [Paketo Google Stackdriver Buildpack][bp/google-stackdriver]                 | Optional          | Contributes Stackdriver agents and configures them to connect to the service.                   |
| [Paketo JMX Buildpack][bp/jmx]                                               | Optional          | Configures JMX for JVM applications.                                                            |
| [Paketo Encrypt At Rest Buildpack][bp/encrypt-at-rest]                       | Optional          | Encrypts an application layer and contributes a profile script that decrypts it at launch time. |
| [Paketo Environment Variables Buildpack][bp/environment-variables]           | Optional          | Contributes arbitrary user-provided environment variables to the image.                         |
| [Paketo Image Labels Buildpack][bp/image-labels]                             | Optional          | Contributes OCI-specific and arbitrary user-provided labels to the image.                       |
 -->
| Buildpack                                                                    | 必須/任意 | 役割                                                                                     |
| ---------------------------------------------------------------------------- | ----------| ---------------------------------------------------------------------------------------- |
| [Paketo CA Certificates Buildpack][bp/ca-certificates]                       | 任意      | ビルド時点と実行時点の両方で、指定した CA 証明書をシステムのトラストストアへ追加します   |
| [Paketo BellSoft Liberica Buildpack][bp/bellsoft-liberica]                   | **必須**  | JDK あるいは JRE をインストールします                                                    |
| [Paketo Gradle Buildpack][bp/gradle]                                         | 任意      | ソースコードから Gradle アプリケーションをビルドします                                   |
| [Paketo Leiningen Buildpack][bp/leiningen]                                   | 任意      | ソースコードから Leiningen アプリケーションをビルドします                                |
| [Paketo Maven Buildpack][bp/maven]                                           | 任意      | ソースコードから Maven アプリケーションをビルドします                                    |
| [Paketo SBT Buildpack][bp/sbt]                                               | 任意      | ソースコードから SBT アプリケーションをビルドします                                      |
| [Paketo Executable JAR Buildpack][bp/executable-jar]                         | 任意      | 実行可能形式の jar ファイルを起動するプロセス種類を構成します                            |
| [Paketo Apache Tomcat Buildpack][bp/apache-tomcat]                           | 任意      | war ファイルを配置した Tomcat を起動するプロセス種類を構成します                         |
| [Paketo DistZip Buildpack][bp/dist-zip]                                      | 任意      | 配布形式のzipアーカイブになっているアプリケーションを起動するプロセス種類を構成します    |
| [Paketo Spring Boot Buildpack][bp/spring-boot]                               | 任意      | Spring Boot アプリケーションの設定とメタデータを構成します                               |
| [Paketo Procfile Buildpack][bp/procfile]                                     | 任意      | [Procfile][procfiles] でアプリケーションが任意のプロセス種類を定義できるようにします     |
| [Paketo Azure Application Insights Buildpack][bp/azure-application-insights] | 任意      | Azure Application Insight の Java エージェントと接続情報、資格情報を構成します           |
| [Paketo Debug Buildpack][bp/debug]                                           | 任意      | デバッグ接続のための設定を構成します                                                     |
| [Paketo Google Stackdriver Buildpack][bp/google-stackdriver]                 | 任意      | Google Stackdriver の Java エージェントと接続情報、資格情報を構成します                  |
| [Paketo JMX Buildpack][bp/jmx]                                               | 任意      | JMX 接続のための設定を構成します                                                         |
| [Paketo Encrypt At Rest Buildpack][bp/encrypt-at-rest]                       | 任意      | 暗号化したアプリケーションレイヤーを、起動時に復号するプロファイルスクリプトを構成します |
| [Paketo Environment Variables Buildpack][bp/environment-variables]           | 任意      | 任意の環境変数をコンテナイメージに埋め込みます                                           |
| [Paketo Image Labels Buildpack][bp/image-labels]                             | 任意      | OCI 専用のラベルや、任意のラベルをコンテナイメージに設定します                           |

<!-- buildpacks -->
[bp/apache-tomcat]:https://github.com/paketo-buildpacks/apache-tomcat
[bp/azure-application-insights]:https://github.com/paketo-buildpacks/azure-application-insights
[bp/bellsoft-liberica]:https://github.com/paketo-buildpacks/bellsoft-liberica
[bp/amazon-corretto]:https://github.com/paketo-buildpacks/amazon-corretto
[bp/azul-zulu]:https://github.com/paketo-buildpacks/azul-zulu
[bp/eclipse-openj9]:https://github.com/paketo-buildpacks/eclipse-openj9
[bp/graalvm]:https://github.com/paketo-buildpacks/graalvm
[bp/dragonwell]:https://github.com/paketo-buildpacks/graalvm
[bp/microsoft]:https://github.com/paketo-buildpacks/graalvm
[bp/sap-machine]:https://github.com/paketo-buildpacks/sap-machine
[bp/ca-certificates]:https://github.com/paketo-buildpacks/ca-certificates
[bp/debug]:https://github.com/paketo-buildpacks/debug
[bp/dist-zip]:https://github.com/paketo-buildpacks/dist-zip
[bp/encrypt-at-rest]:https://github.com/paketo-buildpacks/encrypt-at-rest
[bp/environment-variables]:https://github.com/paketo-buildpacks/environment-variables
[bp/executable-jar]:https://github.com/paketo-buildpacks/executable-jar
[bp/google-stackdriver]:https://github.com/paketo-buildpacks/google-stackdriver
[bp/gradle]:https://github.com/paketo-buildpacks/gradle
[bp/image-labels]:https://github.com/paketo-buildpacks/image-labels
[bp/java]:https://github.com/paketo-buildpacks/java
[bp/jmx]:https://github.com/paketo-buildpacks/jmx
[bp/leiningen]:https://github.com/paketo-buildpacks/leiningen
[bp/maven]:https://github.com/paketo-buildpacks/maven
[bp/procfile]:https://github.com/paketo-buildpacks/procfile
[bp/sbt]:https://github.com/paketo-buildpacks/sbt
[bp/spring-boot]:https://github.com/paketo-buildpacks/spring-boot


<!-- other references -->
[liberica]:https://bell-sw.com/
[composite buildpack]:{{< ref "docs/concepts/buildpacks#component-buildpacks" >}}
