---
title: "OpenAPI Generatorを用いて自動生成したREST API clientのSDKのnpmパッケージ紹介"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blockchain", "symbol", "openapigenerator", "typescript", "npm"]
published: true
---

# SymbolブロックチェーンのTypeScript向けREST API clientコードの自動生成とDual package(CommonJS/ES Modules)＆CDN対応済npmパッケージ公開の紹介

## 要約

この記事では、[OpenAPI Generator](https://openapi-generator.tech/)を用いてTypeScript向けにSymbolブロックチェーンのREST API clientを自動生成し、CommonJS/ES Modules両対応のDual packageとしてnpmパッケージを公開すると同時にwebpackでCDN用のビルドファイルも同梱してCDNからもSDKを使用できるようにした方法を解説します。

また、作成したSDKの使い方についてもサンプルコードを交えて紹介します。

公式SDKへ依存していたこれまでから一歩踏み出して、コミュニティドリブンなOSSとして今後もより良いSDKにしていきたいと考えています。

どんな形の貢献でも大歓迎で、どうぞお気軽にDiscussionへの参加やIssue作成やプルリクエスト作成等、ご参加くださいますと幸いです。Symbolブロックチェーンにさほど関心ない方もお手頃なOSSへの貢献実績として(小粒な内容ですが`good first issue`的なIssueも色々と作成してみたので)ぜひお気軽にご参加ください。

詳細については、SDKのレポジトリやnpmパッケージの以下リンクをご参照ください。

- Symbol SDK OpenAPI Generator typescript-axios版
  - GitHub
    - [https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-axios](https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-axios)
  - npm
    - [https://www.npmjs.com/package/@nemtus/symbol-sdk-openapi-generator-typescript-axios](https://www.npmjs.com/package/@nemtus/symbol-sdk-openapi-generator-typescript-axios)
- Symbol SDK OpenAPI Generator typescript-fetch版
  - GitHub
    - [https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-fetch](https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-fetch)
  - npm
    - [https://www.npmjs.com/package/@nemtus/symbol-sdk-openapi-generator-typescript-fetch](https://www.npmjs.com/package/@nemtus/symbol-sdk-openapi-generator-typescript-fetch)

## 背景

Symbolブロックチェーンの特徴の一つとして、REST APIを用いたブロックチェーン上の情報へのアクセスのしやすさがあります。

この部分をJavaScript/TypeScript向けのSDK(GitHub: [https://github.com/symbol/symbol-sdk-typescript-javascript](https://github.com/symbol/symbol-sdk-typescript-javascript), npm: [https://www.npmjs.com/package/symbol-sdk/v/2.0.1](https://www.npmjs.com/package/symbol-sdk/v/2.0.1))が力強くサポートしてくれていたのですが、コア開発チームの方針変更により、このSDK(=以降旧公式SDKと呼ぶ)は非推奨となり、新たにJavaScript向けのシンプルなSDK(GitHub: [https://github.com/symbol/symbol/tree/dev/sdk/javascript](https://github.com/symbol/symbol/tree/dev/sdk/javascript), npm: [https://www.npmjs.com/package/symbol-sdk/v/3.0.0](https://www.npmjs.com/package/symbol-sdk/v/3.0.0)=以降新公式SDKと呼ぶ)の開発とメンテナンスがコア開発チームによって進められることになりました。

結果的に、新公式SDKの今後の開発ターゲットにはTypeScript向け実装や、REST API clientの実装等が入らないこととなり、REST APIから得られたレスポンスをTypeScriptで型付きで扱うには、非推奨扱いの旧公式SDKを一定のリスクを許容して使い続けるか、各自で何とかするかの選択を迫られることになりました。

オフィシャルに非推奨扱いの旧公式SDKを使い続けることにはやはりリスクがあります。そのため、各自で何とかする方法の一つとして、OpenAPI形式のSchemaの情報を元に、OpenAPI Generatorを用いてTypeScript向けにREST API clientを自動生成し、npmパッケージとして公開することで、旧公式SDKのTypeScript向けREST API client相当の機能を代替することを第一目標として、[NEMTUS](https://nemtus.com/)にて取組を始めました。

## アーキテクチャ

全体的な構成は以下の通りです。

1. [https://github.com/symbol/symbol-openapi](https://github.com/symbol/symbol-openapi)にてOpenAPI形式のSchemaがメンテナンスされているようなので、その情報を元にOpenAPIのymlファイルを生成し、
2. OpenAPI Generatorと1で生成したymlファイルを用いてTypeScript向けREST API clientコードを自動生成し、
3. Dual Package(CommonJS/ES Modules両対応)に対応したnpmパッケージとして公開するとともに、
4. webpackでCDN用にビルドしたものも併せて公開することでブラウザからCDNを直接読み込んで利用することもできるようにする

TypeScript向けのGeneratorは複数種類があり、どれを使うか迷ったのですが、過去に使用したことがあるという点から[typescript-axios](https://openapi-generator.tech/docs/generators/typescript-axios)版と、業界の今後の動向としてブラウザネイティブなWeb標準へ寄せていく技術的動向を感じている点から[typescript-fetch](https://openapi-generator.tech/docs/generators/typescript-fetch)版の2種類を選んで作成してみることにしました。

## SDK開発におけるポイント

### ディレクトリ構成

詳細な構成は以下をご参照ください。他にもESLint, Prettier, GitHub Actions, GitHubの各種設定ファイルやディレクトリ等がありますが、OpenAPIのSchemaのymlファイルのビルドやDual Package化やCDN対応に関連する部分を中心に説明を書いています。さらなる詳細はGitHubのレポジトリを直接ご参照ください。

```shell
.
├── bundle.js ... LICENSE, README.md, package.json, package-lock.json等をnpm publish対象ディレクトリにコピーするスクリプト
├── dist ... このディレクトリ内がnpm publishされる
│   ├── LICENSE
│   ├── README.md
│   ├── cjs ... CommonJS向けビルド生成物
│   ├── esm ... ES Modules向けビルド生成物
│   ├── index.min.js ... CDN向けビルド生成物
│   ├── package-lock.json
│   └── package.json
├── examples ... サンプルコード
│   ├── browser-cdn ... CDN向けサンプルコード
│   ├── nodejs-javascript ... Node.js x JavaScript環境向けサンプルコード
│   └── nodejs-typescript ... Node.js x TypeScript環境向けサンプルコード
├── openapi-generator-config.yml ... OpenAPI Generatorの設定ファイル(メソッドのリクエストパラメーターが1オブジェクトにまとめられ複数個にならないよう設定)
├── openapitools.json ... OpenAPI Generatorのバージョン管理ファイル(v6.0.0で設定)
├── package-lock.json
├── package.json ... Dual Package向け設定やCommonJS/ES Modules, CDN向けビルドのスクリプト等が入っている。distディレクトリの中身だけnpm publishする構成を意図しており、package.jsonの中のmainとmoduleの項目は、package.jsonそれ自体がdistディレクトリにコピーされた後にnpm publishされる前提でのパスが指定されていることに注意。
├── post-build-cdn.js ... CDN向けビルド後に実行されるスクリプト
├── post-build-cjs.js ... CommonJS向けビルド後に実行されるスクリプト
├── post-build-esm.js ... ES Modules向けビルド後に実行されるスクリプト
├── pre-build-cdn.js ... CDN向けビルド前に実行されるスクリプト
├── pre-build-cjs.js ... CommonJS向けビルド前に実行されるスクリプト
├── pre-build-esm.js ... ES Modules向けビルド前に実行されるスクリプト
├── src
│   ├── api ... OpenAPI Generatorで自動生成されるディレクトリ。このディレクトリ配下は原則として手動で編集しない。
│   ├── cdn.ts ... CDN向けにwebpackでまとめるためのエントリーポイント
│   └── index.ts ... apiディレクトリからexportされたものをそのままexport
├── symbol-openapi ... gitのsubmodule機能で https://github.com/symbol/symbol-openapi レポジトリの内容を組み込み、npm run buildして生成されるsymbol-openapi/_build/openapi3.ymlファイルを使用する。
│   ├── _build ... このディレクトリ内にopenapi3.ymlファイルがOpenAPIのSchemaファイルのビルド結果ファイルとして生成される。
│   ├── package-lock.json
│   └── package.json
├── tsconfig.json ... tscやwebpackでビルドする際の挙動に影響する設定ファイル
└── webpack.config.js ... CDN用のwebpackビルド設定
```

### Node.jsだけでなくJavaを実行できる環境が必要

OpenAPI GeneratorのCLI実行にはJavaが必要です。Dockerを使う方法もあるかもしれませんが、今回はシンプルにJavaをインストールして開発を進めました。ここは各自の環境に合わせてご準備ください。

### OpenAPI形式のschemaのymlファイルを生成

[https://github.com/symbol/symbol-openapi](https://github.com/symbol/symbol-openapi)レポジトリをgit submoduleとしてSDKのレポジトリに組み込んであります。その中でOpenAPIのymlファイルをビルドするコマンドをまず実行し、OpenAPI形式のschemaのymlファイルを生成します。

```shell
# OpenAPIのSchemaを管理しているレポジトリのsubmoduleのディレクトリへ移動
cd symbol-openapi

# Schemaのymlファイルのビルドに必要なパッケージをインストールする
npm install

# Schemaのymlファイルをビルドする
npm run build
# このコマンド実行後にsymbol-openapi/_build/openapi3.ymlが生成されるので以降はそのファイルを使う

```

### OpenAPI GeneratorでTypeScript向けREST API clientコードを自動生成

OpenAPI形式のschemaのymlファイルが生成されたら、そのファイルを使ってOpenAPI GeneratorでTypeScript向けREST API clientコードを自動生成します。

- `-i`オプション
  - 入力となるOpenAPI形式のschemaのymlファイルパスを指定
  - 今回は`./symbol-openapi/_build/openapi3.yml`を指定
- `-g`オプション
  - どのGeneratorを使うかを指定。
  - 今回は`typescript-axios`と`typescript-fetch`の2種類を使用
  - [https://openapi-generator.tech/docs/generators#client-generators](https://openapi-generator.tech/docs/generators#client-generators)に利用可能なGeneratorの一覧あり
- `-o`オプション
  - 出力先ディレクトリの指定
  - 今回は`src/api`を指定
- `-c`オプション
  - その他にも様々な設定を指定可能
  - 今回は設定ファイルのパスを指定する書き方で、
  - かつ、リクエストパラメーターが複数にならないよう一つのオブジェクトにまとめるよう設定
    - `useSingleRequestParameter: true`

```shell
# レポジトリルートに戻る
cd ..

# 必要なパッケージをインストールする(OpenAPI Generatorもインストールされる)
npm install

# OpenAPI Generatorのバージョンを明示的に指定する
npm run openapi:set:version
# 実際には以下コマンドを実行している
# npx @openapitools/openapi-generator-cli version-manager set 6.0.0

# OpenAPI GeneratorでTypeScript向けREST API clientコードを自動生成する
npm run openapi:generate
# 実際には以下コマンドを実行している
# typescript-axiosの場合の例
# npx @openapitools/openapi-generator-cli generate -i ./symbol-openapi/_build/openapi3.yml -g typescript-axios -o ./src/api -c ./openapi-generator-config.yml
# typescript-fetchの場合、 `-g typescript-fetch` -> `-g typescript-fetch`となる
# 他の場合も基本的には`-g`オプションの指定を変えてやればOKだと思う
```

このように自動生成された`./src/api`ディレクトリのコードをgit管理対象とするか否かは少し迷ったのですが、どういうコードが結果的に生成されているかを明示的にに把握できていたほうが、開発やデバッグ等に便利かと考え、現時点ではgit管理対象とすることにしています。

### CommonJS向けビルド

自動生成されたコードを各環境向けにビルドしていきます。
まずCommonJS向けです。

```shell
# CommonJS向けビルド
npm run build:cjs
# 実際には以下コマンドを実行している
# node pre-build-cjs && tsc --build --clean && tsc --target es5 --module commonjs && node post-build-cjs
```

ビルドの処理の実体は `tsc --target es5 --module commonjs`の箇所です。

`tsconfig.json`の中で`"outDir": "lib/"`という設定がされているので、`lib/`ディレクトリにビルドされるようになっています。

ビルド後に`node post-build-cjs`が実行されることで、`post-build-cjs.js`ファイル内に書かれた処理が実行され、CommonJS向けに`lib/`ディレクトリにビルドされた生成物を`dist/cjs`にコピーして、コピー元の`lib/`ディレクトリを削除しています。

またビルド前に、`node pre-build-cjs`で、コピー先、コピー元双方を削除してリセットしています。

### ES Modules向けビルド

次はES Modules向けです。

```shell
# ES Modules向けビルド
npm run build:esm
# 実際には以下コマンドを実行している
# node pre-build-esm && tsc --build --clean && tsc --target esnext --module esnext && node post-build-esm
```

ビルドの処理の実体は `tsc --target esnext --module esnext`の箇所です。

`tsconfig.json`の中で`"outDir": "lib/"`という設定がされているので、`lib/`ディレクトリにビルドされるようになっています。

ビルド後に`node post-build-esm`が実行されることで、`post-build-esm.js`ファイル内に書かれた処理が実行され、ES Modules向けに`lib/`ディレクトリにビルドされた生成物を`dist/esm`にコピーして、コピー元の`lib/`ディレクトリを削除しています。

またビルド前に、`node pre-build-esm`で、コピー先、コピー元双方を削除してリセットしています。

### CDN向けビルド

最後にCDN向けビルドです。

```shell
# CDN向けビルド
npm run build:cdn
# 実際には以下コマンドを実行している
# node pre-build-cdn && tsc --build --clean && tsc --target esnext --module esnext && webpack && node post-build-cdn
# tsc --target esnext --module esnext はCDN向けには直接的には不要だが、ES Modules向けビルドが壊れていないことを確認する意味合いとして一応残してある。消してもいいかも。
```

ビルドの処理の実体は `webpack`の箇所です。

`webpack.config.js`の中で`entry: './src/cdn.ts'`, `output: { filename: '../cdn/main.js' }`という設定がされているので、`'./src/cdn.ts'`のファイルがエントリーポイントとなって`../cdn/main.js`にビルドされます。ビルド先の相対パスの指定はレポジトリルート起点ではなく、entryファイルを起点の相対パス指定となっていることに注意が必要です。

エントリーポイントの`'./src/cdn.ts'`ファイル内でグローバル`window`オブジェクトに`symbolSdkOpenAPIGeneratorTypeScriptAxios`をぶら下げているので、ビルドされたファイルをシングルファイルとしてsrcタグ等でブラウザで読み込むと`window.symbolSdkOpenAPIGeneratorTypeScriptAxios`のようにSDKが使用できます。

ビルド後に`node post-build-cdn`が実行されることで、`post-build-cdn.js`ファイル内に書かれた処理が実行され、CDN向けにビルドされた`cdn/main.js`ファイルを`dist/index.min.js`にコピーして、コピー元の`cdn/main.js`ディレクトリを削除しています。

またビルド前に、`node pre-build-cdn`で、コピー先、コピー元双方を削除してリセットしています。

### `npm publish`対象ディレクトリに必要なファイルをコピー

CommonJS, ES Modules, CDN向けそれぞれにビルドされた生成物がdistディレクトリにコピーされましたが、distディレクトリを直接`npm publish`したいため、以下のようなファイルを`dist`ディレクトリにコピーする必要があります。これを`node bundle`でやっています。

- `LICENSE`
- `package-lock.json`
- `package.json`
- `README.md`

ここまで完了すると、distディレクトリの中が以下のようになっており、distディレクトリを`npm publish`することで、CommonJS/ES Modules両対応のDual Packageとしてのnpmパッケージ公開かつブラウザで直接CDN等から読み込んで使えるものも含んだ形で様々な環境で使用できる形での公開準備が整いました。

```shell
├── LICENSE
├── README.md
├── cjs ... CommonJS向けビルド生成物
├── esm ... ES Modules向けビルド生成物
├── index.min.js ... CDN向けビルド生成物
├── package-lock.json
└── package.json

# package.jsonの中の`"main": "./cjs/index.js",`がCommonJS向けエントリーポイントがdistの中の`./cjs/index.js`であることを示しており、
# package.jsonの中の`"module": "./esm/index.js",`がES Modules向けエントリーポイントがdistの中の`./esm/index.js`であることを示している形となり、
# CDN向けにはCDNのURLから`./index.min.js`のファイルを指定すればOK
```

レポジトリ上では上記ビルド手順を`npm run build`でまとめて一括実行しています。

### `npm publish`して公開

最後にnpm publishして公開しています。
GitHub Actionsで`workflow_dispatch`で手動で公開処理をトリガーできるようにしています。

## SDKの使用方法

npmパッケージのインストールは以下コマンドで可能です。

typescript-axios版

```shell
npm install @nemtus/symbol-sdk-openapi-generator-typescript-axios

```

typescript-fetch版

```shell
npm install @nemtus/symbol-sdk-openapi-generator-typescript-fetch

```

SDKの使い方について、サンプルコードを少し紹介しながら説明します。
サンプルコードはレポジトリ内の`examples`ディレクトリ配下にて、そのまま動作可能な形で配置しているので、以下の通り、SDKのレポジトリをクローンして試してみるのが楽だと思います。

### typescript-axios版の使用方法

まずSDKのレポジトリをクローンして`npm i`してください。

```shell
git clone https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-axios.git
cd symbol-sdk-openapi-generator-typescript-axios
npm install

```

#### Node.js x TypeScript環境(typescript-axios版)

TypeScript向け環境でサンプルコードを試すためのディレクトリに移動して`npm i`してください。

```shell
cd examples/nodejs-typescript
npm install

```

まずはrequestParametersを指定する必要がない場合のサンプルコードを試しに実行してみましょう。

```shell
$ npx ts-node api/NodeRoutesApi/getNodeInfo.ts

200
OK
{
  version: 16777987,
  publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
  networkGenerationHashSeed: '57F7DA205008026C776CB6AED843393F04CD458E0AA2D9F1D5F31A402072B2D6',
  roles: 3,
  port: 7900,
  networkIdentifier: 104,
  host: 'symbol-sakura-16.next-web-technology.com',
  friendlyName: 'next-web-technology',
  nodePublicKey: '9545F928A1B2FB4AC944BC1EC2F01FB84A503F6449B6BE3451B3F7A0F06B5BCF'
}

```

実行したサンプルコードは以下の通りです。

```typescript:examples/nodejs-typescript/api/NodeRoutesApi/getNodeInfo.ts
import { Configuration, ConfigurationParameters, NodeInfoDTO, NodeRoutesApi } from '@nemtus/symbol-sdk-openapi-generator-typescript-axios';
import { AxiosResponse } from 'axios';

(async () => {
  const configurationParameters: ConfigurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
  };
  const configuration: Configuration = new Configuration(configurationParameters);
  const nodeRoutesApi: NodeRoutesApi = new NodeRoutesApi(configuration);
  const response: AxiosResponse<NodeInfoDTO, any> = await nodeRoutesApi.getNodeInfo();
  const dto: NodeInfoDTO = response.data;
  console.log(response.status);
  console.log(response.statusText);
  console.dir(dto, { depth: null });
})();

```

ConfigurationParametersという型のオブジェクトのbasePathにノードのURLを指定して、それを使ってConfigurationをインスタンス化し、さらにNodeRoutesApiをインスタンス化すると、`nodeRoutesApi.getNodeInfo()`のようにREST APIを叩く処理を実行できるようになります。

当然レスポンスにも型の情報がついていて、エディタでの自動補完や、コードジャンプ等で細かなドキュメントが無くてもある程度処理をサクサク書いていける環境が整いました。嬉しい！

次に、requestParametersを指定する必要がある場合のサンプルコードを実行してみましょう。

```shell
$ npx ts-node api/AccountRoutesApi/getAccountInfo.ts

200
OK
{
  account: {
    version: 1,
    address: '68A48712C4D6FDCBDDFEEF35EB6E3430638700D1DA98C120',
    addressHeight: '1',
    publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
    publicKeyHeight: '447',
    accountType: 1,
    supplementalPublicKeys: {
      linked: {
        publicKey: '5F87A37D1EAD570F4D0FD4C11A9D5EED5ABE82EF2E992B97CCDAC84F241470E0'
      },
      vrf: {
        publicKey: '806E9448598C922B371DA8CFD7E16E8F5F53594B3AECE13F0708778A4480A752'
      }
    },
    activityBuckets: [
      {
        startHeight: '1465920',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476848133893'
      },
      {
        startHeight: '1465200',
        totalFeesPaid: '0',
        beneficiaryCount: 2,
        rawScore: '476783918668'
      },
      {
        startHeight: '1464480',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476790611323'
      },
      {
        startHeight: '1463760',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476796916067'
      },
      {
        startHeight: '1463040',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476803888986'
      }
    ],
    mosaics: [
      { id: '6BED913FA20223F8', amount: '516727736888' },
      { id: '24F7CF825DBCDD42', amount: '499999886' },
      { id: '310378C18A140D1B', amount: '923' },
      { id: '6AE25FA5E8CA0646', amount: '1000000000' }
    ],
    importance: '476783918668',
    importanceHeight: '1465920'
  },
  id: '60517BE5CCA17918A561056D'
}

```

実行したサンプルコードは以下の通りです。requestParametersという1オブジェクトに全ての変数をセットしてREST APIを叩くメソッドに引数として渡しているのがわかると思います。そのオブジェクトの型も提供されていて、どんなパラメーターを指定してREST APIを叩く必要があるかを覚えていなくても型の情報からサクサク推測して書けるようになりました。これも嬉しい！

```typescript:examples/nodejs-typescript/api/AccountRoutesApi/getAccountInfo.ts
import {
  AccountInfoDTO,
  AccountRoutesApi,
  AccountRoutesApiGetAccountInfoRequest,
  Configuration,
  ConfigurationParameters,
} from '@nemtus/symbol-sdk-openapi-generator-typescript-axios';
import { AxiosResponse } from 'axios';

(async () => {
  const configurationParameters: ConfigurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
  };
  const configuration: Configuration = new Configuration(configurationParameters);
  const accountRoutesApi: AccountRoutesApi = new AccountRoutesApi(configuration);
  const requestParameters: AccountRoutesApiGetAccountInfoRequest = {
    accountId: 'NCSIOEWE2364XXP65426W3RUGBRYOAGR3KMMCIA',
  };
  const response: AxiosResponse<AccountInfoDTO, any> = await accountRoutesApi.getAccountInfo(requestParameters);
  const dto: AccountInfoDTO = response.data;
  console.log(response.status);
  console.log(response.statusText);
  console.dir(dto, { depth: null });
})();

```

#### Node.js x JavaScript環境(typescript-axios版)

次にNode.js x JavaScript環境でサンプルコードの使用方法を説明します。
JavaScript向け環境でサンプルコードを試すためのディレクトリに移動して`npm i`してください。

```shell
cd examples/nodejs-javascript
npm install

```

まずはrequestParametersを指定する必要がない場合のサンプルコードを試しに実行してみましょう。

```shell
$ node api/NodeRoutesApi/getNodeInfo.js
# レスポンスは省略
```

実行したサンプルコードは以下の通りです。

```javascript:examples/nodejs-javascript/api/NodeRoutesApi/getNodeInfo.js
const symbolSdk = require('@nemtus/symbol-sdk-openapi-generator-typescript-axios');

(async () => {
  const configurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
  };
  const configuration = new symbolSdk.Configuration(configurationParameters);
  const nodeRoutesApi = new symbolSdk.NodeRoutesApi(configuration);
  const response = await nodeRoutesApi.getNodeInfo();
  console.log(response.status);
  console.log(response.statusText);
  console.dir(response.data, { depth: null });
})();

```

次に、requestParametersを指定する必要がある場合のサンプルコードも実行してみましょう。

```shell
$ node api/AccountRoutesApi/getAccountInfo.js
# レスポンスは省略
```

実行したサンプルコードは以下の通りです。

```javascript:examples/nodejs-javascript/api/AccountRoutesApi/getAccountInfo.js
const symbolSdk = require('@nemtus/symbol-sdk-openapi-generator-typescript-axios');

(async () => {
  const configurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
  };
  const configuration = new symbolSdk.Configuration(configurationParameters);
  const accountRoutesApi = new symbolSdk.AccountRoutesApi(configuration);
  const requestParameters = {
    accountId: 'NCSIOEWE2364XXP65426W3RUGBRYOAGR3KMMCIA',
  };
  const response = await accountRoutesApi.getAccountInfo(requestParameters);
  console.log(response.status);
  console.log(response.statusText);
  console.dir(response.data, { depth: null });
})();

```

#### Browser x CDN環境(typescript-axios版)

Browser上でCDNでSDKを読み込んでサンプルコードを試すためのディレクトリに移動して、

```shell
cd examples/browser-cdn

```

requestParametersを指定する必要がない場合のサンプルコードを試しに実行してみましょう。

以下コードのhtmlファイルをChrome等のブラウザで直接開いて開発者ツールのconsoleに結果が表示されているか確認してみましょう。

```html:examples/browser-cdn/api/NodeRoutesApi/getNodeInfo.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.jsdelivr.net/npm/@nemtus/symbol-sdk-openapi-generator-typescript-axios@0.1.0/index.min.js"></script>
  </head>
  <body>
    <script>
      (async () => {
        const symbolSdk = window.symbolSdkOpenAPIGeneratorTypeScriptAxios;
        const configurationParameters = {
          basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
        };
        const configuration = new symbolSdk.Configuration(configurationParameters);
        const nodeRoutesApi = new symbolSdk.NodeRoutesApi(configuration);
        const responseNodeInfo = await nodeRoutesApi.getNodeInfo();
        console.log(responseNodeInfo.status); // Example: 200
        console.log(responseNodeInfo.statusText); // Example: "OK"
        console.log(responseNodeInfo.data);
        // Example:
        /*
        {
          version: 16777987,
          publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
          networkGenerationHashSeed: '57F7DA205008026C776CB6AED843393F04CD458E0AA2D9F1D5F31A402072B2D6',
          roles: 3,
          port: 7900,
          networkIdentifier: 104,
          host: 'symbol-sakura-16.next-web-technology.com',
          friendlyName: 'next-web-technology',
          nodePublicKey: '9545F928A1B2FB4AC944BC1EC2F01FB84A503F6449B6BE3451B3F7A0F06B5BCF'
        }
        */
      })();
    </script>
  </body>
</html>

```

CDNからSDKを読み込むことで、グローバルな`window`に対して`window.symbolSdkOpenAPIGeneratorTypeScriptAxios`にSDKの全てが入っている状態になるので、それを使ってブラウザ上でそのままSDKを使ってREST APIを叩く処理を実行できるようになります。

requestParametersを指定する必要がある場合は以下のようになります。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.jsdelivr.net/npm/@nemtus/symbol-sdk-openapi-generator-typescript-axios@0.1.0/index.min.js"></script>
  </head>
  <body>
    <script>
      (async () => {
        const symbolSdk = window.symbolSdkOpenAPIGeneratorTypeScriptAxios;
        const configurationParameters = {
          basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
        };
        const configuration = new symbolSdk.Configuration(configurationParameters);
        const accountRoutesApi = new symbolSdk.AccountRoutesApi(configuration);
        const requestParameters = {
          accountId: 'NCSIOEWE2364XXP65426W3RUGBRYOAGR3KMMCIA',
        };
        const responseAccountInfo = await accountRoutesApi.getAccountInfo(requestParameters);
        console.log(responseAccountInfo.status); // Example: 200
        console.log(responseAccountInfo.statusText); // Example: "OK"
        console.log(responseAccountInfo.data);
        // Example:
        /*
        {
          account: {
            version: 1,
            address: '68A48712C4D6FDCBDDFEEF35EB6E3430638700D1DA98C120',
            addressHeight: '1',
            publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
            publicKeyHeight: '447',
            accountType: 1,
            supplementalPublicKeys: {
              linked: {
                publicKey: '5F87A37D1EAD570F4D0FD4C11A9D5EED5ABE82EF2E992B97CCDAC84F241470E0'
              },
              vrf: {
                publicKey: '806E9448598C922B371DA8CFD7E16E8F5F53594B3AECE13F0708778A4480A752'
              }
            },
            activityBuckets: [
              {
                startHeight: '1455840',
                totalFeesPaid: '0',
                beneficiaryCount: 1,
                rawScore: '476665546298'
              },
              {
                startHeight: '1455120',
                totalFeesPaid: '0',
                beneficiaryCount: 0,
                rawScore: '476672260900'
              },
              {
                startHeight: '1454400',
                totalFeesPaid: '0',
                beneficiaryCount: 0,
                rawScore: '476678670151'
              },
              {
                startHeight: '1453680',
                totalFeesPaid: '0',
                beneficiaryCount: 0,
                rawScore: '476685156832'
              },
              {
                startHeight: '1452960',
                totalFeesPaid: '0',
                beneficiaryCount: 0,
                rawScore: '476691995197'
              }
            ],
            mosaics: [
              { id: '6BED913FA20223F8', amount: '516465569230' },
              { id: '24F7CF825DBCDD42', amount: '499999886' },
              { id: '310378C18A140D1B', amount: '923' },
              { id: '6AE25FA5E8CA0646', amount: '1000000000' }
            ],
            importance: '476665546298',
            importanceHeight: '1455840'
          },
          id: '60517BE5CCA17918A561056D'
        }
        */
      })();
    </script>
  </body>
</html>

```

何らかの理由でnpm packageを使うことが難しいような時には、このようにCDNを簡易的に利用すると便利だと思います。(ただし、セキュリティ的にセンシティブな内容の場合は、読み込み先のCDNが改竄されていないか等、色々と注意を払う必要があるかもしれません。)

### typescript-fetch版

typescript-fetch版についても、使い方はほとんどtypescript-axiosと同じです。ただし、fetchがブラウザのみで使えて、Node.jsで使えないといったところや、レスポンスの型にちょっとした違いがあります。以下ではその違いについて簡単にふれながらサンプルコードを紹介します。
まずSDKのレポジトリをクローンして`npm i`するところは変わりありません。

```shell
git clone https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-fetch.git
cd symbol-sdk-openapi-generator-typescript-fetch
npm install

```

#### Node.js x TypeScript環境(typescript-fetch版)

TypeScript向け環境でサンプルコードを試すためのディレクトリに移動して`npm i`するところも同じです。

```shell
cd examples/nodejs-typescript
npm install

```

そしてrequestParametersを指定する必要がない場合のサンプルコードを試しに実行するところも同じです。

```shell
$ npx ts-node api/NodeRoutesApi/getNodeInfo.ts

200
OK
{
  version: 16777987,
  publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
  networkGenerationHashSeed: '57F7DA205008026C776CB6AED843393F04CD458E0AA2D9F1D5F31A402072B2D6',
  roles: 3,
  port: 7900,
  networkIdentifier: 104,
  host: 'symbol-sakura-16.next-web-technology.com',
  friendlyName: 'next-web-technology',
  nodePublicKey: '9545F928A1B2FB4AC944BC1EC2F01FB84A503F6449B6BE3451B3F7A0F06B5BCF'
}

```

実行したサンプルコードは以下の通りですが、ここには少し、違いやコツがあります。

typescript-fetch版のfetchはブラウザ上では使えますが、Node.js上では使えません。fetchはブラウザ上だけで使えるものだからです。

そのため、Node.js上でブラウザ上のfetchと同様の動作をするものに置き換えて実行してやる必要があります。具体的にはConfigurationParametersの中のfetchApiにfetchを置き換えるものを指定してやる必要があります。

ここではnode-fetchのv2系のものを使用して(そのままでは型が完全に一致しないエラーが出たのをあまり良くない方法で)無理やり実装していますが、ここはもっと適切な方法があるように感じました。もしご存知の方いらっしゃいましたら、ぜひContributeくださるととても嬉しいです。

```typescript:examples/nodejs-typescript/api/NodeRoutesApi/getNodeInfo.ts
import {
  Configuration,
  ConfigurationParameters,
  NodeInfoDTO,
  NodeRoutesApi,
  FetchAPI,
} from '@nemtus/symbol-sdk-openapi-generator-typescript-fetch';
import fetch from 'node-fetch'; // Note: Use version 2.x

(async () => {
  const configurationParameters: ConfigurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
    fetchApi: fetch as unknown as FetchAPI, // Note: Maybe there's a better way to do this.
  };
  const configuration: Configuration = new Configuration(configurationParameters);
  const nodeRoutesApi: NodeRoutesApi = new NodeRoutesApi(configuration);
  const response: NodeInfoDTO = await nodeRoutesApi.getNodeInfo();
  console.dir(response, { depth: null });
})();

```

なお、レスポンスの型を見ると、axiosの場合と異なり、レスポンスそのものが`DTO`で終わる型(DTOはData Type Objectの略称)になっていることに気づくと思います。(typescript-axios版はresponse.dataがDTOだったが、typescript-fetch版はresponseがDTOになっている。)

次にrequestParametersを指定する必要がある場合のサンプルコードを試しに実行してみます。

```shell
$ npx ts-node api/AccountRoutesApi/getAccountInfo.ts

{
  id: '60517BE5CCA17918A561056D',
  account: {
    version: 1,
    address: '68A48712C4D6FDCBDDFEEF35EB6E3430638700D1DA98C120',
    addressHeight: '1',
    publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
    publicKeyHeight: '447',
    accountType: 1,
    supplementalPublicKeys: {
      linked: {
        publicKey: '5F87A37D1EAD570F4D0FD4C11A9D5EED5ABE82EF2E992B97CCDAC84F241470E0'
      },
      node: undefined,
      vrf: {
        publicKey: '806E9448598C922B371DA8CFD7E16E8F5F53594B3AECE13F0708778A4480A752'
      },
      voting: undefined
    },
    activityBuckets: [
      {
        startHeight: '1465920',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476848133893'
      },
      {
        startHeight: '1465200',
        totalFeesPaid: '0',
        beneficiaryCount: 2,
        rawScore: '476783918668'
      },
      {
        startHeight: '1464480',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476790611323'
      },
      {
        startHeight: '1463760',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476796916067'
      },
      {
        startHeight: '1463040',
        totalFeesPaid: '0',
        beneficiaryCount: 0,
        rawScore: '476803888986'
      }
    ],
    mosaics: [
      { id: '6BED913FA20223F8', amount: '516727736888' },
      { id: '24F7CF825DBCDD42', amount: '499999886' },
      { id: '310378C18A140D1B', amount: '923' },
      { id: '6AE25FA5E8CA0646', amount: '1000000000' }
    ],
    importance: '476783918668',
    importanceHeight: '1465920'
  }
}
```

実行したサンプルコードは以下の通りです。requestParametersの型の名前がtypescript-axios版は`AccountRoutesApiGetAccountInfoRequest`だったのに対し、typescript-fetch版は`GetAccountInfoRequest`になっているという違いがあることに気づくと思います。

```typescript:examples/nodejs-typescript/api/AccountRoutesApi/getAccountInfo.ts
import {
  AccountInfoDTO,
  AccountRoutesApi,
  Configuration,
  ConfigurationParameters,
  FetchAPI,
  GetAccountInfoRequest,
} from '@nemtus/symbol-sdk-openapi-generator-typescript-fetch';
import fetch from 'node-fetch'; // Note: Use version 2.x

(async () => {
  const configurationParameters: ConfigurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
    fetchApi: fetch as unknown as FetchAPI, // Note: Maybe there's a better way to do this.
  };
  const configuration: Configuration = new Configuration(configurationParameters);
  const accountRoutesApi: AccountRoutesApi = new AccountRoutesApi(configuration);
  const requestParameters: GetAccountInfoRequest = {
    accountId: 'NCSIOEWE2364XXP65426W3RUGBRYOAGR3KMMCIA',
  };
  const response: AccountInfoDTO = await accountRoutesApi.getAccountInfo(requestParameters);
  console.dir(response, { depth: null });
})();

```

まとめるとtypescript-axios版と比べて、typescript-fetch版には以下のような違いがあるようでした。

- Node.js環境等の非ブラウザ環境ではfetchを置き換えてやる必要があること
- requestParametersが必要な場合、その型の名前が、typescript-axios版は`HogeRoutesApiGetFugaRequest`的な型の名前だったのに対し、typescript-fetch版は`GetFugaRequest`的な型の名前になっていること
- レスポンスの型がそのままData Type Objectになっていること

もちろん、typescript-fetch版についても、Node.js x JavaScript環境、Browser x CDN環境等で以下のように動作させることができます。

必要に応じて展開してご参照ください。

#### Node.js x JavaScript環境(typescript-fetch版)

<details>

```javascript: examples/nodejs-javascript/api/NodeRoutesApi/getNodeInfo.js
const symbolSdk = require('@nemtus/symbol-sdk-openapi-generator-typescript-fetch');
const fetch = require('node-fetch');

(async () => {
  const configurationParameters = {
    basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
    fetchApi: fetch,
  };
  const configuration = new symbolSdk.Configuration(configurationParameters);
  const nodeRoutesApi = new symbolSdk.NodeRoutesApi(configuration);
  const response = await nodeRoutesApi.getNodeInfo();
  console.dir(response, { depth: null });
  // Example:
  /*
  {
    version: 16777987,
    publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
    networkGenerationHashSeed: '57F7DA205008026C776CB6AED843393F04CD458E0AA2D9F1D5F31A402072B2D6',
    roles: 3,
    port: 7900,
    networkIdentifier: 104,
    host: 'symbol-sakura-16.next-web-technology.com',
    friendlyName: 'next-web-technology',
    nodePublicKey: '9545F928A1B2FB4AC944BC1EC2F01FB84A503F6449B6BE3451B3F7A0F06B5BCF'
  }
  */
})();

```

</details>

#### Browser x CDN環境(typescript-fetch版)

<details>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.jsdelivr.net/npm/@nemtus/symbol-sdk-openapi-generator-typescript-fetch@0.1.0/index.min.js"></script>
  </head>
  <body>
    <script>
      (async () => {
        const symbolSdk = window.symbolSdkOpenAPIGeneratorTypeScriptFetch;
        const configurationParameters = {
          basePath: 'http://symbol-sakura-16.next-web-technology.com:3000',
        };
        const configuration = new symbolSdk.Configuration(configurationParameters);
        const nodeRoutesApi = new symbolSdk.NodeRoutesApi(configuration);
        const responseNodeInfo = await nodeRoutesApi.getNodeInfo();
        console.log(responseNodeInfo);
        // Example:
        /*
        {
          version: 16777987,
          publicKey: 'B86304B01045894ED9250B3DCD6313DC2EC0DD529B4E864EA376A2F341D3CFD4',
          networkGenerationHashSeed: '57F7DA205008026C776CB6AED843393F04CD458E0AA2D9F1D5F31A402072B2D6',
          roles: 3,
          port: 7900,
          networkIdentifier: 104,
          host: 'symbol-sakura-16.next-web-technology.com',
          friendlyName: 'next-web-technology',
          nodePublicKey: '9545F928A1B2FB4AC944BC1EC2F01FB84A503F6449B6BE3451B3F7A0F06B5BCF'
        }
        */
      })();
    </script>
  </body>
</html>

```

</details>

## 今後の展望

元となっているSchemaをがっちり検証したわけではなく、ふわっと自動生成コードを作っただけといったレベル感ではありますが、第一目標としていた型支援の効くREST API clientという点では、ある程度網羅できたかなと感じています。

今後は以下のような取組を進めながら、Symbolブロックチェーンを用いたフロントエンド開発を、よりスムーズに進めていけるようなエコシステムの発展を、コア開発者やオフィシャルなレポジトリ上の実装のみに頼らず、我々コミュニティの側からも推進していけるよう、継続的に活動していきたいと考えています。

SDK開発の計画のスプレッドシート
[https://docs.google.com/spreadsheets/d/1s-F-Wy43R4JVeqzKB2gtVJZR_kpYXZIN2oAuh1CIyas/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1s-F-Wy43R4JVeqzKB2gtVJZR_kpYXZIN2oAuh1CIyas/edit?usp=sharing)

明確なマイルストーンを設けて計画的に進めていきたいと考えているもの

1. v0.1.0 ... 型支援の効くREST API clientの実装。今回の記事で紹介させてもらった段階である程度網羅できた認識。
2. v0.2.0 ... REST APIレスポンスの中で、データ形式の変換が必要で、新公式SDKで直接定義されていなかったり複雑な変換が必要な処理をwrapして提供する。
3. v1.0.0-alpha.1 ... トランザクションを送信するためのデータを生成する部分を型支援が効くようにする。(ただし、手作業で型を実装するのではなく、既存のデータから上手く自動生成できるような枠組みにしたい。)

常に少しずつ追加や改善を進めていくもの

- バグ修正
- サンプルコード等のドキュメントの作成
- テストコードの実装方針検討及び実装

NEMTUSでも頑張りますが、NEMTUSだけで頑張るのではなく、NEM/Symbolに関わるコミュニティの皆さまや、OSS活動への興味を持っている様々な方々の力をお借りして、より良いSDKにしていきながら、継続的に安定して維持していけるような取組にしていきたいと思います。

今後ともよろしくお願いします。

## 最後に

もしNEMTUSに対しNEMやSymbol関連記事の寄稿や、公開したSDKについて何かありましたら、以下GitHubにて記事やSDKを公開しておりますので、お気軽にDiscussionやIssueやPull Request等、連携くださいますと幸いです。どんな形のContributionも大歓迎です。

- [https://github.com/nemtus/](https://github.com/nemtus/)

NEMTUSとして、NEM, Symbolに関する様々な技術情報を継続的に発信していくとともにエコシステムへ貢献していきたいと考えていますので、今後ともどうぞよろしくお願いします。

## 記事作成者

- 名前
  - 松岡靖典
- 所属
  - NPO法人NEMTUS
    - [https://nemtus.com/](https://nemtus.com/)
- 略歴
  - 株式会社CauchyEでWeb開発やブロックチェーンに関する開発に従事した後、
  - 2022年8月からNPO法人NEMTUSにてブロックチェーン技術の普及推進活動に副理事長としてフルタイムで従事
- SNS
  - twitter: [https://twitter.com/salaryman_tousi](https://twitter.com/salaryman_tousi)
  - GitHub: [https://github.com/YasunoriMATSUOKA](https://github.com/YasunoriMATSUOKA)

## 参考資料

Dual Packageの構成については、以下記事と、前職で関わっていたCosmos SDK製ブロックチェーン向けSDKの構成を参考にさせて頂きました。

1. [Node.js Dual Packages (CommonJS/ES Modules) に対応した npm パッケージの開発](https://blog.cybozu.io/entry/2020/10/06/170000)
2. [@cosmos-client/core](https://github.com/cosmos-client/cosmos-client-ts)

CDN向けにwebpackでビルドして公開する方法については以下記事を参考にさせて頂きました。

1. [npmライブラリを公開する（TypeScript、CDN対応）](https://zenn.dev/nino_cast/articles/98a0a87f58026f)

この場を借りてお礼申し上げます。
