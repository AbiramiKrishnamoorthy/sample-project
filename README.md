<!-- cSpell:ignore sysname Adop PSWAGGER appprops -->

# Training DX WebApi Server

## Project への fork

Git のログは成果物として納品対象になる可能性があります。

shared の Git 履歴については対象外としたいためプロジェクト取り込み時は初期化して削除していください。

- コミットメッセージにforkを行ったバージョンを記載しておくとバージョンアップ時の参考になります

```bash
rm -rf .git
git init
git add .
git commit -m 'Acts v1.0.0からfork'
```

`gradle.properties` を編集します。

| プロパティ名         | 説明                            |
| -------------------- | ------------------------------- |
| sysname              | リポジトリ名                    |
| sysGroup             | プロジェクトパッケージ名        |
| sysVersion           | 次期リリースバージョン          |
| acn.project.packages | プロジェクトパッケージ名        |
| btsType              | プロジェクトで使用しているBTS名 |

`nexus` から始まっているプロパティ名は以下に変更します。

- Adopの設定値に依存するので問題がある場合はDevOpsCOEに問い合わせて下さい

```properties
nexusHost=https://[プロジェクトADOPのホスト名].acnshared.com
nexusRepositoryPath=nexus/repository
# ライブラリダウンロード用(shared/acts/project - release/snapshotが混ざっている)
nexusMvnPublic=maven-public
# ライブラリダウンロード用(shared/acts/project)
nexusMvnPublicReleases=maven-proxy-shared
nexusMvnPublicSnapshots=acts-snapshots
# ライブラリアップロード用(project)
nexusMvnReleases=maven-releases
nexusMvnSnapshots=maven-snapshots
nexusRawId=specification
nexusArtifactsRawId=archive-artifacts
```

`src/main/java/com/skeleton` を必要に応じてプロジェクトのパッケージ名に変更します。

`settings.gradle` を `gradle.properties` の `sysname` と同じにします。

`scanBasePackages` を検索してプロジェクトパッケージ名を追加します。

`src/main/resources/appprops/application.env.yml` を開きます。

- `skeleton` で検索してプロジェクトのパッケージ名に変更します。
- DB設定を変更します。

## インハウスリポジトリへのアクセス設定

共通ライブラリをインハウスリポジトリ(Nexus)から取得する必要があります。

インハウスリポジトリには ID/Password の認証が必要になります。

ユーザホームの `.gradle` フォルダに `gradle.properties` を作成して以下の内容を設定します。

```text
nexusUsername=[自分のID]
nexusPassword=[パスワード]
```

認証情報が不正な場合以下のようなエラーが発生します。

```bash
Could not resolve all files for configuration ':compileClasspath'.
> Could not resolve com.accenture.acts:acts-core-server-lib:1.0.4.  Required by:
      project :
   > Could not resolve com.accenture.acts:acts-core-server-lib:1.0.4.
      > Could not get resource 'https://adp001012.acnshared.com/nexus/content/repositories/releases/com/accenture/acts/acts-core-server-lib/1.0.4/acts-core-server-lib-1.0.4.pom'.
         > Could not GET 'https://adp001012.acnshared.com/nexus/content/repositories/releases/com/accenture/acts/acts-core-server-lib/1.0.4/acts-core-server-lib-1.0.4.pom'. Received status code 403 from server: Forbidden
```

- `401` エラーの場合は ID/パスワードが間違っています。
- `403` エラーの場合は認証は OK なのですがインハウスリポジトリへのアクセス権限が無いのでプロジェクト管理者に問い合わせて下さい。

## Gradle コマンド

[asciidoctor](https://asciidoctor.org/)や[graphviz](https://graphviz.org/)がインストールされていてドキュメント生成が可能な環境なら以下のコマンドでドキュメントが生成されます。

`build/docs/html5/` 配下に HTML が生成されます。

```bash
gradlew clean javadoc generateSwaggerUI generateReDoc redpen asciidoc
```

依存ライブラリのアップデート状況や脆弱性を出力する場合は以下を実行します。

```bash
gradlew clean dependencyUpdates dependencyCheckAnalyze
```

| コマンド               | 説明                                                                                                                           | 備考                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| javadoc                | Javadoc を `build/docs/javadoc` に生成する                                                                                     |                                                                                        |
| generateSwaggerUI      | `build/swagger-yaml/swagger.yaml` から [Swagger UI](https://swagger.io/swagger-ui/) を`build/docs/html5/openapi/ui` に生成する |                                                                                        |
| generateReDoc          | `build/swagger-yaml/swagger.yaml` から [ReDoc](https://rebilly.github.io/ReDoc/) を`build/docs/html5/openapi/re` に生成する    |                                                                                        |
| redpen                 | `src/docs/asciidoc` に[RedPen](http://redpen.cc/)による文書規約の自動精査を実行する                                            |                                                                                        |
| asciidoc               | asciidoc ファイルから HTML と PDF を `build/docs` に生成する                                                                   |                                                                                        |
| dependencyUpdates      | `build/asciidoc/dependencyUpdates/index.adoc` にアップデート可能のライブラリ一覧を生成する                                     |                                                                                        |
| dependencyCheckAnalyze | `build/docs/dependency-check/dependency-check-report.html` に依存ライブラリの脆弱性情報を出力します。                          | 初回はローカルにデータベースをキャッシュするので10分ほど(PCスペックに依存)かかります。 |
| messageYaml2Csv        | `src/main/resources/messages` 配下のメッセージ Yaml を CSV に変換して `build/messages/csv` 配下に出力します。                  | `src/main/resources/messages` に配置することでメッセージリソースとして使用出来ます     |
| messageCsv2Yaml        | `src/main/resources/messages` 配下のメッセージ CSV を Yaml に変換して `build/messages/yaml` 配下に出力します。                 | `src/main/resources/messages` に配置することでメッセージリソースとして使用出来ます     |
| genApiModel            | `build/openapi/gen/openapi-ja.yaml` からコードを`build/openapi-gen/` に生成する                                                | 生成された model は Entity として、controller はインターフェースとして利用する         |

## メッセージリソースについて

### CSV/Yaml について

`src/main/resources/messages.properties` の出力内容は `src/main/resources/messages` にあるリソースが元となります。

- CSV/Yaml 形式に対応していて両方のフォーマットが配置されていても問題ありません。

### const class について

`src/main/resources/messages` に定義したメッセージのキーと値を const class として自動生成します。

出力先は `build.gradle` の `messageResources2Properties.genConstClass` を変更することで対応可能です。

- 不要の場合は `genConstClass` を削除することで出力されなくなります。

```groovy
messageResources2Properties {
  genConstClass 'com.skeleton.messages.MessageConst'
}
```

- 下記コマンドで、OpenAPIからAPI HTML `build/docs/adoc/html5/ja/openapi/` に生成。また、API PDFを `build/docs/adoc/pdf/ja/openapi/` に生成可能です。

```bash
gradlew genApiAsciidoc genApiHtml2 asciidoc
```

- また、API仕様書や画面仕様書が `build/docs/adoc/html5/ja/` および `build/docs/adoc/pdf/ja/` に生成されます。

- 下記コマンドで、仕様書のHTMLをConfluenceにUpload可能です。

```bash
04_01_publishToConfluence.bat
```

## Open API

一般的なOpenAPI仕様には存在しないActs固有の機能について説明します。

抜粋となるので詳細な説明や最新の情報が必要な場合は[Acts Gradle Plugin](https://adp001011.acnshared.com/gitlab/jp_ata_share/gradle-plugin/multi-file-openapi)のReadmeを参照して下さい。

<!-- @suppress SuccessiveSentence -->
### 多言語化

同じレベルに `x-*-i18n` と置き換えするプロパティとその子ノードとして言語別の文字列を設定します。

```yaml
post:
  summary: 友人紹介API
  x-summary-i18n:
    en: Introducing friends API
  description: |
    友人紹介メッセージ情報を取得する。
  x-description-i18n:
    en: |
      Get friend information message information.
```

生成コマンドは以下になります。

```bash
gradlew clean
gradlew generateSwaggerUI generateReDoc
gradlew generateSwaggerUI generateReDoc -PSWAGGER_LANG=en
gradlew asciidoc
```

- `build\openapi\gen` にそれぞれの言語に対応した openapi.yaml が出力されます。
  - openapi-ja.yaml
  - openapi-en.yaml
- `-PSWAGGER_LANG` オプションを省略すると `x-*-i18n` を無視してデフォルトの値を使用しますが出力ファイル/ディレクトリは `ja` として扱われます。
- asciidoc で生成した HTML などは build/docs/html5/index.html に生成されています。

サンプルから生成された swagger.yaml は以下の様になります。

openapi-ja.yaml

```yaml
/account/invitation/create:
  post:
    summary: 友人紹介API
    description: |
      友人紹介メッセージ情報を取得する。
```

openapi-en.yaml

```yaml
/account/invitation/create:
  post:
    summary: Introducing friends API
    description: |
      Get friend information message information.
```

### changelog出力

Asciidocフォーマットでchangelog出力を行います。

`src/docs/openapi/yaml/paths` のAPI定義ファイルにある `summary` と同レベルに `x-changelog` がある場合にchangelogを出力します。

- `get`,`put`,`post`,`delete`,`options`,`head`,`patch`  に対応しています。
  - 使用される項目は `summary`, `operationId`, `x-changelog` になります。
  - `x-changelog`  の項目は結合時に削除されますので結合後のyamlファイルには出力されません。
  - `x-changelog.version` が `snapshot` の場合はリリース作業時にリリースバージョンに置換されます。

- `level` は以下の対応になります。

| 項目       | ja         | en             |
| ---------- | ---------- | -------------- |
| new        | 新規       | New            |
| severe     | 重度な変更 | Severe changes |
| minor      | 軽度な変更 | Minor changes  |
| deprecated | 廃止       | Deprecated     |

```yaml
post:
  summary: 友人紹介API
  description: 「メールアドレス+パスワード」によるログイン認証を実行する。
  operationId: authLogin
  x-changelog:
    - version: snapshot
      level: minor
      description:
        - 非必須のパラメータを修正
        - 必須項目の `name` を非必須に変更
    - version: v1.0.3
      level: severe
      description:
        - エンドポイントを修正
    - version: v1.0.2
      level: new
      description:
        - APIを実装し公開
    - version: v1.0.1
      level: new
      description:
        - API定義書のみ公開
```

APIは廃止したが仕様書は残したい場合など詳細な手順については Acts Gradle Pluginの [multi-file-openapi](https://adp001011.acnshared.com/gitlab/jp_ata_share/gradle-plugin/multi-file-openapi) を参照して下さい。

## OpenAPI の Readme

- [openapi/README.md](./src/docs/openapi/README.md)

## Docker Image

### イメージ作成

```shell
gradlew bootjar
make image
```

作成したイメージを削除するには以下を実行する

```shell
make rmi
```

### イメージの push

イメージの push にはプライベートリポジトリが必要です。

- Makefile のログイン部分を aws/azure などに合わせて修正して下さい
- サンプルコードは AWS になります

```shell
make push
```

latest タグを push する場合は以下になります。

```shell
make latest
```

### 作成したイメージに bash でログイン

<!-- cSpell:disable -->
```shell
make run-bash

bash-4.2$ ls -lah
total 34M
drwxr-xr-x 1 nobody nobody 4.0K Oct  7 16:20 .
drwxr-xr-x 1 root   root   4.0K Oct  7 17:07 ..
-rw-r--r-- 1 nobody nobody  34M Oct  7 16:18 service.jar
```

## Acts トレーニングDX 各プロジェクトの開発ガイドなど

ディレクトリ構成はトレーニングで実装が必要なもののみ抜粋

### Vue (acts-webapp-vue)

- 参考資料
  - リポジトリ内の開発ガイド(Asciidoc) docs/index.adoc
  - Vue.jsアーキテクチャ概要 acts-front-vuejs-architecture_ja.pptx
- ディレクトリ構成
  - src/business-acts 配下
    - pages 各画面
    - repositories APIとの通信
    - stores Piniaを用いたストア

### Angular (acts-sample-front)

- 参考資料
  - リポジトリの README.md にリンクのある acts-devguide - front (PDF)
- ディレクトリ構成
  - src/business 配下
    - pages 各画面
    - queries ストアからデータを取得
    - services APIとの通信
    - states データストア

### Java (acts-sample-webapi)

- 参考資料
  - リポジトリ内にあるドキュメント
    - README.md (root)
    - OpenApiのReadme src/docs/openapi/README.md
    - トレーニングDXの開発ガイド src/docs/asciidoc/index.adoc
  - Acts(Java)開発ガイド (PDF)
- ディレクトリ構成
  - src/main/java/com/accenture/tdx 配下
    - controller APIのエンドポイント（HTTPリクエストを受け取りJsonを返す）
    - repository DBアクセス
    - service, service/impl ビジネスロジック（Controller と Repository を橋渡しする）
  - build/openapi-gen/src/main/java/com/accenture/tdx 配下 (OpenAPI定義ファイルから自動生成されたコード)
    - controller (interface)
    - model
<!-- cSpell:enable -->
