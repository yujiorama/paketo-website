---
title: "PHP Buildpack リファレンス"
menu:
  main:
    parent: reference
    identifier: php-reference
    name: "PHP Buildpack"
---

{{% reference_exec_summary bp_name="Paketo PHP Buildpack" bp_repo="https://github.com/paketo-buildpacks/php" howto_docs_path="/docs/howto/php" %}}

## 対応している依存対象

<!-- The PHP Paketo Buildpack supports several versions of PHP.
For more details on the specific versions supported in a given buildpack
version, see the [release notes](https://github.com/paketo-buildpacks/php/releases). -->
PHP Paketo Buildpack は PHP ランタイムの複数のバージョンに対応しています。
具体的なバージョンは [Buildpack のリリースノート](https://github.com/paketo-buildpacks/php/releases) で確認してください。

## Buildpack の設定する環境変数

<!-- The PHP CNB sets a number of environment variables during the `build` and
`launch` phases of the app lifecycle. The sections below describe each
environment variable and its impact on your app. -->
PHP Buildpack はアプリケーションライフサイクルのビルドフェーズや起動フェーズでいくつかの環境変数を設定します。
このセクションではアプリケーションに影響するであろう環境変数について説明します。

### APP_ROOT

* Set by: `httpd` buildpack
* Phases: `launch`
* Value: path of app source

### SERVER_ROOT

* Set by: `httpd` buildpack
* Phases: `launch`
* Value: path of the httpd installation

### MIBDIRS

* Set by: `php-dist` buildpack
* Phases: `build` and `launch`
* Value: See [php documentation](https://www.php.net/manual/en/snmp.installation.php)

### PATH

* Set by: `php-dist` buildpack
* Phases: `build` and `launch`
* Value: path to the php executable

### PHP_API

* Set by: `php-dist` buildpack
* Phases: `build` and `launch`
* Value: internl api version (YYYYMMDD)

### PHP_EXTENSION_DIR

* Set by: `php-dist` buildpack
* Phases: `build` and `launch`
* Value: location of directory with dynamic libraries for extensions

### PHP_HOME

* Set by: `php-dist` buildpack
* Phases: `build` and `launch`
* Value: location of php installation

### PHP_INI_SCAN_DIR

* Set by: `php-web` buildpack
* Phases: `build` and `launch`
* Value: `<APP-ROOT>/.php.ini.d`