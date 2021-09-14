---
title: "Bill of Materials"
weight: 440
menu:
  main:
    parent: "concepts"
---

<!-- In the Getting Started tutorial, you used the Paketo builder to build a Node.js app. Once you have an app image, you can access metadata about all of the dependencies present in the final app image using the bill of materials.
チュートリアルでは、Paketo ビルダーで Node.js のアプリケーションのコンテナイメージをビルドしました。 -->
部品表（bill of materials）を使用すると、作成したコンテナイメージの使用する全ての依存ライブラリに関する情報をメタデータとして参照できます。

## 部品表（bill of materials）とは何か
<!--
A bill of materials (BOM) is an industry standard mechanism of surfacing metadata about dependencies in images or applications. The metadata consists of various fields such as:
* `version`: the dependency version
* `uri`: URI to compiled dependency
* `checksum`: a CycloneDX-supported hash algorithm (such as SHA-256) and value of the dependency
* [`licenses`][LICENSE]: dependency licenses in SPDX format
* `deprecation-date`: dependency deprecation date
* `source uri`: URI to upstream source dependency
* `source checksum`: a CycloneDX-supported hash algorithm and value of the upstream source dependency
* [`CPE`][CPE]: common platform enumeration
* [`pURL`][PURL]: package URL
 -->
部品表（BOM：bill of materials）とは、コンテナイメージやアプリケーションの依存対象に関する情報をメタデータとして公開する、業界標準の仕組みのことです。
メタデータは次のようなフィールドで構成されています。

* `version`: 依存対象のバージョン
* `uri`: コンパイル済みファイルへアクセスできる URI
* `checksum`: [CycloneDX][format/cyclonedx] が対応しているハッシュアルゴリズム（SHA-256 など）、および、依存対象に関するチェックサムの値
* [`licenses`][LICENSE]: 依存対象の、[SPDX][format/spdx] 形式のライセンス
* `deprecation-date`: 依存対象（のバージョン）が非推奨になる日時
* `source uri`: 依存対象の上流の元データへアクセスできる URI
* `source checksum`: [CycloneDX][format/cyclonedx] が対応しているハッシュアルゴリズム（SHA-256 など）、および、依存対象の上流の元データに関するチェックサムの値
* [`CPE`][CPE]: 共通するプラットフォームの一覧
* [`pURL`][PURL]: パッケージの URL

## BOM が役に立つ理由

<!-- The information from the bill of materials is largely used to help understand the dependencies involved in your app's secure software supply chain. -->
BOM の情報は、主に、アプリケーションのセキュリティサプライチェインに含まれる依存対象を理解するのに役立ちます。

### 脆弱性スキャン
<!-- The bill of materials can be passed to one of the many existent vulnerability scanning tools, such as [Dependency Track][tool/dependency-track] and [Trivy][tool/trivy], in order to identify vulnerabilities. -->
世の中には、BOM を入力として使用できるさまざまな脆弱性スキャンツールがあります。
例えば、[Dependency Track][tool/dependency-track] や [Trivy][tool/trivy] で脆弱性を発見できます。

<!-- The BOM contains two fields that are primarily concerned with vulnerability identification: -->
主に脆弱性を特定するために使用するフィールドが CPE と pURL です。

#### CPEs
<!-- CPEs, or common platform enumerations, are standard notation to look up dependency version-specific vulnerabilities and related patches in the [NIST National Vulnerabilty Database][NIST]. -->
共通プラットフォーム一覧（CPE：common platform enumeration）は、[NIST National Vulnerabilty Database][NIST] で依存対象の特定のバージョンに関する脆弱性と修正パッチを検索できる標準的な記法です。

#### pURLs
<!-- PURLs, or package URLs are [a universal representation][PURL definition] of package location regardless of vendor, project, or ecosystem. -->
pURL はベンダーやプロジェクトやエコシステムに依存しない、[汎用的なパッケージ URL 表現][PURL definition] です。

### コンプライアンスチェック
<!-- The inclusion of license information in the bill of materials helps with application legal compliance by providing the information in a consumable way for each dependency involved with your application image. -->
BOMのライセンス情報を使うと、アプリケーションの法律面のコンプライアンスを確認するのに便利です。
コンテナイメージに含まれるそれぞれの依存対象のライセンス情報を使いやすい形式で取得できます。

#### ライセンス
<!-- The licenses in the Paketo BOM are obtained from license scanning tools. Due to the unstandardized nature of license inclusion in software, the detection tools assign "confidence scores" to each license. We include **every license** discovered by the scanning tools in the BOM, regardless of the "confidence score" that the tool has provided, to avoid risk of missing an important license. Because of this feature, advanced compliance checking may be required to filter out false positive licenses. -->
Paketo BOM のライセンス情報は、ライセンス検出ツールで収集したものです。
ソフトウェアにライセンス情報を含める方法は標準化されていないので、検出ツールは発見したライセンスに「信頼スコア」を割り当てます。
Paketo プロジェクトでは、重要なライセンス情報を取りこぼすリスクを回避するため、「信頼スコア」の値に関わらず、検出ツールが発見した **全てのライセンス情報** を  BOM へ登録します。
そのため、詳細なコンプライアンスチェックをするときは、間違って発見した（偽陽性の）ライセンス情報を取り除かなければなりません。

## Paketo BOM の出力形式

<!-- Buildpacks provide a bill of materials by adding metadata to the app image when it is built. Paketo buildpacks add entries to the bill of materials as `JSON` objects with the following schema: -->
Buildpack は作成したコンテナイメージのメタデータに BOM を追加します。
Paketo Buildpack は次のような `JSON` 形式の情報を追加します。

```plain
{
  "name": <name of the dependency>,
  "metadata": {
    "checksum": {
      "algorithm": <CycloneDX-supported hash algorithm ('MD5', 'SHA-1', 'SHA-256', 'SHA-384', 'SHA-512', 'SHA3-256', 'SHA3-384', 'SHA3-512', 'BLAKE2b-256', 'BLAKE2b-384', 'BLAKE2b-512', 'BLAKE3')>,
      "hash": <hash of the dependency>
    },
    "cpe": <dependency/version specific common platform enumeration>,
    "deprecation-date": <date of package deprecation>,
    "licenses": <[list of all licensesn SPDX format]>,
    "purl": <dependency/version specific package URL>,
    "source": {
      "checksum": {
        "algorithm": <CycloneDX-supported hash algorithm ('MD5', 'SHA-1', 'SHA-256', 'SHA-384', 'SHA-512', 'SHA3-256', 'SHA3-384', 'SHA3-512', 'BLAKE2b-256', 'BLAKE2b-384', 'BLAKE2b-512', 'BLAKE3')>,
        "hash": <hash of the dependency>
      },
      "uri": <package upstream source URI>
    },
    "uri": "<compiled package URI>",
    "version": <dependency version>
  }
}
```

<!-- Paketo buildpacks generate two main types of BOM entries: _buildpack entries_ and _language module entries_. To help explain these types, we will use as an example the bill of materials of the app in the [How to Access the Bill of Materials guide][howto/access-bom]. -->
Paketo Buildpack は __Buildpack__ と __言語モジュール__ という2種類の主要な BOM 要素を出力します。
ここでは、「[部品表（Bill of Materials）へアクセスする][howto/access-bom]」で作成したコンテナイメージの BOM についてそれぞれの内容を説明します。

### Buildpack 要素

<!-- A buildpack entry is an entry for a dependency that a Paketo buildpack installs directly (i.e. _without_ using a dependency manager). Examples include: a JVM, the .NET runtime, or the Node.js runtime. The buildpacks generate these entries using metadata obtained during the construction of the dependency itself. -->
Buildpack 要素は、Paketo Buildpack が依存性管理ツールを _使わずに_ 直接インストールする依存対象に関する情報です。
具体的には、JVM や .NET Runtime、Node.js ランタイムなどが挙げられます。
Buildpack はこれらの依存対象を構築する途中で得られたメタデータを使用して要素を生成します。

<!-- A buildpack entry contains the version, vulnerability identifiers (CPE and pURL), all potential licenses, checksums for both the source and the compiled dependency, source information, and the name of the buildpack that installed the dependency. -->
Buildpack 要素に含まれるのは依存対象に関するバージョン、脆弱性識別子（CPE や pURL）、使用していると考えられるライセンス、元データおよびコンパイル済みデータのチェックサム、元データの情報、名前です。

<!-- Below, see an example of the buildpack entry for the Node.js runtime installed by the [Paketo Node Engine buildpack][bp/node-engine]. -->
次の例は、[Paketo Node Engine buildpack][bp/node-engine] のインストールする Node.js ランタイムに関する Buildpack 要素です。

```json
{
  "name": "Node Engine",
  "metadata": {
    "checksum": {
      "algorithm": "SHA-256",
      "hash": "a50ee095f936b51fffe5c032a7377a156723145c1ab0291ccc882f04719f1b54"
    },
    "cpe": "cpe:2.3:a:nodejs:node.js:16.7.0:*:*:*:*:*:*:*",
    "deprecation-date": "2024-04-30T00:00:00Z",
    "licenses": [
      "0BSD",
      "Apache-2.0",
      "Artistic-2.0",
      "BSD-2-Clause",
      "BSD-3-Clause",
      "BSD-3-Clause-Clear",
      "CC0-1.0",
      "MIT",
      "MIT-0",
      "Unicode-TOU"
    ],
    "purl": "pkg:generic/node@v16.7.0?checksum=0c4a82acc5ae67744d56f2c97db54b859f2b3ef8e78deacfb8aed0ed4c7cb690&download_url=https://nodejs.org/dist/v16.7.0/node-v16.7.0.tar.gz",
    "source": {
      "checksum": {
        "algorithm": "SHA-256",
        "hash": "0c4a82acc5ae67744d56f2c97db54b859f2b3ef8e78deacfb8aed0ed4c7cb690"
      },
      "uri": "https://nodejs.org/dist/v16.7.0/node-v16.7.0.tar.gz"
    },
    "stacks": [
      "io.buildpacks.stacks.bionic"
    ],
    "uri": "https://deps.paketo.io/node/node_v16.7.0_linux_x64_bionic_a50ee095.tgz",
    "version": "16.7.0"
  },
  "buildpacks": {
    "id": "paketo-buildpacks/node-engine",
    "version": "1.2.3"
  }
}
```

### 言語モジュール要素
<!-- A language module entry is an entry for a dependency that a Paketo buildpack installs using a dependency manager (e.g. Node.js module, Maven package). The buildpacks generate these entries by gathering metadata about packages after they have been installed. -->
言語モジュール要素は、Paketo Buildpack が依存性管理ツールでインストールした依存対象の情報です（Node.js モジュールや Maven パッケージなど）。
Buildpack はインスト―ルしたそれぞれのパッケージに関するメタデータを収集して要素を生成します。

<!-- A language module entry contains the version, vulnerability identifiers (pURL), all potential licenses, and the name of the buildpack that generated the entry. -->
言語モジュール要素に含まれるのは依存対象に関するバージョン、脆弱性識別子（pURL）、使用していると考えられるライセンス、名前です。

<!-- Below, see an example of a language module entry for a Node.js module. -->
次の例は、Node.js モジュールに関する言語モジュール要素です。

```json
{
  "name": "httpdispatcher",
  "metadata": {
    "licenses": [
      "MIT"
    ],
    "purl": "pkg:npm/httpdispatcher@2.1.2",
    "version": "2.1.2"
  },
  "buildpack": {
    "id": "paketo-buildpacks/node-module-bom",
    "version": "1.2.3"
  }
}
```

<!-- **Note:** The metadata that Paketo buildpacks generate does not conform to a specific industry-standard bill of materials format. Some industry-standard formats include [CycloneDX][format/cyclonedx] and [SPDX][format/spdx], both of which can be presented as `JSON` files. While the Paketo bill of materials doesn't currently conform to one of these formats, the goal is to eventually support both. -->
  **注意**：Paketo Buildpack の生成するメタデータは、業界標準のBOM形式と適合しない場合があります。業界標準の形式の中でも [CycloneDX][format/cyclonedx] や [SPDX][format/spdx] は、どちらも `JSON` 形式で表現できます。将来的にはどちらの形式にも対応する予定です。

<!-- References -->
[CPE]:{{< relref "#cpes" >}}
[PURL]:{{< relref "#purls" >}}
[LICENSE]:{{< relref "#compliance-checking" >}}

[howto/access-bom]:{{< ref "docs/howto/bom#access-the-bill-of-materials-on-a-sample-node-application" >}}

[tool/dependency-track]:https://dependencytrack.org/
[tool/trivy]:https://github.com/aquasecurity/trivy

[format/cyclonedx]:https://cyclonedx.org/
[format/spdx]:https://spdx.dev/

[bp/node-engine]:{{< bp_repo "node-engine" >}}

[NIST]:https://nvd.nist.gov/products/cpe/search
[PURL definition]:https://github.com/package-url/purl-spec
