---
title: "Symbol x React その1 Reactでsymbol-sdkを使うための環境構築"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blockchain", "symbol", "react", "typescript", "rxjs"]
published: true
---

# Reactでsymbol-sdkを使うための環境構築

## 概要

この記事では、Reactで作成されたWebアプリ上でsymbol-sdkというnpmパッケージを用いてSymbolというブロックチェーンを利用するための第一歩となる環境構築について説明します。

以下URLでサンプルコードを実際に動かしているので、必要に応じてご参照ください。

- サンプルページ(ホームページ) [https://symbol-sample-angular.nemtus.com](https://symbol-sample-angular.nemtus.com)
- サンプルページ(アカウント情報表示ページ) [https://symbol-sample-angular.nemtus.com/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](https://symbol-sample-angular.nemtus.com/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA) ... URL末尾のアドレスを変更することで、任意のアドレスの情報を表示できます。
- サンプルページのGitHubレポジトリ [https://github.com/nemtus/symbol-sample-angular](https://github.com/nemtus/symbol-sample-angular)

## 要約

画像テスト
ペーこぺこぺこぺこぺこ！
![ぺこーら](/images/react-articles/0346E5C8-DFF9-4658-9D92-BF3B3CF96800.jpeg)

当記事は著作権、人格権、財産権を侵害する目的はございません。
報道、批評、研究を目的としており、記事内の素材は全て下記のいずれかに該当します
・当方が独自に作成したもの
・著作権法第30条2項に基づく「付随対象著作物の利用」
・著作権法第32条1項に基づく「引用」
・著作権法第41条に基づく「時事の事件の報道のための利用」
・著作権法第63条に基づく「許諾を得た著作物の利用」

## 前提となる環境

- OS
  - MacOS Big Sur
  - MacBook Pro (13-inch, M1, 2020)
  - Apple M1
- Node.js
  - v16.11.0
- npm
  - 8.0.0

## Symbolとは

Symbolとは、NEMブロックチェーンの新世代バージョンとして、長い開発期間を経て、2021年3月17日にローンチされた、パブリックブロックチェーンです。

Symbolの前バージョンのNEM(NIS1)には以下のような特徴がありました。

- 利用しやすいAPI
- 開発者が独自にコントラクトを実装せずとも利用可能なようにブロックチェーン自体に実装済の豊富な機能

Symbolでは、前バージョンのNEM(NIS1)の良さを継承しつつ、以下のような大幅な機能追加と性能改善と設計の見直しが行われました。

- 分散された多数のノードが自律的に維持されるようなインセンティブ設計
- スケーラビリティ等の性能改善
- 複数のトランザクションをまとめて実行する機能の追加によるトークンのトラストレスな交換等のサポート
- ブロックチェーン上に、より柔軟にデータを刻むことが可能な機能の追加

詳細は以下の公式なドキュメントをご参照ください。

- Symbol公式ドキュメント [https://docs.symbolplatform.com/ja/index.html](https://docs.symbolplatform.com/ja/index.html)
- NEM公式ドキュメント [https://nem.io/](https://nem.io/)

## symbol-sdkとは

Symbolブロックチェーンでは、APIノードが公開しているAPIを利用した開発を行うことになります。

API呼び出し等の処理を自前で実装しても良いのですが、以下のようにTypeScriptがサポートされたSDKがsymbol-sdkというnpmパッケージとして公式にリリースされているので、そちらを使用すると便利でしょう。

- symbol-sdk GitHubレポジトリ [https://github.com/symbol/symbol-sdk-typescript-javascript](https://github.com/symbol/symbol-sdk-typescript-javascript)
- symbol-sdk npmパッケージ [https://www.npmjs.com/package/symbol-sdk](https://www.npmjs.com/package/symbol-sdk)
- 公式ドキュメント symbol-sdk 使用方法ガイド アカウント情報取得 [https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html#method-01-using-the-sdk](https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html#method-01-using-the-sdk)

なお、symbol-sdkでは、後述するrxjsが積極的に使用されています。rxjsは使いこなすことができると非常に便利なのですが、学習コストは高めで、積極的な使用には賛否が分かれるところではあると思います。好みによってはPromiseに置き換えて使用するという戦略もあるかもしれませんので、案件に応じてご検討頂くとよろしいかと思います。

## rxjsとは

前述の通り、symbol-sdkの中では、rxjsが積極的に利用されています。

rxjsでは、非同期なデータを、Observableという川の流れのような概念に見立てて、流れを生み出す機能や、流れをモニタリングして上流から流れを受け取って加工した上で下流に流す機能や、流れをモニタリングして何かが流れてきたら何らかの処理を行う機能等が提供されています。

Promise, Thenの仕組みが1回限りの非同期処理の完了を待って次の作業を行うのと似ていますが、rxjsでは1回限りではなく連続した流れを継続的に扱うための仕組みが提供されているとも言えるでしょう。

rxjsは多くの方に取って、それなりに学習コストが大きいものと思いますが、データの流れや、各種オペレーターの動作をイメージできるようになると、REST APIやWeb Socket等の非同期な通信処理をきわめて柔軟に効率よく記述することが可能となります。

しかし、rxjsのオフィシャルなドキュメントを初見で読むと、処理の種類があまりにも豊富にあることと、言葉で説明された処理の概念や内容がとても意味不明に感じられてつらいと思います。

rxjs習得における個人的なおすすめは、基礎的な概念(≒Observableという概念)やメジャーなオペレーター(map, mergeMap, combineLatest)に絞って、以下のマーブル図や簡易的な説明でイメージを固めつつ、イメージが固まったら必要に応じて公式サイト等のドキュメントも参考にしつつsymbol-sdkやAngularのようなrxjsを使うコードをちょっとだけ書いてみて動作を試すのが良いと思います。

- rxjs マーブル図 [https://rxmarbles.com/](https://rxmarbles.com/)
- rxjs 公式サイト [https://rxjs.dev/](https://rxjs.dev/)

## Reactとは

ReactはJavascriptフレームワークです。

色々と記載はありますが、基本的にはウェブの画面を作れるようになるフレームワークです。
もちろん使い方によってはiosアプリケーションや、androidアプリケーションも作れるようになるアプリケーションです。

個人的な感想ですが、
AngularやVueやStencilなど色々フレームワークがありますが、
基本的には「好み」もしくは「案件」によりますが、正直学習コストもそんなに変わりませんし、
今回僕がReactの記事を記載するに当たっても「たまたまReactを案件で使ったことがあるから」
という理由なので、よくあるフレームワーク戦争に僕を巻き込むのはやめてください。
そういうのはTwitterで僕の目の届かないところでお願いします。

## Reactのアプリケーションを作ってみる

まずはnodeのバージョンの確認をしましょう。
基本的に安定バージョンが入っていれば問題はないのですが、
時たまバージョンの都合でうまくいきませんということもあるので、
その時はまずは落ち着きましょう！
![まだ慌てるような時間ではない](/images/react-articles/unnamed.jpeg)

```sh
$ node -v
v16.11.0
```

そうすると自動的にnpmもくっついてくるはずなので一応バージョン確認しておきましょう

```sh
$ npm -v
8.0.0
```

あとは好きな階層にディレクトリを作成しましょう。
僕はこんな感じでディレクトリの名前をreact_symbol_typescriptとしました。

![ディレクトリ（フォルダって言った方がいいかな？）](/images/react-articles/directory.png)

```sh
$ npx create-react-app . --template typescript --use-npm
npm を使用して色々とreactとtypescriptの最初のプロジェクトを作成していきます。
```

npx create-react-app .
この書き方で現在のディレクトリに作成するので、特に問題ないです。
これは僕の好みで実施しています

--template typescript
こちらはデフォルトをtypescriptにしてくれます。
Symbol SDKもTypescriptで書いているのでそっちに合わせましょう！と言ったところです。
正直TypeScriptは最初はとても難しいと思うようになるのですが、
これも要するに慣れなので、最初はいっぱいエラーが吐いてしんどいと思いますが、
慣れです。

この慣れという考えが非常に重要なので、エラーに対して悩むのではなく、
楽しんで行ってもらえますと幸いです。

Reactのアプリケーションができますと以下のメッセージが出ます

```sh
Happy hacking!
```

なので楽しく開発しましょう！！

## 初回起動

```sh
$ npm start
このコマンドでローカルのPC環境にアプリケーションを実行することができます
```

![初めてのローカルホスト](/images/react-articles/firstlocalhost.png)

[ここまでのURL](https://github.com/nemtus/symbol-sample-react)


## Tailwind CSS導入

必要なモジュールのインストール

```sh
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
npm install @craco/craco
```

package.json scriptの編集

```package.json script
{
    // ...
    "scripts": {
     "start": "craco start",
     "build": "craco build",
     "test": "craco test",
    // ...
    },
  }
```

![参考画像](/images/react-articles/carco.png)

craco.configの作成

```sh
touch craco.config.js
```

![参考画像](/images/react-articles/carco-config.png)

tailwind.config.js, postcss.config.jsの生成

```sh
$ npx tailwindcss init -p

Created Tailwind CSS config file: tailwind.config.js
Created PostCSS config file: postcss.config.js
```

tailwind.config.jsのpurge設定追加
tailwind.config.jsの中身をコピーしましょう

```tailwind.config.js
module.exports = {
    purge: ['./pages/**/*.{js,ts,jsx,tsx}', './components/**/*.{js,ts,jsx,tsx}'],
    darkMode: false,
    theme: {
        extend: {},
    },
    variants: {
        extend: {},
    },
    plugins: [],
}
```

![参考画像](/images/react-articles/tailwind-config.png)

./src/index.cssの編集

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

![参考画像](/images/react-articles/index-css.png)

prettierの設定 : settingsでRequire Config + Format On Saveにチェック

```sh
touch .prettierrc
```

```prettierrc
{
    "singleQuote": true,
    "semi": false
}
```

![参考画像](/images/react-articles/prettier.png)

:::message alert
デザインに関してはマテリアルデザインやその他色々ありますが、
現在僕はこのTailwindCSSを使用することが多くなっています。
正直なところデザインは「好み」の問題なので、好みが異なると批判が出ることも重々承知です。
言語やフレームワークについて、システムエンジニア同士でも争いをしているので「生産性の向上を語るシステムエンジニアが生産性に一切寄与しない言語間フレームワーク間闘争」をしていますが
正直僕はどっちでもいいですがこの「ツギハギ漂流作家」から大事なことを学びましょう！！
:::

![参考画像](/images/react-articles/blog1558.jpeg)

:::message
ポイントは「何で作るかではなく、何を作るか？」です。
:::

一度このタイミングでnpm startをしてみましょう。
エラーがなければよかったです！

```sh
npm start
```

:::message alert
./src/index.css (./node_modules/css-loader/dist/cjs.js??ref--5-oneOf-4-1!./node_modules/postcss-loader/src??postcss!./src/index.css)
Error: PostCSS plugin tailwindcss requires PostCSS 8.
Migration guide for end-users:
https://github.com/postcss/postcss/wiki/PostCSS-8-for-end-users

このエラーが出た場合は以下の対策をとってほしいです。
:::

一旦tailwindcssのバージョンを落とそうといった感じです。

```sh
npm uninstall tailwindcss postcss autoprefixer
npm install tailwindcss@npm:@tailwindcss/postcss7-compat @tailwindcss/postcss7-compat postcss@^7 autoprefixer@^9
```

現時点でのリポジトリはこちら
[tailwindcss導入](https://github.com/nemtus/symbol-sample-react/tree/tailwind-css)

## アプリケーション作成のためのタスク管理

さてさて、これで環境構築ができました。
正直ここを突破しないと何も続きができないので是非とも突破してほしいです。

これ以降はタスクを一つずつ作成していき、そのタスクを処理していきます。
これは個人的な感想ですが、タスクを作成して一つずつ処理していけばアプリはできるはずです。
ただそのタスクを考える力やタスクを無心となって処理する力
タスク自体の設定が間違っていたときに
「詰まる」という状況になります。

今回はGmailのアカウントを保有していればできる方法を採用します。

[タスク管理](https://docs.google.com/spreadsheets/d/1-WTAIUGgQmJ34JLCK3tkPtZqv57FRAL9hrGggieKvig/edit?usp=sharing)

## デザイン確認

今回はサンプルアプリケーション（松岡さんが作成されたもの）
をReactで作成する予定なのでデザインは簡単にできます。

:::message
個人的にデザインがイマイチだなと思った時がデザインを学ぶ時です。
僕も色々試行錯誤はします。
ですが最初は誰かが作ったものを真似するところからスタートします。
:::

以下デザインチェックを実施した資料になります。

[デザインチェック](https://docs.google.com/presentation/d/1ayIrqPLpvPRRpsTsl8JnDY632EaCRklmF8PQZXjsYLk/edit?usp=sharing)

## 機能確認

次にどういった機能が必要なのかを考えます。
機能的にはSymbol-SDKを使用したものが必要なのかな？という想定で
実施します。

- アカウントの生成
- パブリックキーの取得
- 保有モザイクの取得
- インポータンスの取得

の４つです。
なのでこの時点でsymbolの機能で作成すべき内容が決まりました。

タスク管理のオレンジの以下の部分を作成します。

[タスク管理](https://docs.google.com/spreadsheets/d/1-WTAIUGgQmJ34JLCK3tkPtZqv57FRAL9hrGggieKvig/edit?usp=sharing)

:::message
一人で作成する場合は機能開発からすべきなのか？
デザイン開発からすべきなのか？
僕は機能開発からしています。
あくまでも僕はです。
:::

## 機能開発のためにsymbol-sdkをインストールします

[ここにインストール方法を書いています](https://github.com/symbol/symbol-sdk-typescript-javascript)

```sh
npm install symbol-sdk rxjs
```

これでインストールができてから動作確認をしましょう

```sh
npm start
```

ここでエラーが出た場合は教えてくださいね。

[エラーはイシューとして作成して下さい](https://github.com/nemtus/symbol-sample-react/issues)

[現在はこんな感じです。](https://github.com/nemtus/symbol-sample-react/tree/symbol-sdk-rxjs-install)

## アカウントの生成

さてさてまずは機能開発を実施しましょう。
アドレスの生成です。

松岡さんの記事を参考にするのであればURLの後ろにアドレスを付与すればいいのですが、
今回はどうしようかな？
そうですね、テストネットを使用するので皆さん好きな方法を採用してみてください。

そもそもアカウントの生成方法はどのような方法があるのか？という話を考えます。

Symbol-SDKでアカウントを作成する方法はいくつかあります。

というわけなのでまずは機能を確認するだけなのでsrc/App.tsxにsymbol-sdkをimportしましょう。

```src/App.tsx
import { Account } from 'symbol-sdk'
```

![Accountでできること](/images/react-articles/Account.png)

はい、というわけでアカウントの作成については
写真のようにcreateFromPrivateKeyとgenerateNewAccountの二つの方法があります。

[参考資料](https://docs.symbolplatform.com/ja/guides/account/creating-an-account.html)

アドレスの表示については他の方法があるのですが、まずはアカウントの生成の部分を確認していきます。
おそらくウォレットを何か作りたいなぁとなった時はまずは

- 新規アカウントを作成するか
- 秘密鍵からアカウントを作成するか？

の２つの手段がとられると思います。

:::message
アプローチは人の数だけあると思いますが、
今回はこんな感じです。
:::

### 新規アカウントを作る方法

Symbol-SDKからアカウントを新規に作成する方法は
AccountとNetworkTypeが必要になります

```src/App.tsx
import logo from './logo.svg'
import './App.css'
import { Account, NetworkType } from 'symbol-sdk'

function App() {
  const accountCreate = () => {
    const account = Account.generateNewAccount(NetworkType.TEST_NET)
    console.log(
      'Your new account address is:',
      account.address.pretty(),
      'and its private key',
      account.privateKey
    )
  }

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <button onClick={accountCreate}>アカウントの作成</button>
      </header>
    </div>
  )
}

export default App
```

さてこれを実行するとこんな感じになります

![アカウントの生成](/images/react-articles/accountGenerate.png)

アカウントの作成という文字がボタンになっています。

まずはChromeの検証（inspector）をクリックします。

![検証](/images/react-articles/Inspecter.png)

そうすると「コンソール」もしくは「console」という項目がありますのでクリックします。

![コンソール](/images/react-articles/console.png)

これでアプリケーション内にconsoleに出力するという命令があるとここに表示されます。

現時点ではsrc/App.tsx内に存在するaccountCreateの関数にconsole.logという関数が実装されています。
ここでコンソールに出力するという命令になっています。

この関数の流れとしては

1. まずはアカウントを生成してaccountという定数の中に格納します
2. 次にコンソールログで'Your new account address is: アカウントのアドレスの文字列を表示'となっています
3. また最後には'and its private key アカウントのプライベートキーを表示'

という処理が実施されます。

```src/App.tsx
const accountCreate = () => {
    const account = Account.generateNewAccount(NetworkType.TEST_NET)
    console.log(
      'Your new account address is:',
      account.address.pretty(),
      'and its private key',
      account.privateKey
    )
  }
```

それではこのアカウントを生成のボタンを押してみます。
どうなるかな？

```console
Your new account address is: TDK7FE-VBYAE7-BHNFPK-DWXTIL-HJJ7G2-U6MWMJ-6CY and its private key A03B6B24549989C381A88149E18AF8C7B2E2639C1CE919E6B659A1F3C8C307E7
```

となっています。

![結果](/images/react-articles/accountGenerateResult.png)

このようになっていればOKです！！

ちなみにaddress.pretty()となっているところを少しいじるとこんな感じになります

account.addressの時

```result
Your new account address is: Address {address: 'TDJNAYAZY7EJJIVQX7UVXFDR4F7PHLRWWJGUSBY', networkType: 152}address: "TDJNAYAZY7EJJIVQX7UVXFDR4F7PHLRWWJGUSBY"networkType: 152[[Prototype]]: Object and its private key A51C696BE2102A36F766222C8B5305AD4EA52C9FD325DDCECE0A2C0D7326B7B2
```

account.networkTypeの時

```result
Your new account address is: 152 and its private key 651230EBAA228E9A1C306F2DECD04C16483B049E6245B5BF3F703189351FE676
```

まぁ色々機能があるんで試してみてください！！

:::message
重要なことは色々試してみることです！
:::

![試した結果](/images/react-articles/somethingTest.png)

### 秘密鍵から生成する方法

さてアカウントを生成する方法はわかったので次は既存のアカウントを生成する方法です。
symbolブロックチェーンではアカウントの生成には秘密鍵を使用することができます。

イメージとしては他のサービスで生成したアカウントの秘密鍵を自分が作成しているアプリケーションに「復元」するイメージの方がいいかなと思っています。

![アカウントの生成イメージ](/images/react-articles/AccountGenerateImage.png)

それでは実装していきましょう。

この手で僕はどのような手順で実装していくのかと言いますと。

まずは

1. デスクトップウォレットでアカウントを作成
2. デスクトップウォレットから秘密鍵を取得
3. Reactのアプリケーションで復元できるか確認
4. useStateを使用して秘密鍵の文字列を入力してから復元できるか確認

この４つのタスクを処理できれば秘密鍵からアカウントを生成する方法は身につくはずです。

ではやってみましょう。

![デスクトップウォレット](/images/react-articles/symbol-desktop-wallet.png)

はい、こちらデスクトップウォレットです。

デスクトップウォレットからアカウントを作成します。

今回はシードアカウント１というアカウントを使用します。

![シードアカウント](/images/react-articles/seedAccount.png)

そこでPrivate Keyとなっているところのshowを選択して
パスワードを入力します。

そうすると秘密鍵が表示されるのでコピーをしましょう。

:::message alert
秘密鍵を公開するのはやめましょう。
後悔しますよ。
こうかいだけに。
:::

さてさて、これで秘密鍵は取得できました。
次は秘密鍵からアカウントを復元する必要があります。

[参考にするソースコードはこちら](https://github.com/symbol/symbol-docs/blob/main/source/resources/examples/typescript/account/OpeningAnAccount.ts)

それでは作っていきましょう。

秘密鍵からアカウントを復元するには２つの材料が必要です。

1. 秘密鍵
2. ネットワークタイプ

```src/App.tsx
  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      "7B20E0615755D6EEDA0DAB45E5D8A4331EC603F8702D7F4E6171FB81CF83CF78",
      NetworkType.TEST_NET
    )
    console.log(
      'Your account address is:',
      account.address.pretty(),
      'and its private key',
      account.privateKey
    )
  }
```

:::message alert
良い子のみんなは秘密鍵を公開しないようにしましょう。
後悔します。
こうかいだけに。
:::

関数を作成したので、これをボタンで呼び出しましょう。

```src/App.tsx
return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <button onClick={accountCreate}>アカウントの作成</button>
        <button onClick={accountCreateFromPrivateKey}>秘密鍵からアカウントを作成する</button> {/* ここが追加されるよ！！ */}
      </header>
    </div>
  )
```

さてさて、動作確認をしましょう。

できてますね。

![プライベートキーからアカウントを生成する](/images/react-articles/privateKeyGenerate.png)

アカウントの文字列が合っているか確認します。

![アカウントのアドレスの確認](/images/react-articles/privateKeyGenerateConsole.png)

合ってますね。

これで秘密鍵からアカウントを生成することができました！！

さてさてそれでは二つをコンポーネント化します。

理由としては次の作業にとても邪魔になるからです。（個人差はありますので、別に気にならない人はそれでいいかと思います）

:::message
アプリケーションのコードの整理方法には色々あります。
オニオンフレームワークとか
あと、ドメイン駆動型とか
ですが、まずは「気持ちのいい開発方法」を探してもらえますと幸いです。
所詮こういったのは全て「整理整頓」です。
夫婦でも洗濯物の畳み方で喧嘩するぐらい人間の「整理整頓」の価値観は様々です。
なのでまずは自分の気持ちいい開発手法を実施してみましょう。
この開発手法での戦争はAKBの中で誰がいいのか？ぐらいどうでもいい話です。
生産性はまずは開発しやすい心理状態を作るところからスタートです。（結構ここをバカにしているエンジニアは多いので誰かと一緒に作る際は要注意です。
:::

:::message alert
仕事の場合はちょっと聞きかじったエンジニアがDDDとか言ってきますが、
その時はお賃金分だけ頑張りましょう！！
:::

さて、コンポーネント化ですが、さっきも言ったように
ただの整理整頓です。
今はアタッシュケースの中に乱雑に入っている感じですが、それを靴下の場所、パンツの場所、シャツの場所
みたいに整理することです。

![コンポーネント化](/images/react-articles/componentism.png)

まぁ整理するとそら取り出しやすいよね。
みたいな感じです。

ただ、これはあくまでも個人単位で実施した方がいいです。
整理整頓と一つにしても前職の障がい支援では自閉スペクトラム症の方での整理整頓のルールが異なっていたので
まずは「個人単位」です。

:::message alert
仕事の場合はお賃金分だけ頑張りましょう！！
:::

さて、それではこの画像のようにコンポーネント化するためのディレクトリ（フォルダ）とファイルを作成しましょう。

ターミナルでsrcの下の階層にcomponentという名前のディレクトリを作成します。

```sh
react_symbol_typescript$ mkdir src/component
```

次にファイルを作成します。
先ほど作成したcomponentのディレクトリの階層の下にファイルを二つ作成します。
名前はGenerateNewAccount.tsxとCreateFromPrivateKey.tsxの二つです。

```sh
react_symbol_typescript$ touch src/component/GenerateNewAccount.tsx
react_symbol_typescript$ touch src/component/CreateFromPrivateKey.tsx
```

![こんな感じになります](/images/react-articles/componentFile.png)

:::message
VSCodeの方は好みでこのVSCodeの拡張機能を使うといい感じになります。
どういい感じになるかは使ってみてのお楽しみ
https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets
今回のようにコンポーネントファイルを作成した後にファイルでrafceと入力するとこんな感じになります。
:::

![rafce](/images/react-articles/rafce.png)

```src/component/CreateFromPrivateKey.tsx
import React from 'react'

const CreateFromPrivateKey = () => {
  return (
    <div>
      
    </div>
  )
}

export default CreateFromPrivateKey

```

コンポーネントの初期状態を自動で記載してくれます。（これ結構僕気に入っています。）

さて、移植しましょう。

```src/component/CreateFromPrivateKey.tsx
import React from 'react'
import { Account, NetworkType } from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      '7B20E0615755D6EEDA0DAB45E5D8A4331EC603F8702D7F4E6171FB81CF83CF78',
      NetworkType.TEST_NET
    )
    console.log(
      'Your account address is:',
      account.address.pretty(),
      'and its private key',
      account.privateKey
    )
  }
  return (
    <div>
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>{' '}
      {/* ここが追加されるよ！！ */}
    </div>
  )
}

export default CreateFromPrivateKey

```

```src/component/GenerateNewAccount.tsx
import React from 'react'
import { Account, NetworkType } from 'symbol-sdk'

const GenerateNewAccount = () => {
  const accountCreate = () => {
    const account = Account.generateNewAccount(NetworkType.TEST_NET)
    console.log(
      'Your new account address is:',
      account.address.pretty(),
      'and its private key',
      account.privateKey
    )
  }
  return (
    <div>
      <button onClick={accountCreate}>アカウントの作成</button>
    </div>
  )
}

export default GenerateNewAccount

```

```src/App.tsx
import logo from './logo.svg'
import './App.css'


function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
      </header>
    </div>
  )
}

export default App
```

さてこれで移植ができましたのであとはsrc/App.tsxで呼び出す必要があります。

```src/App.tsx
import logo from './logo.svg'
import './App.css'
import CreateFromPrivateKey from './component/CreateFromPrivateKey'
import GenerateNewAccount from './component/GenerateNewAccount'


function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <CreateFromPrivateKey></CreateFromPrivateKey>
        <GenerateNewAccount></GenerateNewAccount>
      </header>
    </div>
  )
}

export default App
```

先ほどの違いとして、秘密鍵からアカウントを生成する方法とアカウントを新しく生成する方法の順番を入れ替えています。

![こんな感じになります](/images/react-articles/finishComponent.png)

それぞれのボタンを押して動くか確認してくださいね！

[ここまでの成果物](https://github.com/nemtus/symbol-sample-react/tree/address-create)

## アカウント情報表示ページの実装

## まとめ

## 最後に

もしNEMTUSに対しNEMやSymbol関連記事の寄稿や、サンプルとして公開したアプリについて何かありましたら、以下GitHubにて記事やサンプルアプリを公開しておりますので、お気軽にIssueやPull Request等、連携くださいますと幸いです。

- [https://github.com/nemtus/](https://github.com/nemtus/)

NEMTUSとして、NEM, Symbolに関する様々な技術情報を継続的に発信していきたいと考えていますので、今後ともどうぞよろしくお願いします。

## 記事作成者

- 名前
  - 松本一将
- 所属
  - 株式会社Opening Line
    - [https://www.opening-line.co.jp/](https://www.opening-line.co.jp/)
  - NPO法人NEMTUS
    - [https://nemtus.com/](https://nemtus.com/)
- 略歴
  - 障がい者向けアプリケーションを作ろうとして、頑張った結果ブロックチェーンの企業で働いています。
- SNS
  - twitter: [https://twitter.com/salaryman_tousi](https://twitter.com/salaryman_tousi)
  - GitHub: [https://github.com/YasunoriMATSUOKA](https://github.com/YasunoriMATSUOKA)
