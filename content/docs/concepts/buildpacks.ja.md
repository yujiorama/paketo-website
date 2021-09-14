---
title: "Buildpacks"
weight: 400
menu:
  main:
    parent: "concepts"
aliases:
  - /docs/buildpacks/
---

<!-- In the Getting Started tutorial, you ran a `pack build` command to build a sample app. This resulted in some output similar to this block: -->
チュートリアルでコンテナイメージを作成するために実行した `pack build` コマンドは、次のような出力をしていたはずです。

```
...
===> DETECTING
paketo-buildpacks/node-engine 0.1.1
paketo-buildpacks/npm-install 0.2.0
paketo-buildpacks/npm-start   0.0.2
...
===> BUILDING
Paketo Node Engine Buildpack 0.1.1
...
Paketo NPM Install Buildpack 0.2.0
...
Paketo NPM Start Buildpack 0.0.2
...
```

<!-- In this section, we will make sense of this output and explain how the
buildpacks detect what dependencies are needed by your app to build it into a runnable app image. -->
このドキュメントでは出力している内容の意味を説明し、Buildpack が実行可能なコンテナイメージをビルドするために必要な依存対象をどのように検出しているのか説明します。

## Buildpack とは何か
<!-- Buildpacks examine your app source code, identify and gather dependencies, and
output OCI compliant app and dependency layers. -->
Buildpack はアプリケーションのソースコードを調査し、依存対象を特定、収集します。
そして、依存対象をレイヤーとして積み重ねた、OCI に準拠したコンテナイメージを作成します。

<!-- **Paketo buildpacks provide language runtime support for your favorite
languages.** -->
**Paketo Buildpack は利用者の要求するプログラミング言語のランタイムを提供します**

<!-- Each buildpack is a modular unit, responsible for providing a single
dependency. Multiple implementation buildpacks come together to provide all of
your app's dependencies. -->
それぞれの Buildpack は単一の依存対象を提供する、独立したモジュールです。
複数の Buildpack を組み合わせることで、アプリケーションの要求する全ての依存対象を提供します。

<!-- A buildpack operates on your source code in two phases: **detect** and
**build**. -->
Buildpack はアプリケーションのソースコードを **検出（detect）** と **構築（build）** の2段階のフェーズで処理します。

### 検出フェーズ
<!-- In the `detect` phase, the buildpack looks for indicators in
your source code to determine whether or not it needs to be included to build your app. -->
`detect` フェーズでは、アプリケーションのコンテナイメージに含めなければならない依存対象を特定する印を発見するため、ソースコードを調査します。

<!-- In the Getting Started tutorial, you can see in the output that
`paketo-buildpacks/npm-install` is used. This is because the NPM Install
Buildpack's detection criteria looks for a `package.json` file in the app
source code. Since it's present in the sample app we used, detection passes on
the NPM Install Buildpack. -->
チュートリアルで実行した `pack build` コマンドの出力より、`paketo-buildpacks/npm-install` が使われていることが分かるでしょう。
これは、NPM Install Buildpack がソースコードから `package.json` ファイルを発見するようになっているからです。
これを、検出基準（detection criteria）と呼びます。
サンプルリポジトリのアプリケーションには `package.json` が含まれているため、NPM Install Buildpack の検出は成功したのです。

<!-- Different buildpacks have different detection criteria according to the
dependencies they are responsible for. Once detection has passed for a
buildpack, the buildpack returns a contract of what it requires, and what it
will provide to the subsequent `build` phase. -->
それぞれの Buildpack は担当する依存対象に応じた独自の検出基準を実装しています。
検出に成功した Buildpack は、その Buildpack が必要とする依存対象（requires）と、後に続く `build` フェーズへ提供する依存対象（provides）からなる契約（contract）を返します。

### 構築フェーズ
<!-- In the `build` phase, the buildpack contributes to the final
app image, fulfilling the contract given by the `detect` phase. These
contributions could be adding an image layer containing a dependency binary
(like the Node.js engine) or could be as simple as a running a command (like
`npm install`). -->
`build` フェーズでは、`detect` フェーズに与えられた契約を満たすことで、最終的なコンテナイメージの構築に参加します。
コンテナイメージに、依存対象の実行可能ファイル（例えば Node.js のエンジン）を含むレイヤーを追加する場合もありますし、単純にコマンドを実行するだけレイヤーを追加する場合もあります（例えば `npm install`）。

<!-- In the Getting Started tutorial, the `pack build` output contains a section in the
build phase for the NPM Install Buildpack under a "BUILDING" header. You can
see that the buildpack runs `npm install` to install the app's dependencies.
Subsequently, the NPM Start Buildpack sets the start command to `node server.js`. -->
チュートリアルで実行した `pack build` コマンドの出力には、"BUILDING" ヘッダーの後に、NPM Install Buildpack が構築フェーズで実行した処理の出力が含まれています。
具体的には、アプリケーションの依存ライブラリをインストールするため `npm install` コマンドを実行しています。
その後、NPM Start Buildpack が `ndoe server.js` をコンテナイメージの起動コマンドに設定しているのが分かります。

<!-- The image below illustrates how buildpacks contribute layers to the final
runnable app image: -->
次の図は、最終的なコンテナイメージを構成するレイヤーに、Buildpack がどのように寄与しているのか示しています。

![Final app image](/images/docs-buildpacks-app-image.png)

<!-- For more information about buildpacks, visit
[buildpacks.io](https://buildpacks.io/docs/concepts/components/buildpack/) -->
Buildpack について詳しく知りたければ [buildpacks.io](https://buildpacks.io/docs/concepts/components/buildpack/) を参照してください。

### コンポーネント Buildpacks
<!-- Paketo provides many component buildpacks, each with a well-defined
responsibility. Component buildpacks may require contributions from upstream
buildpacks and/or provide required components to downstream buildpacks. -->
Paketo は明確な責務を定義したさまざまなコンポーネントとしての Buildpack を提供しています。
コンポーネント Buildpack は、上流の Buildpack が生成したレイヤー（貢献）を必要とする場合もあるし、下流の Buildpack が必要とするレイヤー（貢献）を生成する場合もあります。

<!-- For example, the Gradle Buildpack is a component buildpack, responsible for
installing Gradle in the build container and using Gradle to compile and
package a JVM application. It requires that an upstream component to provide a
JDK. It provides a compiled JVM application to downstream buildpacks. -->
例えば、Gradle Buildpack は、ビルド用のコンテナに Gradle をインストールして、Gradle で JVM アプリケーションのパッケージをコンパイルする役割を担当する コンポーネント Buildpack です。
上流の Buildpack が提供する JDK を必要とし、下流の Buildpack にコンパイル済みの JVM アプリケーションを提供します。

### 合成 Buildpack

<!-- Component buildpacks can be combined to compose higher-level composite
buildpacks. Composite buildpacks contain an ordered list of component
buildpacks. Some buildpacks in the ordering may be optional, participating only
when they detect that they are needed. -->
複数のコンポーネント Buildpack を合成して、より高水準の Buildpack を構成できます。
合成 Buildpack に含まれるのは、コンポーネント Buildpack の順序付きリストです。
順序が未定義の Buildpack もありますし、特定の依存対象を検出した場合だけ活性化する Buildpack もあります。

## Paketo Buildpack はどのように連携しているのか
<!-- The Paketo language family buildpacks are [composite
buildpacks](#composite-buildpacks) that provide easy out-of-the-box support the
most popular language runtimes and app configurations. These buildpacks combine
multiple component buildpacks into ordered groupings. The groupings satisfy
each buildpack's requirements (mentioned in the  `detect` section). -->
プログラミング言語別の Paketo Buildpack は [合成 Buildpack](#composite-buildpacks) です。
ほとんどの一般的なプログラミング言語のランタイムやアプリケーションの構成について、ダウンロードした状態そのままで利用できるようになっています。
それぞれの Buildpack は、複数のコンポーネント Buildpack を順序付きのグループとして構成しています。
それぞれのグループの満足条件は、グループに所属する全ての Buildpack の要求を満たすことです（詳しくは検出フェーズで説明しています）。`

## Paketo Buildpack と Cloud Native Buildpack プロジェクトはどのような関係なのか

<!-- Paketo Buildpacks implement the Buildpack API described in the [Cloud Native Buildpacks
Specification](https://github.com/buildpacks/spec). The `build` and `detect`
phases of Paketo Buildpacks are designed to be run by the [CNB
lifecycle](https://buildpacks.io/docs/concepts/components/lifecycle/). -->
Paketo Buildpack は [Cloud Native Buildpack の仕様](https://github.com/buildpacks/spec) として定義された Buildpack API を実装しています。
Paketo Buildpack の `build` フェーズと `detect` フェーズは、[CNB のライフサイクル](https://buildpacks.io/docs/concepts/components/lifecycle/) で実行できるように設計されています。

## Paketo Buildpack と Cloud Foundry Buildpack は何が違うのか
<!--
| Paketo Buildpacks | CF Buildpacks |
| ------------------- | -------------- |
| Produce OCI images that can be **deployed anywhere** | Produce droplets that must be deployed with CF |
| Each language buildpack is made up of **small, composable components** | Each language buildpack is one monolithic codebase |
| **Easy to add features** by writing your own buildpack | Must fork a buildpack to extend features |
 -->
| Paketo Buildpacks   | CF Buildpacks |
| ------------------- | -------------- |
| **どんな環境でも** デプロイできる OCI イメージを作成します | CF にデプロイするための droplet を作成します |
| **小規模で、合成可能なコンポーネント** として開発されています | 単一のモノリシックなコードベースです |
| 独自の Buildpack を作成して **簡単に機能追加できます** | 機能追加するには Buildpack をフォークしなければなりません |

