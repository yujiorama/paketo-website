---
title: "Paketo Buildpack を作成する"
weight: 344
menu:
  main:
    parent: "howto"
aliases:
  - /docs/tutorials/create-paketo-buildpack/
---

<!-- If the entire Cloud Native Buildpack experience is new to you, you may want to
stop and take some time to read about [authoring a a Cloud Native Buildpack](https://buildpacks.io/docs/buildpack-author-guide/create-buildpack/)(CNB). Packit is a Go library that is an abstraction that conforms to the [CNB specification](https://github.com/buildpacks/spec) that takes a minimal
approach in terms of the features that are implemented giving a lot of fine
control to the buildpack author. -->
初めて Cloud Native Buildpack を使うなら、最初に [Cloud Native Buildpack(CNB) の作り方](https://buildpacks.io/docs/buildpack-author-guide/create-buildpack/) を読んでください。
Packit は [CNB の仕様](https://github.com/buildpacks/spec) を満たす Buildpack を作成するために必要な抽象を提供する Go 言語のライブラリです。

<!-- This tutorial's goal is to take you from nothing to a buildpack that puts a
dependency on the filesystem as fast as possible, so with that let's get
started! -->
このチュートリアルの目標は、ファイルシステム上に1つの依存関係を要求するだけの Buildpack を作成することです。
早速やっていきましょう。

## [Packit](https://github.com/paketo-buildpacks/packit)

[![GoDoc](https://img.shields.io/badge/pkg.go.dev-doc-blue)](https://pkg.go.dev/github.com/paketo-buildpacks/packit)

<!-- For complete documentation of the Packit library, you can browse the godocs
linked above. In the interest of saving you time, let's talk about the three
artifacts that will need to be present in our final built buildpack. In the end
we will need a `buildpack.toml` file, a `bin/detect` binary, and a `bin/build`
binary for this buildpack to be CNB compliant. -->
Packit の完全なドキュメントは手前のリンクから参照してください。
読者の時間を無駄にさせないためにも、ここではこれから作成する Buildpack に含める3種類のアーティファクトについて説明します。
CNB の仕様を満たすために最低限必要なのは、 `buildpack.toml` と `bin/detect` と `bin/build` です。

## 前提条件
<!-- You will need the following tools installed on your machine to aid you in
building and testing your buildpack. -->
Buildpack をビルドし、テストするには、ローカルPCに次のようなツールが必要です。

- [Docker](https://docs.docker.com/install/)
- [Pack](https://buildpacks.io/docs/install-pack/)
- [Go](https://golang.org/doc/install)

## それでは始めましょう
<!-- For demonstration purposes we are going to build a buildpack that installs a
`nodejs` engine, which is based off the Paketo
[node-engine](https://github.com/paketo-buildpacks/node-engine). -->
[Node Engine Cloud Native Buildpack](https://github.com/paketo-buildpacks/node-engine) で Node.js エンジンをインストールするだけの Buildpack を作成する様子を説明します。

<!-- A sample repository containing all of the code is
[here](https://github.com/ForestEckhardt/tutorial-cnb). To start a brand new Go
project all you need to do is create a new directory to contain you project run
the following command: -->
[サンプルリポジトリ](https://github.com/ForestEckhardt/tutorial-cnb) には全てのソースコードが残っています。
新しい Go 言語のプロジェクトを作成するには、ディレクトリを作成して、そこで次のコマンドを実行します。

{{< code/copyable >}}
go mod init </path/to/project>
{{< /code/copyable >}}

### `buildpack.toml`

<!-- The `buildpack.toml` file should contain the following: -->
`buildpack.toml` を作成します。
次のような内容を記述しましょう。

{{< code/copyable >}}
# ライフサイクルと互換性のあるバージョンを定義します
api = "0.2"

# ライフサイフルに対する Buildpack の基本的なメタデータを記述します
[buildpack]
  id = "com.example.node-engine"
  name = "Node Engine Buildpack"
  version = "0.0.1"

# Buildpack と互換性のあるスタックを記述します
[[stacks]]
  id = "io.buildpacks.stacks.bionic"
{{< /code/copyable >}}

### `bin/detect`
<!-- **Note**: For a complete rundown on how the Detect Phase works you can read the
`Detect Phase` section in the [packit godocs](https://pkg.go.dev/github.com/paketo-buildpacks/packit). -->
**注意**：検出フェーズの詳細な説明は [packit の godocs](https://pkg.go.dev/github.com/paketo-buildpacks/packit) の `Detect Phase` セクションを参照してください。

<!-- In the Packit library, there is a
[`packit.Detect()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Detect)
function that consumes a
[`packit.DetectFunc`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#DetectFunc).
The `packit.DetectFunc` is created by buildpack authors and contains all of the
buildpack specific detection logic. So with that let's write some code and talk
about it. -->
Packit には [`packit.DetectFunc`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#DetectFunc) を引数とする関数 [`packit.Detect()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Detect) があります。
`packit.DetectFunc` は Buildpack の作者が実装する関数で、Buildpack に必要な全ての検出ロジックを内包する関数です。
具体的な実装を説明していきましょう。

`cmd/detect/main.go`

{{< code/copyable >}}
package main

import (
    "<path/to/project>"
    "github.com/paketo-buildpacks/packit"
)

func main() {
    packit.Detect(node.Detect())
}
{{< /code/copyable >}}

`node/detect.go`

{{< code/copyable >}}
package node

import (
    "fmt"
    "github.com/paketo-buildpacks/packit"
)

func Detect() packit.DetectFunc {
	return func(context packit.DetectContext) (packit.DetectResult, error) {
		return packit.DetectResult{}, fmt.Errorf("always fail")
	}
}
{{< /code/copyable >}}

<!-- Alright let's talk about this for a minute. With these pieces of code, we have
written a buildpack that will always detect false. The reason that we've split
out the definition of the `packit.DetectFunc` into its own package is that this
allows us to separate Packit related code from any actual business specific
logic of the buildpack. All buildpack specific business logic will go into the
`node` package from now on. This may seem small but if you plan on potentially
expanding this buildpack we highly recommend that you follow this structure. -->
この時点の Buildpack は常に検出に失敗します。
`packit.DetectFunc` を独自パッケージで実装することにしたのは、Buildpack のビジネスロジックを Packit に関連する実装と分離するためです。
つまり、Buildpack の全てのビジネスロジックは `node` パッケージで実装することになります。
将来的に Buildpack を拡張していく可能性があるなら、こういう構造にしておくことをお勧めします。

<!-- To test out progress so far we can build the detect binary ourselves with the
following command: -->
`detect` コマンドをビルドして、ここまでの状態を確かめてみましょう。

{{< code/copyable >}}
GOOS=linux go build -ldflags="-s -w" -o ./bin/detect ./cmd/detect/main.go
{{< /code/copyable >}}

<!-- Once you've done that you should be able to use the `pack` cli to try and build
a container for a `nodejs` app. For the purposes of this demonstration I will
be using the following [simple app](https://github.com/paketo-buildpacks/node-engine/tree/main/integration/testdata/simple_app)
from the `node-engine` repo linked above. The following command will allow you
to build the app: -->
そうしたら、`pack` コマンドで Node.js アプリケーションのコンテナイメージをビルドしてみましょう。
ここでは、 [Node Engine Cloud Native Buildpack の simple app](https://github.com/paketo-buildpacks/node-engine/tree/main/integration/testdata/simple_app) で試しています。

{{< code/copyable >}}
pack build <app-name> \
  --path <path/to/app> \
  --buildpack <path/to/project> \
  --builder paketobuildpacks/builder:base \
  --verbose
{{< /code/copyable >}}

{{< code/output >}}
===> DETECTING
[detector] ======== Output: com.example.node-engine@0.0.1 ========
[detector]
[detector] always fail
[detector] ======== Results ========
[detector] err:  com.example.node-engine@0.0.1 (1)
[detector] ERROR: No buildpack groups passed detection.
[detector] ERROR: Please check that you are running against the correct path.
[detector] ERROR: failed to detect: no buildpacks participating
ERROR: failed with status code: 6
{{< /code/copyable >}}

<!-- Now that we have a skeleton laid out let's make it do something more useful
than just fail all of the time. -->
骨格が出来たことを確認できたので、「常に失敗する」のをまともに動作するするようにしてみましょう。

<!-- To do that we need to dive into the concept of provides and requires. If you
want to read the buildpacks specification for provides and requires you can
look
[here](https://github.com/buildpacks/spec/blob/master/buildpack.md#phase-1-detection). -->
provides と requires という考え方に取り組まなければなりません。
詳しくは [CNB の仕様](https://github.com/buildpacks/spec/blob/master/buildpack.md#phase-1-detection) を参照してください。

<!-- **Quick summary**: A buildpack's detect binary has as part of it -->
**簡単な説明**：Buildpack の detect コマンドの構成要素
<!-- [`packit.DetectResult`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#DetectResult)
pass out a
[`packit.BuildPlan`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#BuildPlan)
in which there is a list of provisions and requirements. A buildpack can state
that it provides dependencies or that it requires dependencies or it can both
require and provide dependencies. A buildpack can only detect true if all of
its provides match up with requires from itself or a subsequent buildpacks in a
buildpack sequence and that all of is requires match up with provides from
itself or previous buildpacks in a sequence of buildpacks. -->
[`packit.DetectResult`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#DetectResult) から参照できる [`packit.BuildPlan`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#BuildPlan) は provisions と requirements のリストを持っています。
provisions と requirements は、その Buildpack が依存対象として使用できる成果物を提供するのか、依存対象を要求するのか、あるいはその両方になるのかを説明します。
Buildpack の detect コマンドが true になるのは次の両方の条件を満たした場合だけです。
1つ目の条件は、Buildpack の全ての provides が、自分自身と後に続く全ての Buildpack の requires にマッチした場合。
2つ目の条件は、Buildpack の全ての requires が、自分自身と前に並ぶ全ての Buildpack の provides にマッチした場合。

<!-- In our situation, our buildpack is the only one running in the sequence so in
order for us to pass detection we have to both provide `node` as a dependency
but we also need to require `node` as well. So let's go implement that provide
and require relationship in our detect code. -->
このチュートリアルの場合、他に実行している Buildpack はないので、detect コマンドを true にするには、依存対象として `node` を提供し、同様に `node` を必須としなければなりません。
detect コマンドに実装してみましょう。

`node/detect.go`

{{< code/copyable >}}
package node

import (
	"github.com/paketo-buildpacks/packit"
)

func Detect() packit.DetectFunc {
	return func(context packit.DetectContext) (packit.DetectResult, error) {
		return packit.DetectResult{
			Plan: packit.BuildPlan{
				Provides: []packit.BuildPlanProvision{
					{Name: "node"},
				},
				Requires: []packit.BuildPlanRequirement{
					{Name: "node"},
				},
			},
		}, nil
	}
}
{{< /code/copyable >}}

<!-- Let's do a quick rundown of what is happening now. In the `packit.DetectResult`
we are now returning a `Plan`. Inside that plan we state that this buildpack
provides the `node` dependency and that it requires the `node` dependency. If
you were to run the code that the used to build and test our code earlier you
should get something that look like this: -->
何をしているのか簡単に説明していきます。
まず、 `packit.DetectResult` のフィールド `Plan` に値を詰めるようにしました。
値は provides に依存対象として `node` を、requires に依存対象として `node` を持つオブジェクトです。
前の例のように detect コマンドをビルドして、pack コマンドを実行してみましょう。

{{< code/copyable >}}
GOOS=linux go build -ldflags="-s -w" -o ./bin/detect ./cmd/detect/main.go
pack build <app-name> \
  --path <path/to/app> \
  --buildpack <path/to/project> \
  --builder paketobuildpacks/builder:base \
  --verbose
{{< /code/copyable >}}

{{< code/output >}}
===> DETECTING
[detector] ======== Results ========
[detector] pass: com.example.node-engine@0.0.1
[detector] Resolving plan... (try #1)
[detector] com.example.node-engine 0.0.1
===> ANALYZING
[analyzer] Previous image with name "index.docker.io/library/test-node:latest" not found
===> RESTORING
===> BUILDING
[builder] ERROR: failed to build: fork/exec /cnb/buildpacks/com.example.node-engine/0.0.1/bin/build: no such file or directory
ERROR: failed with status code: 7
{{< /code/output >}}

<!-- As you can see detection now passes! However, we still get an error because we
have no `build` binary yet, so let's fix that. -->
検出フェーズ（DETECTING）を通過していることがわかりますね。
そして、`build` コマンドが存在しないせいでエラーになっています。
修正してきいきましょう。

### `bin/build`

<!-- For a complete on how the Build Phase works you can read the `Build Phase`
section in the [packit godocs](https://pkg.go.dev/github.com/paketo-buildpacks/packit), but once again I
will give you a quick overview. The Packit library has a
[`packit.Build()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Build)
function which consumes a
[`packit.BuildFunc`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#BuildFunc)
which contains all of the buildpack specific build logic and is written by us.
So lets put down some skeleton code. -->
ビルドフェーズの詳細な説明は [packit の godocs](https://pkg.go.dev/github.com/paketo-buildpacks/packit) の `Build Phase` セクションを参照してください。
Packit には [`packit.BuildFunc`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#BuildFunc) を引数とする関数 [`packit.Build()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Build) があります。
`packit.BuildFunc` は Buildpack の作者が実装する関数で、Buildpack に必要な全てのビルドロジックを内包する関数です。
スケルトンコードを作ってみましょう。

`cmd/build/main.go`

{{< code/copyable >}}
package main

import (
	"<path/to/project or github.com/<some-org or some-user>/<some-repo>>/node"
	"github.com/paketo-buildpacks/packit"
)

func main() {
	packit.Build(node.Build())
}
{{< /code/copyable >}}

`node/build.go`

{{< code/copyable >}}
package node

import (
	"fmt"

	"github.com/paketo-buildpacks/packit"
)

func Build() packit.BuildFunc {
	return func(context packit.BuildContext) (packit.BuildResult, error) {
		return packit.BuildResult{}, fmt.Errorf("always fail")
	}
}
{{< /code/copyable >}}

<!-- We have now written some code that will cause our build process to fail every
time. You can test that by ensuring that you have a `detect` binary built and
then by running the following command to build your `build` binary and run
`pack build`: -->
この実装はビルドプロセスで常に失敗します。
detect コマンドと同じように build コマンドをビルドしましょう。
それから pack コマンドを実行してみましょう。

{{< code/copyable >}}
GOOS=linux go build -ldflags="-s -w" -o ./bin/build ./cmd/build/main.go
pack build <app-name> \
  --path <path/to/app> \
  --buildpack <path/to/project> \
  --builder paketobuildpacks/builder:base \
  --verbose
{{< /code/copyable >}}

{{< code/output >}}
===> DETECTING
[detector] ======== Results ========
[detector] pass: com.example.node-engine@0.0.1
[detector] Resolving plan... (try #1)
[detector] com.example.node-engine 0.0.1
===> ANALYZING
[analyzer] Previous image with name "index.docker.io/library/test-node:latest" not found
===> RESTORING
===> BUILDING
[builder]
[builder] always fail
[builder] ERROR: failed to build: exit status 1
ERROR: failed with status code: 7
{{< /code/output >}}

<!-- Now that we have a build process set up let's talk about how we are going to
add our dependency to our final image. To pass dependency metadata into the
build process we use the `buildpack.toml` and add the following fields: -->
これで、最終的なイメージへ格納する成果物を、ビルドプロセスでどのように作成するのか説明できるようになりました。
ビルドプロセスに依存対象に関するメタデータを渡すには `buildpack.toml` を使用します。
ファイルを作成して次のようなフィールドを記述しましょう。

{{< code/copyable >}}
[metadata]
  [[metadata.dependencies]]
    uri = "https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz"
{{< /code/copyable >}}

<!-- Let's talk about the chunk we just added. -->
記述したのは次のような内容です。

{{< code/copyable >}}
# Buildpack 固有のメタデータを記述します
[metadata]
  [[metadata.dependencies]]
    # 実行している stack と互換性のある依存対象の URI
    uri = "https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz"
{{< /code/copyable >}}

<!-- So let's add some code that parses the uri field out. For toml parsing we
generally use [BurntSushi's TOML Library](https://www.godoc.org/github.com/BurntSushi/toml), so with that let's
write up some quick parsing code. -->
uri フィールドを参照する実装を追加しましょう。
ここでは [BurntSushi's TOML Library](https://www.godoc.org/github.com/BurntSushi/toml) で TOML をパースしています。

`node/build.go`

{{< code/copyable >}}
package node

import (
	"fmt"
	"os"
	"path/filepath"

	"github.com/BurntSushi/toml"
	"github.com/paketo-buildpacks/packit"
)

func Build() packit.BuildFunc {
	return func(context packit.BuildContext) (packit.BuildResult, error) {
		file, err := os.Open(filepath.Join(context.CNBPath, "buildpack.toml"))
		if err != nil {
			return packit.BuildResult{}, err
		}

		var m struct {
			Metadata struct {
				Dependencies []struct {
					URI string `toml:"uri"`
				} `toml:"dependencies"`
			} `toml:"metadata"`
		}
		_, err = toml.DecodeReader(file, &m)
		if err != nil {
			return packit.BuildResult{}, err
		}

		uri := m.Metadata.Dependencies[0].URI
		fmt.Printf("URI -> %s", uri)

		return packit.BuildResult{}, fmt.Errorf("always fail")
	}
}
{{< /code/copyable >}}

<!-- Compiling our `build` executable and running `pack build` should produce the following: -->
build コマンドをビルドして、`pack build` コマンドを実行してみましょう。

{{< code/copyable >}}
GOOS=linux go build -ldflags="-s -w" -o ./bin/build ./cmd/build/main.go
pack build <app-name> \
  --path <path/to/app> \
  --buildpack <path/to/project> \
  --builder paketobuildpacks/builder:base \
  --verbose
{{< /code/copyable >}}

{{< code/output >}}
===> DETECTING
[detector] ======== Results ========
[detector] pass: com.example.node-engine@0.0.1
[detector] Resolving plan... (try #1)
[detector] com.example.node-engine 0.0.1
===> ANALYZING
[analyzer] Previous image with name "index.docker.io/library/test-node:latest" not found
===> RESTORING
===> BUILDING
[builder] URI -> https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz
[builder] ERROR: failed to build: exit status 1
[builder] always fail
ERROR: failed with status code: 7
{{< /code/output >}}

<!-- Now that we have the URI of the dependency that we need to download, let's set
up the layer structure for the dependency to be installed on to. We do that
with the following piece: -->
ダウンロードすべき依存対象の URI が取得できるようになりました。
次は、依存対象をインストールするためのレイヤー構造を準備しましょう。

{{< code/copyable >}}
package node

import (
	"fmt"
	"os"
	"path/filepath"

	"github.com/BurntSushi/toml"
	"github.com/paketo-buildpacks/packit"
)

func Build() packit.BuildFunc {
	return func(context packit.BuildContext) (packit.BuildResult, error) {
		file, err := os.Open(filepath.Join(context.CNBPath, "buildpack.toml"))
		if err != nil {
			return packit.BuildResult{}, err
		}

		var m struct {
			Metadata struct {
				Dependencies []struct {
					URI string `toml:"uri"`
				} `toml:"dependencies"`
			} `toml:"metadata"`
		}
		_, err = toml.DecodeReader(file, &m)
		if err != nil {
			return packit.BuildResult{}, err
		}

		uri := m.Metadata.Dependencies[0].URI
		fmt.Printf("URI -> %s", uri)

		nodeLayer, err := context.Layers.Get("node")
		if err != nil {
			return packit.BuildResult{}, err
		}

		nodeLayer, err = nodeLayer.Reset()
		if err != nil {
			return packit.BuildResult{}, err
		}

		nodeLayer.Launch = true

		return packit.BuildResult{}, fmt.Errorf("always fail")
	}
}
{{< /code/copyable >}}

<!-- Let's talk about the `context.Layers.Get()` function and the
`nodeLayer.Reset()` function individually. The
[`packit.Layers.Get()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layers.Get)
function takes in the name of a layer and returns a
[`packit.Layer`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layer) object
that has been populated with any metadata that exists from previous builds of
your app. This metadata can be useful in determining whether or not a layer can
be reused from cache but we don't need to worry about that for now. The
[`packit.Layer.Reset()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layer.Reset)
function clears all of the metadata and existing files restored from cache for
the given layer, giving a clean slate to work with. Finally,
[`nodeLayer.Launch = true`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layer)
lets the lifecycle know that this layer needs to be available during the launch phase. Now that
we have our layers set up, let's download our dependency and untar it onto the layer. -->
[`packit.Layers.Get()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layers.Get) 関数にはレイヤー名を渡します。
返り値の [`packit.Layer`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layer) には前回ビルドしたときの情報が設定されています。
このメタデータを参照すると、レイヤーを再利用できるか判断できるのですが、ここでは無視しています。
[`packit.Layer.Reset()`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layer.Reset) 関数は、取得したレイヤーの関するキャッシュや過去の情報を初期化します。
[`nodeLayer.Launch = true`](https://pkg.go.dev/github.com/paketo-buildpacks/packit#Layer) とするのは、そのレイヤーが起動フェーズで利用可能であることを教えるためです。
レイヤーが準備できたので、依存対象をダウンロードして展開してみましょう。

{{< code/copyable >}}
package node

import (
	"fmt"
	"io/ioutil"
	"os"
	"os/exec"
	"path/filepath"

	"github.com/BurntSushi/toml"
	"github.com/paketo-buildpacks/packit"
)

func Build() packit.BuildFunc {
	return func(context packit.BuildContext) (packit.BuildResult, error) {
		file, err := os.Open(filepath.Join(context.CNBPath, "buildpack.toml"))
		if err != nil {
			return packit.BuildResult{}, err
		}

		var m struct {
			Metadata struct {
				Dependencies []struct {
					URI string `toml:"uri"`
				} `toml:"dependencies"`
			} `toml:"metadata"`
		}
		_, err = toml.DecodeReader(file, &m)
		if err != nil {
			return packit.BuildResult{}, err
		}

		uri := m.Metadata.Dependencies[0].URI
		fmt.Printf("URI -> %s\n", uri)

		nodeLayer, err := context.Layers.Get("node")
		if err != nil {
			return packit.BuildResult{}, err
		}

		nodeLayer, err = nodeLayer.Reset()
		if err != nil {
			return packit.BuildResult{}, err
		}

		nodeLayer.Launch = true

		downloadDir, err := ioutil.TempDir("", "downloadDir")
		if err != nil {
			return packit.BuildResult{}, err
		}
		defer os.RemoveAll(downloadDir)

		fmt.Println("Downloading dependency...")
		err = exec.Command("curl",
			uri,
			"-o", filepath.Join(downloadDir, "node.tar.xz"),
		).Run()
		if err != nil {
			return packit.BuildResult{}, err
		}

		fmt.Println("Untaring dependency...")
		err = exec.Command("tar",
			"-xf",
			filepath.Join(downloadDir, "node.tar.xz"),
			"--strip-components=1",
			"-C", nodeLayer.Path,
		).Run()
		if err != nil {
			return packit.BuildResult{}, err
		}

		return packit.BuildResult{
			Plan: context.Plan,
			Layers: []packit.Layer{
				nodeLayer,
			},
		}, nil
	}
}
{{< /code/copyable >}}

<!-- If you rebuild the `build` binary and run the `pack build` command once more
you will finally have your first successful build, HOORAY! -->
build コマンドを再びビルドして、`pack build` コマンドを実行すると、今度は成功するはずです。

{{< code/copyable >}}
GOOS=linux go build -ldflags="-s -w" -o ./bin/build ./cmd/build/main.go
pack build <app-name> \
  --path <path/to/app> \
  --buildpack <path/to/project> \
  --builder paketobuildpacks/builder:base \
  --verbose
{{< /code/copyable >}}

<!-- The additional lines that were added are fairly straightforward, we create a
temporary directory to put out downloaded tarball into in order to make sure
that we don't pollute our image. We then download the dependency tarball with a
`curl` command and untar the resulting tarball directly in the `nodeLayer` path
using `tar` on the command line. It is worth noting that this is not the most
efficient way of handling this in Go, but for demonstration purposes it is the
most straightforward. You may notice that we use the `--strip-components` flag
on our tar commands because this particular tarball comes with a top level
directory before you reach the `lib`, `bin`, and `include` directories. We want
these directories to be at the top level of our layer path because these files
are interpreted directly by the lifecycle, which you can read more about
[here](https://github.com/buildpacks/spec/blob/master/buildpack.md#environment).
With this you have a perfectly functional buildpack, which you can test by
running a `pack build` like we did earlier and then running the following
`docker` command if you are using the simple app indicated earlier: -->
追加したコードは単純で特に説明する必要はないと思います。
一時ディレクトリを作成して、`curl` コマンドで取得した tar.gz ファイルを保存しています。
そして、`nodeLayer` の対応するパスへ `tar` コマンドで tar.gz ファイルを展開しています。
Go 言語の実装としては非効率ですが、とても分かりやすいと思います。
tar コマンドの引数に `--strip-components` を付けているのは、ライフサイクルの参照する `lib` や `bin` や `include` ディレクトリの前に存在するディレクトリを無視するためです（ライフサイクルについて詳しくは [CNB の仕様](https://github.com/buildpacks/spec/blob/master/buildpack.md#environment) を参照してください）。

これで、完全に機能する Buildpack が作成できました。
`pack build` コマンドで作成したコンテナイメージを `docker` コマンドで実行してみましょう。

{{< code/copyable >}}
docker run -d -p 8080:8080 -e PORT=8080 <app-name> "node server.js"
{{< /code/copyable >}}

<!-- You can then verify that this is running by either running: -->
実行したコンテナイメージに `curl` コマンドでアクセスして確かめてみましょう。

{{< code/copyable >}}
curl http://localhost:8080
{{< /code/copyable >}}

{{< code/output >}}
hello world
{{< /code/output >}}

<!-- or visiting [`http://localhost:8080`](http://localhost:8080) in your web
browser. Either way you should expect to see "hello world" as your output. -->
Webブラウザで [`http://localhost:8080`](http://localhost:8080) にアクセスしても確認できるはずです。
どちらにしても "hellow world" と表示されれば成功です。