---
title: "Stacks"
weight: 430
menu:
  main:
    parent: "concepts"
aliases:
  - /docs/stacks/
---

<!-- In the Getting Started tutorial, you used the Paketo builder to build a Node.js app. One of the core pieces to Buildpacks and Builders are Stack Images. Stacks provide the buildpack lifecycle with build-time and run-time environments in the form of images. -->
チュートリアルでは、Paketo ビルダーで Node.js のアプリケーションのコンテナイメージをビルドしました。
Buildpack と Builder に欠かせない構成要素がスタック（Stack）です。
スタックは、Buildpack のライフサイクルで使用するビルド時の環境と実行時の環境を提供するコンテナイメージです。

## スタックとは何か
<!-- A `stack` consists of two images:
* `build image`: the environment in which your app is built
* `run image`: the OS layer for your app image -->
スタックは2種類のイメージで構成されています。

* `build image`: アプリケーションをビルドするための環境です
* `run image`: アプリケーションのコンテナイメージを実行するときの OS レイヤーです

<!-- For more information about `stacks`, see [buildpacks.io](https://buildpacks.io/docs/concepts/components/stack/). -->
Stack について詳しく知りたければ [buildpacks.io](https://buildpacks.io/docs/concepts/components/stack/) を参照してください。

## Paketo プロジェクトのリリースしているスタックについて
<!-- The Paketo project releases several stacks. These are: -->
Paketo プロジェクトではいくつかのスタックを公開しています。

### Tiny

#### Build Image
{{< code/copyable >}}
index.docker.io/paketobuildpacks/build:tiny-cnb
{{< /code/copyable >}}

#### Run Images
{{< code/copyable >}}
index.docker.io/paketobuildpacks/run:tiny-cnb
{{< /code/copyable >}}
{{< code/copyable >}}
gcr.io/paketo-buildpacks/run:tiny-cnb
{{< /code/copyable >}}

#### 内容
* Build: ubuntu:bionic + openssl + CA certs + compilers + shell utilities
* Run: distroless-like bionic + glibc + openssl + CA certs

### Base

#### Build Image
{{< code/copyable >}}
index.docker.io/paketobuildpacks/build:base-cnb
{{< /code/copyable >}}

#### Run Images
{{< code/copyable >}}
index.docker.io/paketobuildpacks/run:base-cnb
{{< /code/copyable >}}
{{< code/copyable >}}
gcr.io/paketo-buildpacks/run:base-cnb
{{< /code/copyable >}}

#### 内容
* Build: ubuntu:bionic + openssl + CA certs + compilers + shell utilities
* Run: ubuntu:bionic + openssl + CA certs

### Full

#### Build Image
{{< code/copyable >}}
index.docker.io/paketobuildpacks/build:full-cnb
{{< /code/copyable >}}

#### Run Images
{{< code/copyable >}}
index.docker.io/paketobuildpacks/run:full-cnb
{{< /code/copyable >}}
{{< code/copyable >}}
gcr.io/paketo-buildpacks/run:full-cnb
{{< /code/copyable >}}

#### 内容
* Build: ubuntu:bionic + many common C libraries and utilities
* Run: ubuntu:bionic + many common libraries and utilities

## Paketo スタックはいつ更新するのか

<!-- Stacks are rebuilt whenever a package is patched to fix a CVE.
For more information about CVEs, see [Common Vulnerabilities and Exposures (CVE)](https://cve.mitre.org/).
Stacks are also rebuilt weekly to ensure packages without CVEs are also up to date. -->
スタックのコンテナイメージは、使用しているパッケージに CVE の対応パッチが公開されるたびに再構築しています。
CVE について詳しくは [Common Vulnerabilities and Exposures (CVE)](https://cve.mitre.org/) を参照してください。
それ以外の場合でも、CVE のパッチ適用漏れがないよう、毎週再構築しています。

<!-- We aim to release stack updates that fix High and Critical CVEs within 48 hours of the patch release. For stack updates fixing Low and Medium CVEs, we aim to release within two weeks. -->
Paketo プロジェクトとしては、優先度が高および致命的な CVE のパッチが公開されたら 40 時間以内に対応することを目指しています。
優先度が低および中の CVE のパッチについては、2週間以内に対応することを目指しています。

<!-- **Note:** Security scanning tools might report vulnerabilities in apps even when using the latest stack. This can occur when a CVE patch is not yet available upstream or if Canonical determines that the vulnerability is not severe enough to fix. -->
**注意**：最新のスタックを使用して作成したコンテナイメージにセキュリティスキャンツールを実行すると、脆弱性を検出する場合があります。上流のパッケージ開発元から修正パッチが公開されていない場合や、Canonical 社が修正を不要だと判断した場合があるからです。

<!-- Stacks are backwards compatible. A stack can safely be upgraded to the most recent version within the major version line. If for some reason backwards compatibility is broken, it happens when a new major version is released. -->
スタックは後方互換性を保証します。
メジャーバージョンが同じなら、安全に最新のバージョンへアップグレードできます。
後方互換性を保証できない場合、新しいメジャーバージョンとしてリリースします。

## Paketo スタックが提供するセキュリティハードニングの考え方と対応
<!--
* By using Ubuntu 18.04 as the base image for our stacks, we benefit from all of the security provided by Canonical and Ubuntu. For more information, see the [Canonical web site](https://ubuntu.com/security) and the [Ubuntu wiki](https://wiki.ubuntu.com/Security/Features).
* Our automatic monitoring and patching of CVEs means that our stacks are often updated within hours of Canonical's patches.
* The stack images are run as a dedicated non-root user when building and running applications.
* Each stack image has detailed metadata describing the image's components, such as the base operating system and packages.
* Each stack has separate images for building and running applications. The packages on the runtime image are curated to exclude compilers and other tools that might pose security risks.
 -->
* スタックの起源イメージとして Ubuntu 18.04 を使用しています。つまり、Canonical 社の提供する安全性に依拠しているということです。Canonical 社のセキュリティに対する考え方と対応については [Web サイト](https://ubuntu.com/security) や [Ubuntu Wiki](https://wiki.ubuntu.com/Security/Features) を参照してください
* CVE の監視と修正パッチの適用を自動化しているため、基本的には Canonical 社が修正パッチを公開してから1時間以内に反映できるようになっています
* スタックは、独立した非 root ユーザーでアプリケーションをビルドしたり実行したりするようになっています
* スタックには、使用しているコンポーネント（起源OSやパッケージなど）に関するメタ情報が含まれています
* アプリケーションをビルドするイメージと、アプリケーションを実行するイメージは独立しています。アプリケーションを実行するイメージは、セキュリティリスクになる可能性があるため、各種ツールやコンパイラのパッケージを取り除いてあります

<!-- [Github Repo](https://github.com/paketo-buildpacks/stacks) -->
[Paketo Stack の GitHub リポジトリ]((https://github.com/paketo-buildpacks/stacks))