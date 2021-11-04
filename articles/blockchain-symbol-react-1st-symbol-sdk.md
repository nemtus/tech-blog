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

サンプルページは後ほどURLを共有

[リポジトリ](https://github.com/nemtus/symbol-sample-react)

## 要約

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

[記事の参照](https://zenn.dev/nemtus/articles/blockchain-symbol-angular-1st-symbol-sdk#symbol%E3%81%A8%E3%81%AF)

## symbol-sdkとは

[記事の参照](https://zenn.dev/nemtus/articles/blockchain-symbol-angular-1st-symbol-sdk#symbol-sdk%E3%81%A8%E3%81%AF)

## rxjsとは

[記事の参照](https://zenn.dev/nemtus/articles/blockchain-symbol-angular-1st-symbol-sdk#rxjs%E3%81%A8%E3%81%AF)

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
![まだ慌てるような時間ではない](/images/react-articles/madaawateru.png)

```sh:terminal
$ node -v
v16.11.0
```

そうすると自動的にnpmもくっついてくるはずなので一応バージョン確認しておきましょう

```sh:terminal
$ npm -v
8.0.0
```

あとは好きな階層にディレクトリを作成しましょう。
僕はこんな感じでディレクトリの名前をreact_symbol_typescriptとしました。

![ディレクトリ（フォルダって言った方がいいかな？）](/images/react-articles/directory.png)

```sh:terminal
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

```sh:terminal
Happy hacking!
```

なので楽しく開発しましょう！！

## 初回起動

```sh:terminal
$ npm start
このコマンドでローカルのPC環境にアプリケーションを実行することができます
```

![初めてのローカルホスト](/images/react-articles/firstlocalhost.png)

[ここまでのURL](https://github.com/nemtus/symbol-sample-react)


## Tailwind CSS導入

必要なモジュールのインストール

```sh:terminal
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
npm install @craco/craco
```

package.json scriptの編集

```json:package.json script
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

```sh:terminal
touch craco.config.js
```

``` js:craco.config.js
module.exports = {
  style: {
    postcss: {
      plugins: [
        require('tailwindcss'),
        require('autoprefixer'),
      ],
    },
  },
}
```

![参考画像](/images/react-articles/carco-config.png)

tailwind.config.js, postcss.config.jsの生成

```sh:terminal
$ npx tailwindcss init -p

Created Tailwind CSS config file: tailwind.config.js
Created PostCSS config file: postcss.config.js
```

tailwind.config.jsのpurge設定追加
tailwind.config.jsの中身をコピーしましょう

```js:tailwind.config.js
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

``` css:index.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

![参考画像](/images/react-articles/index-css.png)

prettierの設定 : settingsでRequire Config + Format On Saveにチェック

```sh:terminal
touch .prettierrc
```

```json:prettierrc
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

![参考画像](/images/react-articles/tugihagi.png)

:::message
ポイントは「何で作るかではなく、何を作るか？」です。
:::

一度このタイミングでnpm startをしてみましょう。
エラーがなければよかったです！

```sh:terminal
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

```sh:terminal
npm uninstall tailwindcss postcss autoprefixer
npm install tailwindcss@npm:@tailwindcss/postcss7-compat @tailwindcss/postcss7-compat postcss@^7 autoprefixer@^9
```

現時点でのリポジトリはこちら
[tailwindcss導入](https://github.com/nemtus/symbol-sample-react/tree/tailwind-css)

### この章のまとめ

環境構築は結構大事です。

## アプリケーション作成のためのタスク管理

さてさて、これで環境構築ができました。
正直ここを突破しないと何も続きができないので是非とも突破してほしいです。

これ以降はタスクを一つずつ作成していき、そのタスクを処理していきます。
これは個人的な感想ですが、タスクを作成して一つずつ処理していけばアプリはできるはずです。
ただそのタスクを考える力やタスクを無心となって処理する力
タスク自体の設定が間違っていたときに
「詰まる」という状況になります。

今回の記事ではタスク管理にGoogleスプレッドシートを使用しました。
Googleアカウントがあれば今回の記事でのタスク一覧を以下リンクから自由に参照できますので、必要に応じてご参照ください。

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

### この章のまとめ

何を作るかを書き出すのはとても大事！！

## 機能確認

次にどういった機能が必要なのかを考えます。
機能的にはSymbol SDKを使用したものが必要なのかな？という想定で
実施します。

- アカウントの生成
- アドレス、パブリックキーの取得
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

```sh:terminal
npm install symbol-sdk rxjs
```

これでインストールができてから動作確認をしましょう

```sh:terminal
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

Symbol SDKでアカウントを作成する方法はいくつかあります。

というわけなのでまずは機能を確認するだけなのでsrc/App.tsxにSymbol SDKをimportしましょう。

```tsx:src/App.tsx
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

Symbol SDKからアカウントを新規に作成する方法は
AccountとNetworkTypeが必要になります

```tsx:src/App.tsx
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

```tsx:src/App.tsx
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

``` result:console
Your new account address is: TDK7FE-VBYAE7-BHNFPK-DWXTIL-HJJ7G2-U6MWMJ-6CY and its private key A03B6B24549989C381A88149E18AF8C7B2E2639C1CE919E6B659A1F3C8C307E7
```

となっています。

![結果](/images/react-articles/accountGenerateResult.png)

このようになっていればOKです！！

ちなみにaddress.pretty()となっているところを少しいじるとこんな感じになります

account.addressの時

``` result:console
Your new account address is: Address {address: 'TDJNAYAZY7EJJIVQX7UVXFDR4F7PHLRWWJGUSBY', networkType: 152}address: "TDJNAYAZY7EJJIVQX7UVXFDR4F7PHLRWWJGUSBY"networkType: 152[[Prototype]]: Object and its private key A51C696BE2102A36F766222C8B5305AD4EA52C9FD325DDCECE0A2C0D7326B7B2
```

account.networkTypeの時

``` result:console
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

```tsx:src/App.tsx
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

```tsx:src/App.tsx
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

```sh:terminal
react_symbol_typescript$ mkdir src/component
```

次にファイルを作成します。
先ほど作成したcomponentのディレクトリの階層の下にファイルを二つ作成します。
名前はGenerateNewAccount.tsxとCreateFromPrivateKey.tsxの二つです。

``` sh:terminal
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

```tsx:src/component/CreateFromPrivateKey.tsx
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

```tsx:src/component/CreateFromPrivateKey.tsx
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

``` tsx:src/component/GenerateNewAccount.tsx
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

``` tsx:src/App.tsx
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

``` tsx:src/App.tsx
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

## useState

さて、いい感じにReactにも慣れてきたと思います。
（えっ？慣れていない？そういう時は二郎でも食べにいきましょう！！）

次は秘密鍵を入力するとそのアカウントが作成される部分を作りたいと思います。

先ほど作った秘密鍵の生成のところは毎回文字列をアプリケーションに設定してから生成しました。

それだと・・・

「使い勝手が悪い。」

となったりします（要件次第）

なので皆さんの秘密鍵を入力してそれぞれのアカウントを生成できるようにします。

ワクワクしてきましたね。

[ワクワク！！](https://www.youtube.com/watch?v=WEFCEjDGc64)

そこで使用するのがuseStateというReactの機能です。色々とややこしい説明は省きます、
ここで覚えて欲しいのは

「useStateを使用すれば変数とその内容を変更することができる」です。

今回は秘密鍵の文字列を入力してその秘密鍵を参照して関数を実行するという処理が必要です。

![useStateで実施したい処理](/images/react-articles/useState.png)

この２つを簡単にしてくれるのがuseStateです。

:::message
変数をよく箱で表されるのは変数はもともとメモリの何番目から何番目という区間を指定して
そのメモリ領域を確保するというところから来ているのではないかと僕は勝手に解釈してます。
なのでメモリ領域を箱としてその中身を入れ替えたり、その箱の中身をコピーしたりと色々です。
:::

さてではuseStateを使ってみましょう。

``` src/component/CreateFromPrivateKey.tsx
const [privateKey, setPrivateKey] = useState("");
```

この左のprivateKeyが変数宣言です。
その横のsetPrivateKeyが入れ替える関数です。
右のuseState()でuseStateを使用します。
そのカッコの中で箱の形を宣言できます。

なのでこれで試しに関数とボタンを作成してみましょう。

``` tsx:src/component/CreateFromPrivateKey.tsx
import React, { useState } from 'react'
import { Account, NetworkType } from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  console.log(privateKey)

  const sampleUseState = () => {
    setPrivateKey(
      '7B20E0615755D6EEDA0DAB45E5D8A4331EC603F8702D7F4E6171FB81CF83CF78'
    )
  }

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
      </button>
      <br />
      <button onClick={sampleUseState}>useStateを試してみる</button>
    </div>
  )
}

export default CreateFromPrivateKey

```

このsampleUseStateという関数を作成しました。
これで実行してみます。

最初の状態では秘密鍵の中はからですが、

![最初の状態](/images/react-articles/useStateConsoleBefore.png)

ボタンを押した後には値が表示されています。

![ボタンを押した後](/images/react-articles/useStateConsoleAfter.png)

こんな感じでいい具合に実施します。

というわけでこのsetStateを実施した後にアカウントが生成される処理を作ります。

次はuseStateを使った変数のprivateKeyという変数を使ってアカウントを生成してみます。

秘密鍵の文字列のところにprivateと入力します。

``` tsx:src/component/CreateFromPrivateKey.tsx
import React, { useState } from 'react'
import { Account, NetworkType } from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  console.log('秘密鍵', privateKey)

  const sampleUseState = () => {
    setPrivateKey(
      '7B20E0615755D6EEDA0DAB45E5D8A4331EC603F8702D7F4E6171FB81CF83CF78'
    )
  }

  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
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
      </button>
      <br />
      <button onClick={sampleUseState}>useStateを試してみる</button>
    </div>
  )
}

export default CreateFromPrivateKey
```

できた！！

![できた！](/images/react-articles/variablePrivateKeySetting.png)

流れはuseStateのボタンを押して秘密鍵からのボタンを押します。

## 入力エリアを設定

というわけでuseStateの部分も理解ができたので次は入力エリアの設定をします。

まずは入力エリアを作成します。

``` tsx:src/components/CreateFromPrivateKey.tsx
  return (
    <div>
      <input className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline" />
      <br />
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>
      <br />
      <button onClick={sampleUseState}>useStateを試してみる</button>
    </div>
  )
```

ここでtailwindcssの紹介

[Tailwindcss](https://v1.tailwindcss.com/components/forms)

Tailwindcssはクラスに指定する感じでいい感じのデザインにしてくれるいい感じのものです。
最初の環境構築の時に設定したものがいい感じに効果を発揮してくれます。

なので[このサイト](https://v1.tailwindcss.com/components/forms)へ訪れていい感じのデザインを使ってしまいましょう！！

現時点ではこのような形になります。

![入力エリア追加](/images/react-articles/inputform.png)

さて入力エリアはできたので、次は入力した値を反映させる必要があります。

手順は以下の流れです。

1. 入力値を渡せるようにする
2. 入力値をconsole.logで確認する
3. 実際の秘密鍵を入力してアカウントを作成してみる

では確認していきましょう。

まず実施したいことは入力した値を変数に格納します。
先ほど作成したuseStateを使用したprivateKeyに格納します。

``` tsx:src/components/CreateFromPrivateKey.tsx
const [privateKey, setPrivateKey] = useState('')
```

privateKeyの中に新たに値を更新したい時はsetPrivateKeyを使用します

onChangeのプロパティを実行して
(e)の中には更新した内容が入っています。
その中でe.target.valueが入力した内容になっています。

``` tsx:src/components/CreateFromPrivateKey.tsx
<input 
  onChange={(e) => setPrivateKey(e.target.value)}
  className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline" />
```

![初期値](/images/react-articles/onChange.png)

さてコンソールのところですが、秘密鍵の後には空白が入っています。
ではこの中にtestと入力してみます。

![入力後](/images/react-articles/onChangeAfter.png)

その時のコンソールを見てみます。

![コンソール](/images/react-articles/onChangeConsole.png)

入力した文字が一文字ずつ更新されています

こういった形でいい具合にprivateKeyに入力した文字を格納することができました。

:::message
これは入力値が更新される度にこのCreateFromPrivateKey.tsxが更新されると思ってください。
毎回更新されるのを防ぐためにはevent.preventDefaultを使用してください
:::

[イベントプリベントデフォルト](https://developer.mozilla.org/ja/docs/Web/API/Event/preventDefault)

最後に秘密鍵を入力してみましょう。

できました！！

![完成](/images/react-articles/onChangeFinish.png)

さて進捗を確認しましょう。

現時点でアドレスの生成は完了しました！！（おめでとう！！）

[進捗](https://docs.google.com/spreadsheets/d/1-WTAIUGgQmJ34JLCK3tkPtZqv57FRAL9hrGggieKvig/edit?usp=sharing)

[この時点でのソースコード](https://github.com/nemtus/symbol-sample-react/tree/use-state)

## パブリックキーの取得

まずは不要なコンポーネントを非表示にします。
コメントアウトしているところは削除してもらって大丈夫です！！

``` tsx:src/App.tsx
import logo from './logo.svg'
import './App.css'
import CreateFromPrivateKey from './component/CreateFromPrivateKey'
// import GenerateNewAccount from './component/GenerateNewAccount'

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <CreateFromPrivateKey></CreateFromPrivateKey>
        {/* <GenerateNewAccount></GenerateNewAccount> */}
      </header>
    </div>
  )
}

export default App
```

``` tsx:src/components/CreateFromPrivateKey.tsx
import React, { useState } from 'react'
import { Account, NetworkType } from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  console.log('秘密鍵', privateKey)

  // const sampleUseState = () => {
  //   setPrivateKey(
  //     '7B20E0615755D6EEDA0DAB45E5D8A4331EC603F8702D7F4E6171FB81CF83CF78'
  //   )
  // }

  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
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
      <input 
      onChange={(e) => setPrivateKey(e.target.value)}
      className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline" />
      <br />
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>
      {/* <br />
      <button onClick={sampleUseState}>useStateを試してみる</button> */}
    </div>
  )
}

export default CreateFromPrivateKey
```

さて、続きです。

今回はアカウントと公開鍵を取得します。

以下手順です。

1. アドレスの取得方法を確認する
2. アドレスを取得する
3. 公開鍵を取得する方法を確認する
4. 公開鍵を取得する
5. アプリケーションに表示する

さて実施しましょう。

まずはアドレスの取得方法です。

アドレスは前回のところで実施しました、秘密鍵の生成から
account.address.pretty()で取得できたことを覚えていますか？

![アドレスの取得](/images/react-articles/privateKeyGenerateConsole.png)

これでアドレスは取得できました。

次に公開鍵を取得します

公開鍵の取得方法は秘密鍵から生成した時に

account.publicKey

で取得できます。

``` tsx:src/components/CreateFromPrivateKey.tsx
const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
      NetworkType.TEST_NET
    )
    console.log(
      'アドレス',
      account.address.pretty(),
      '公開鍵',
      account.publicKey,
      '秘密鍵',
      account.privateKey
    )
  }
```

![公開鍵を取得](/images/react-articles/getPublicKey.png)

さて、これでアドレスと公開鍵が取得できましたのであとはアプリケーションに表示します。

ここで先ほど使用したuseStateを使用します。

``` tsx:src/components/CreateFromPrivateKey.tsx
import React, { useState } from 'react'
import { Account, NetworkType } from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  const [address, setAddress] = useState('')
  const [publicKey, setPublicKey] = useState('')
  console.log('秘密鍵', privateKey)

  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
      NetworkType.TEST_NET
    )
    setAddress(account.address.pretty())
    setPublicKey(account.publicKey)
  }
  return (
    <div>
      <input
        onChange={(e) => setPrivateKey(e.target.value)}
        className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline"
      />
      <br />
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>
      <p>アドレス: {address}</p>
      <p>公開鍵: {publicKey}</p>
    </div>
  )
}

export default CreateFromPrivateKey
```

まずは箱を設定するuseStateを二つ用意します。

``` tsx:src/components/CreateFromPrivateKey.tsx
  const [address, setAddress] = useState('')
  const [publicKey, setPublicKey] = useState('')
```

アカウントを作成するアドレスや公開鍵が取得できるので
取得した内容をsetAddressやsetPublicKeyを使用して
変数の中身を入れ替えます。

``` tsx:src/components/CreateFromPrivateKey.tsx
  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
      NetworkType.TEST_NET
    )
    setAddress(account.address.pretty())
    setPublicKey(account.publicKey)
  }
```

あとはuseStateの変数を表示します。
変数を表示する場合は{}（カーリーブラケットもしくは波括弧）を使用します。

``` tsx:src/components/CreateFromPrivateKey.tsx
  <button onClick={accountCreateFromPrivateKey}>
    秘密鍵からアカウントを作成する
  </button>
  <p>アドレス: {address}</p>
  <p>公開鍵: {publicKey}</p>
```

こんな感じで表示できました。

![アドレスとパブリックキーを取得](/images/react-articles/addressPublicKeyGet.png)

[進捗](https://docs.google.com/spreadsheets/d/1-WTAIUGgQmJ34JLCK3tkPtZqv57FRAL9hrGggieKvig/edit?usp=sharing)

[現在のソースコード](https://github.com/nemtus/symbol-sample-react/tree/publickey_get)

## 保有モザイクとインポータンスの取得

さて、次は保有モザイクとインポータンスの取得です。

こういった情報はアカウント情報として全て格納されていますので（便利ですね）

なので手順としては

1. アカウント情報の取得方法を確認する
2. 実際に取得してみる
3. 表示する

の３ステップになります。

頑張ってやっていきましょう！！

さてアカウントの取得方法はこちらです。

[アカウント情報の取得](https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html)

[コード](https://github.com/symbol/symbol-docs/blob/main/source/resources/examples/typescript/account/GettingAccountInformation.ts)

取得する方法を確認しましょう。

アカウント情報を取得するために必要な要素は合計で5つです。

1. アドレス
2. アカウント
3. ノードURL
4. リポジトリファクトリ
5. アカウントHTTP

それでは一つずつ見ていきたいと思います。

``` tsx:src/components/CreateFromPrivateKey.tsx
import React, { useState } from 'react'
import {
  Account,
  NetworkType,
  Address, // 追加
  RepositoryFactoryHttp, // 追加
} from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  const [address, setAddress] = useState('')
  const [publicKey, setPublicKey] = useState('')
  console.log('秘密鍵', privateKey)

// ここから追加
  const accountInfo = () => {
    const accountAddress = Address.createFromRawAddress(address)
    console.log("アカウントアドレス",accountAddress)
    const nodeUrl = 'http://ngl-dual-101.testnet.symboldev.network:3000'
    console.log("ノードURL",nodeUrl)
    const repositoryFactory = new RepositoryFactoryHttp(nodeUrl)
    console.log("リポジトリファクトリ",repositoryFactory)
    const accountHttp = repositoryFactory.createAccountRepository()
    console.log("アカウントHttp",accountHttp)
    accountHttp.getAccountInfo(accountAddress).subscribe(
      (accountInfo) => console.log(accountInfo),
      (err) => console.error(err),
    );
  }
// ここまで追加

  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
      NetworkType.TEST_NET
    )
    setAddress(account.address.pretty())
    setPublicKey(account.publicKey)
  }
  return (
    <div>
      <input
        onChange={(e) => setPrivateKey(e.target.value)}
        className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline"
      />
      <br />
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>
      <p>アドレス: {address}</p>
      <p>公開鍵: {publicKey}</p>
      {/* ここを追加 */}
      <button onClick={accountInfo}>アカウント情報を取得する</button>
      {/* ここまでを追加 */}
    </div>
  )
}

export default CreateFromPrivateKey
```

はい、では見ていきます。

まずはアカウントアドレスです。
このアカウントアドレスのコンソールを確認すると

![アカウントアドレスコンソール](/images/react-articles/accountAddress.png)

オブジェクトの型で以下のようなデータになっています。

``` json:console
{
    "address": "TBUKFL3BMEXYBDQYBV5Y7UOWNRM3TDRZ4PNFCZQ",
    "networkType": 152
}
```

このアドレスとネットワーク情報があるオブジェクトが今後必要になるということです。
なのでこの部分はもし知っている場合や固定する場合はわざわざcreateFromRawAddressを呼び出す必要もなくなります。

次にノードURLです。
これはURLをそのまま打ち込んだだけですので特に問題ないかと思います。

![ノードURL](/images/react-articles/nodeURL.png)

次にリポジトリファクトリです
ノードのURLから新しくリポジトリファクトリを作成しています。
中身は僕もそんなによくわかっていませんので、下手に解説しません。
こんな感じのものが必要なんだなと思っておいてください。（僕もわかりません）

``` json:console
{
    "url": "http://ngl-dual-101.testnet.symboldev.network:3000",
    "networkType": {
        "_isScalar": false,
        "source": {
            "_isScalar": false
        }
    },
    "networkProperties": {
        "_isScalar": false,
        "source": {
            "_isScalar": false
        }
    },
    "epochAdjustment": {
        "_isScalar": false,
        "source": {
            "_isScalar": false
        }
    },
    "generationHash": {
        "_isScalar": false
    },
    "nodePublicKey": {
        "_isScalar": false
    },
    "websocketUrl": "http://ngl-dual-101.testnet.symboldev.network:3000/ws",
    "networkCurrencies": {
        "_isScalar": false,
        "source": {
            "_isScalar": false
        }
    }
}
```

そしてアカウントHTTPです。
ここも少々お待ちください。なぜこれを使うのかわかっていません。
とにかく今わかっているのは、これが必要なのでコピペしているということです。

``` json:console
{
    "url": "http://ngl-dual-101.testnet.symboldev.network:3000",
    "accountRoutesApi": {
        "configuration": {
            "configuration": {
                "basePath": "http://ngl-dual-101.testnet.symboldev.network:3000"
            }
        },
        "middleware": []
    }
}
```

さてアカウント情報を取得するというボタンを押した方は
以下の情報が取得できているはずです。
この中に保有モザイクとインポータンスが存在します。

``` json:console
{
    "version": 1,
    "recordId": "6179331BD42C4799D0EBB887",
    "address": {
        "address": "TBA6Z5KQ772LGYDJ2RC72PPV4HHBLPDKNQHWL5A",
        "networkType": 152
    },
    "addressHeight": {
        "lower": 506108,
        "higher": 0
    },
    "publicKey": "595D63553130DD117EFA7583F6735FBF6C7A9020BBFCFBDFEC20163BA2D479CA",
    "publicKeyHeight": {
        "lower": 506110,
        "higher": 0
    },
    "accountType": 0,
    "supplementalPublicKeys": {},
    "activityBucket": [],
    // モザイク
    "mosaics": [
        {
            "id": {
                "id": {
                    "lower": 94036284,
                    "higher": 153060222
                }
            },
            "amount": {
                "lower": 47938864,
                "higher": 0
            }
        }
    ],
    // インポータンス
    "importance": {
        "lower": 0,
        "higher": 0
    },
    "importanceHeight": {
        "lower": 0,
        "higher": 0
    }
}
```

さて保有モザイクとインポータンスの居場所もわかったのであとはそれを表示しましょう。

:::message
モザイクや残高の情報などは扱う数字の桁数が多く

lower: ある桁数から下
higher: ある桁数から上

のように区切って取得しています。
なのであとでくっつけないといけないです。
:::

``` tsx:src/components/CreateFromPrivateKey.tsx
import React, { useState } from 'react'
import {
  Account,
  NetworkType,
  Address,
  RepositoryFactoryHttp,
  Mosaic,
} from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  const [address, setAddress] = useState('')
  const [publicKey, setPublicKey] = useState('')
  const [mosaics, setMosaics] = useState<Mosaic[]>([])
  const [importance, setImportance] = useState({
    lower: 0,
    higher: 0,
  })
  console.log(mosaics)

  const accountInfo = () => {
    const accountAddress = Address.createFromRawAddress(address)
    const nodeUrl = 'http://ngl-dual-101.testnet.symboldev.network:3000'
    const repositoryFactory = new RepositoryFactoryHttp(nodeUrl)
    const accountHttp = repositoryFactory.createAccountRepository()
    accountHttp.getAccountInfo(accountAddress).subscribe(
      (accountInfo) => {
        console.log(accountInfo)
        setMosaics(accountInfo.mosaics)
        setImportance(accountInfo.importance)
      },
      (err) => console.error(err)
    )
  }

  const accountCreateFromPrivateKey = () => {
    const account = Account.createFromPrivateKey(
      privateKey,
      NetworkType.TEST_NET
    )
    setAddress(account.address.pretty())
    setPublicKey(account.publicKey)
  }
  return (
    <div>
      <input
        onChange={(e) => setPrivateKey(e.target.value)}
        className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline"
      />
      <br />
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>
      <p>アドレス: {address}</p>
      <p>公開鍵: {publicKey}</p>
      <button onClick={accountInfo}>アカウント情報を取得する</button>
      {mosaics[0] && importance && (
        <>
          <p>モザイク総量{mosaics[0].amount.higher}</p>
          <p>モザイク総量{mosaics[0].amount.lower}</p>
          <p>モザイクID{mosaics[0].id.id.lower}</p>
          <p>モザイクID{mosaics[0].id.id.higher}</p>
          <p>インポータンス{importance.lower}</p>
          <p>インポータンス{importance.higher}</p>
        </>
      )}
    </div>
  )
}

export default CreateFromPrivateKey

```

:::message
モザイクが複数ある場合はモザイクのIDを識別子(key)
として実施してみてください。
これでうまくいくはずです。
forを実施するときにkeyが一意（それぞれ違うものでないといけない）
というルールがあるらしく、そのルールに従う感じです。
:::

``` tsx:src/components/CreateFromPrivateKey.tsx
  const mosaicList = () => {
    const items = []
    for (let i = 0; i < mosaics.length; i++) {
      items.push(
        <li key={mosaics[i].id.id.higher}>
          モザイク総量{mosaics[i].amount.higher}<br/>
          モザイク総量{mosaics[i].amount.lower}<br/>
          モザイクID{mosaics[i].id.id.lower}<br/>
          モザイクID{mosaics[i].id.id.higher}
        </li>
      )
    }
    return <ul>{items}</ul>
  }

  {mosaics && importance && (
        <>
          {mosaicList()}
          <p>インポータンス{importance.lower}</p>
          <p>インポータンス{importance.higher}</p>
        </>
      )}
```

ちょっとここの解説が頼りないので

リポジトリファクトリはSymbolのAPIに繋ぐためのクライアントを作るためのクラスです。
コンストラクタ引数にAPIのエンドポイントの設定するので、これを使うことでAPIに接続するための設定を共通化できます

[リポジトリファクトリ](https://docs.symbolplatform.com/symbol-sdk-typescript-javascript/0.18.1/classes/_infrastructure_repositoryfactoryhttp_.repositoryfactoryhttp.html)

AccountHttpはアカウントの情報を取得するエンドポイントに接続するためのクライアントです

[accountHttp](https://docs.symbolplatform.com/symbol-sdk-typescript-javascript/0.18.1/classes/_infrastructure_accounthttp_.accounthttp.html)

この辺りはREST APIの知識にも繋がってくるので一度学習されることをお勧めします。

[RESTAPI](https://www.redhat.com/ja/topics/api/what-is-a-rest-api)

:::message
ですが今は、こう言う風に書いたら動いた!!
なぜだかわからないけど、こんな感じで書いたら動いた!!
と言う体験を重要視しているので、そんなに気にしなくていいです。
:::

## 必要な情報をまとめる

lowerやhigherなどが出てきたのでその情報を加工できるようにしましょう。

モザイクはtoHex()でまとめる
インポータンスはstringで表示する
モザイクの総量はstringで表示する

``` tsx:src/components/CreateFromPrivateKey.tsx
const mosaicList = () => {
  const items = []
  for (let i = 0; i < mosaics.length; i++) {
    items.push(
      <li key={mosaics[i].id.id.lower}>
        モザイクID: {mosaics[i].id.id.toHex()}------モザイクの総量: {mosaics[i].amount.toString()}
      </li>
    )
  }
  return <ul>{items}</ul>
}

// -----中略--------

{mosaics && importance && (
  <>
    {mosaicList()}
    <p>インポータンス: {importance.toString()}</p>
  </>
)}
```

useStateを使えるようにする


``` tsx:src/components/CreateFromPrivateKey.tsx
import React, { useState, useEffect, useCallback } from 'react'

  const accountInfo = useCallback(() => {
    const accountAddress = Address.createFromRawAddress(address)
    const nodeUrl = 'http://ngl-dual-101.testnet.symboldev.network:3000'
    const repositoryFactory = new RepositoryFactoryHttp(nodeUrl)
    const accountHttp = repositoryFactory.createAccountRepository()
    accountHttp.getAccountInfo(accountAddress).subscribe(
      (accountInfo) => {
        console.log(accountInfo)
        setPublicKey(accountInfo.publicKey)
        setMosaics(accountInfo.mosaics)
        setImportance(accountInfo.importance)
      },
      (err) => console.error(err)
    )
  }, [address])

  useEffect(() => {
    accountInfo()
  }, [address, accountInfo])
```

使い方としてはある変数が変更になったときに動きましょう！！
といった感じです。

useEffectの中でaccountInfoを呼び出しているのでaccountInfoをuseCallbackで
囲ってあげます。

そうすると警告は消えます。

:::message
ちなみにuseCallbackを使用しなくても想定通りの動きはしますが、
warningは基本的に解消できますので積極的に実施していきましょう！！
:::

ここまでのソースコードになります

:::message alert
addressの初期値を空白にするとtypescript側から警告が出ます。
なのでaddressの初期値には実際に有効なsymbolウォレットのアドレスを設定しています。
これはsymbolの特徴というよりtypescriptの特徴です。
:::

``` tsx:src/components/CreateFromPrivateKey.tsx
import React, { useState, useEffect, useCallback } from 'react'
import {
  Account,
  NetworkType,
  Address,
  RepositoryFactoryHttp,
  Mosaic,
} from 'symbol-sdk'

const CreateFromPrivateKey = () => {
  const [privateKey, setPrivateKey] = useState('')
  const [address, setAddress] = useState(
    'TCUKQQFP6XTIIA2WLHUUGHVFPE62OIMGWUP7SHY' // ここに初期アドレスを設定しておいてください。
  )
  const [publicKey, setPublicKey] = useState('')
  const [mosaics, setMosaics] = useState<Mosaic[]>([])
  const [importance, setImportance] = useState({
    lower: 0,
    higher: 0,
  })



  const mosaicList = () => {
    const items = []
    for (let i = 0; i < mosaics.length; i++) {
      items.push(
        <li key={mosaics[i].id.id.lower}>
          モザイクID: {mosaics[i].id.id.toHex()}------モザイクの総量: {mosaics[i].amount.toString()}
        </li>
      )
    }
    return <ul>{items}</ul>
  }

  const accountInfo = useCallback(() => {
    const accountAddress = Address.createFromRawAddress(address)
    // ちょっとノードをhttpsに対応しているものにしています。
    const nodeUrl = 'https://sym-test.opening-line.jp:3001/'
    const repositoryFactory = new RepositoryFactoryHttp(nodeUrl)
    const accountHttp = repositoryFactory.createAccountRepository()
    accountHttp.getAccountInfo(accountAddress).subscribe(
      (accountInfo) => {
        console.log(accountInfo)
        setPublicKey(accountInfo.publicKey)
        setMosaics(accountInfo.mosaics)
        setImportance(accountInfo.importance)
      },
      (err) => console.error(err)
    )
  }, [address])

  useEffect(() => {
    accountInfo()
  }, [address, accountInfo])

  function accountCreateFromPrivateKey() {
    const account = Account.createFromPrivateKey(
      privateKey,
      NetworkType.TEST_NET
    )
    setAddress(account.address.pretty())
    setPublicKey(account.publicKey)
  }
  return (
    <div>
      <input
        onChange={(e) => setPrivateKey(e.target.value)}
        className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline"
      />
      <br />
      <button onClick={accountCreateFromPrivateKey}>
        秘密鍵からアカウントを作成する
      </button>
      <p>アドレス: {address}</p>
      <p>公開鍵: {publicKey}</p>
      {mosaics && importance && (
        <>
          {mosaicList()}
          <p>インポータンス: {importance.toString()}</p>
        </>
      )}
    </div>
  )
}

export default CreateFromPrivateKey

```

これで秘密鍵を入力すると
アカウントが作成されて色々情報も自動的に更新されるようになりました。

またuseEffectやuseCallbackなどは結構解説が大変なのですが、
Udemyなどでいい教材たくさんあるのでセール期間中に購入して実施されることをお勧めします。

[進捗](https://docs.google.com/spreadsheets/d/1-WTAIUGgQmJ34JLCK3tkPtZqv57FRAL9hrGggieKvig/edit?usp=sharing)

[ソースコード](https://github.com/nemtus/symbol-sample-react/tree/account_have_mosaic)

## デザイン編

さて、ここまでは機能の確認をしました。
次はデザインを確認していきます。

[こちらで確認したように](https://docs.google.com/presentation/d/1ayIrqPLpvPRRpsTsl8JnDY632EaCRklmF8PQZXjsYLk/edit?usp=sharing)

一つずつ作っていきます。

[タスクはこちら](https://docs.google.com/spreadsheets/d/1-WTAIUGgQmJ34JLCK3tkPtZqv57FRAL9hrGggieKvig/edit?usp=sharing)

一つずつ作っていきましょう！！

### デザイン

TailwindCSSでの心構え。

1. 上手い人のを参考にする（初めは丸パクリでもOK)
2. cssを書くのではなくclassNameで設定する
3. Heroiconsはなぜかセットで使われることが多いからその構成も真似してしまえ！！

です。

これは僕は誰かに教わったと言うより、こう言う風にしています。

正直なところ今までやってきた活動でデザインを求められることは多いですが、
まずは機能検証をしたい。

とにかくSymbolを使ってみたい
といった方も多いかと思います。

下手しい最初はソースコードコピペでいいんかなとは思います。

今はできないけど、色々と作っていくとちょっとずつできるようになってくるのではないかなぁと思っています。

:::message alert
こう言う話をすると著作権意識が低いといってくる方はいますが、
正味な話、著作権を意識してできるのは高いレベルを持っているからであって
最初にアプリを作りたいと思うところからスタートする方に著作権を考えるのは酷な話です。
なので使いたい時はメールで使ってもいいですか？と問い合わせて、返事がもらえれば使いましょう！！
案外フリーでデザインできるコンポーネントはいっぱいあります。
:::

[僕はここをよく使っています](https://tailwindcomponents.com/)

:::message
ひとつずつ成長しましょう
誰かのマウントを相手している時間はあなたにはありません！！
:::

[アイコン](https://heroicons.com/)

[サイドバーの参考](https://tailwindcomponents.com/component/jed-dylan-lee)

[ここまでのソースコード](https://github.com/nemtus/symbol-sample-react/tree/header)

:::details ホームコンポーネントの実装

htmlタグでしか記載していませんので、特に解説は致しませんが、
重要なポイントとしてアイコンを使用するときは
heroiconsのjsxをコピーしてくるだけでOKといった感じです。

``` tsx:アイコン例
<svg
  xmlns="http://www.w3.org/2000/svg"
  className="h-6 w-6"
  fill="none"
  viewBox="0 0 24 24"
  stroke="currentColor"
>
  <path
    strokeLinecap="round"
    strokeLinejoin="round"
    strokeWidth={2}
    d="M20.618 5.984A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016zM12 9v2m0 4h.01"
  />
</svg>
```

``` tsx:src/component/Home.tsx
import React from 'react'
import Github from '../images/logo-github.png'
import Symbol from '../images/logo-symbol-color.png'
import NumTus from '../images/logo-nemtus-color.png'
import Twitter from '../images/logo-twitter-color.png'

const Home = () => {
  return (
    <>
      <div className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline">
        Home
      </div>
      <div className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline">
        <div className="bg-grey-light hover:bg-grey text-grey-darkest font-bold py-2 px-2 rounded inline-flex items-center">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            className="h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth={2}
              d="M20.618 5.984A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016zM12 9v2m0 4h.01"
            />
          </svg>
          <span className="ml-6">Note</span>
        </div>

        <p>
          ・ブロックチェーンSymbolとSPAフレームワークReactを用いて作成しているWebアプリケーションのサンプルです。
        </p>
      </div>
      <div className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline">
        <div className="bg-grey-light hover:bg-grey text-grey-darkest font-bold py-2 px-2 rounded inline-flex items-center">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            className="h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth={2}
              d="M20.618 5.984A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016zM12 9v2m0 4h.01"
            />
          </svg>
          <span className="ml-6">Disclaimer</span>
        </div>
        <p>
          ・本サイトのご利用に際しては以下の点にご注意ください。
          <br />
          ・本サイトではSymbolを用いたWebアプリケーションの実装例を示すことに主眼をおいています。
          <br />
          ・本サイトでの実装は十分にテストされていない可能性があることを踏まえ、本サイトのご利用は、実装例を参考にしたり少額のXYMでの動作確認にとどめ、多額のXYMやクリティカルな情報を扱う操作を実行なさならいことを推奨します。
          <br />
          ・本サイトのご利用を通じて生じ得るいかなる問題に対してもNEMTUSとして一切の責任を取ることはできませんのでご注意ください。
          <br />
        </p>
      </div>
      <div className="shadow rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline">
        <div className="bg-grey-light hover:bg-grey text-grey-darkest font-bold py-2 px-2 rounded inline-flex items-center">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            className="h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth={2}
              d="M20.618 5.984A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016zM12 9v2m0 4h.01"
            />
          </svg>
          <span className="ml-6">Reference</span>
        </div>
        <a
          href="https://github.com/nemtus/symbol-sample-react"
          target="_blank"
          rel="noopener noreferrer"
        >
          <button className="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 mt-2 rounded w-full flex flex-grow items-center px-3">
            <img src={Github} alt="github" />
            <span className="ml-6">リポジトリ</span>
          </button>
        </a>
        <a
          href="https://docs.symbolplatform.com/ja/"
          target="_blank"
          rel="noopener noreferrer"
        >
          <button className="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded w-full flex flex-grow items-center px-3 mt-2">
            <img src={Symbol} alt="github" />
            <span className="ml-6">Symbolドキュメント</span>
          </button>
        </a>
        <a
          href="https://nemtus.com/"
          target="_blank"
          rel="noopener noreferrer"
        >
          <button className="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded w-full flex flex-grow items-center px-3 mt-2">
            <img src={NumTus} alt="github" className="w-6" />
            <span className="ml-6">nemtus</span>
          </button>
        </a>
        <a
          href="https://twitter.com/NemtusOfficial"
          target="_blank"
          rel="noopener noreferrer"
        >
          <button className="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded w-full flex flex-grow items-center px-3 mt-2">
            <img src={Twitter} alt="github" />
            <span className="ml-6">Twitter</span>
          </button>
        </a>
      </div>
    </>
  )
}

export default Home

```

:::

:::details サイドバーの実装

ここで重要なポイントはアカウントページとホームページをどう切り換えるかです。
useStateでisHomeという変数を持たせてそれがtrueならこのページを表示
falseならこのページを表示みたいな切り分けをしています。
今回はページルーターを実装するのが手間だったのと、表示するコンポーネントを切り替えるだけ
だったのでこのような実装にしています。

まずは動かしたい内容を決めてどのようにするかがポイントです。
ページの内容を変更したい場合はルータだけでなくこういう方法もあるということです。


``` tsx
const [isHome, setIsHome] = useState(false)

// trueの場合
{isHome && <CreateFromPrivateKey></CreateFromPrivateKey>}
// falseの場合
{!isHome && <Home></Home>}
```

サイドバーの非表示、表示の切り替えも同様です。
isClosedをfalseの状態で非表示
trueの状態で表示となっています。

``` tsx
const [isClosed, setClosed] = useState(false)

{!isClosed && (
<aside
  aria-hidden={isClosed}
  className="bg-gray-800 w-64 min-h-screen flex flex-col text-white"
>
  <div className="border-r border-b px-4 h-10 flex items-center justify-between">
    <span className="text-blue py-2">Symbol-React</span>
  </div>

  <div className="border-r py-4 flex-grow relative">
    <nav>
      <ul className="mt-12">
        <li className="flex w-full justify-between text-white hover:text-gray-500 cursor-pointer items-center mb-6 m-4">
          <button className="flex items-center" onClick={() => {setIsHome(false)}}>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              className="h-6 w-6"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth={2}
                d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"
              />
            </svg>
            <span className="text-sm  ml-2">Home</span>
          </button>
        </li>
        <li className="flex w-full justify-between text-white hover:text-gray-500 cursor-pointer items-center  m-4">
          <button className="flex items-center" onClick={() => {setIsHome(true)}}>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              className="h-6 w-6"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth={2}
                d="M5.121 17.804A13.937 13.937 0 0112 16c2.5 0 4.847.655 6.879 1.804M15 10a3 3 0 11-6 0 3 3 0 016 0zm6 2a9 9 0 11-18 0 9 9 0 0118 0z"
              />
            </svg>
            <span className="text-sm  ml-2">Account</span>
          </button>
        </li>
      </ul>
    </nav>
  </div>
</aside>
)}
```

ヘッダーの部分のボタンの切り替えなどは三項演算子を使用しています。
Reactを触っていたり、上手い人の実装を真似しているとこの実装が多いので参考にしています

``` tsx
<header className="bg-white border-b h-10 flex items-center justify-center">
  {isClosed ? (
    <button
      tabIndex={1}
      className="w-10 p-1"
      aria-label="Open menu"
      title="Open menu"
      onClick={() => setClosed(false)}
    >
      <svg
        aria-hidden="true"
        fill="none"
        strokeLinecap="round"
        strokeLinejoin="round"
        strokeWidth="2"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path d="M4 6h16M4 12h16M4 18h16"></path>
      </svg>
    </button>
  ) : (
    <button
      tabIndex={1}
      className="w-10 p-1"
      aria-label="Close menu"
      title="Close menu"
      onClick={() => setClosed(true)}
    >
      <svg
        fill="none"
        strokeLinecap="round"
        strokeLinejoin="round"
        strokeWidth="2"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path d="M6 18L18 6M6 6l12 12"></path>
      </svg>
    </button>
  )}

  <div className="flex flex-grow items-center px-3">
    <h1 className="text-lg col-start-1 col-end-3">{!isHome ? "Home" : "Account"}</h1>
    <div className="flex-grow"></div>
    <a
      href="https://docs.symbolplatform.com/ja/"
      target="_blank"
      rel="noopener noreferrer"
    >
      <img src={Symbol} alt="symbol" className="col-end-7 col-span-2" />
    </a>
    <a
      href="https://nemtus.com/"
      target="_blank"
      rel="noopener noreferrer"
    >
      <img src={NemTus} alt="nemtus" className="w-6 m-2" />
    </a>
  </div>
</header>
```

以下全文です。

``` tsx:src/component/SideBar.tsx
import React, { useState } from 'react'
import CreateFromPrivateKey from './CreateFromPrivateKey'
import Symbol from '../images/logo-symbol-color.png'
import NemTus from '../images/logo-nemtus-color.png'
import Home from './Home'

const SideBar = () => {
  const [isClosed, setClosed] = useState(false)
  const [isHome, setIsHome] = useState(false)
  return (
    <div className="flex bg-gray-100 w-full">
      {!isClosed && (
        <aside
          aria-hidden={isClosed}
          className="bg-gray-800 w-64 min-h-screen flex flex-col text-white"
        >
          <div className="border-r border-b px-4 h-10 flex items-center justify-between">
            <span className="text-blue py-2">Symbol-React</span>
          </div>

          <div className="border-r py-4 flex-grow relative">
            <nav>
              <ul className="mt-12">
                <li className="flex w-full justify-between text-white hover:text-gray-500 cursor-pointer items-center mb-6 m-4">
                  <button className="flex items-center" onClick={() => {setIsHome(false)}}>
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      className="h-6 w-6"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        strokeLinecap="round"
                        strokeLinejoin="round"
                        strokeWidth={2}
                        d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"
                      />
                    </svg>
                    <span className="text-sm  ml-2">Home</span>
                  </button>
                </li>
                <li className="flex w-full justify-between text-white hover:text-gray-500 cursor-pointer items-center  m-4">
                  <button className="flex items-center" onClick={() => {setIsHome(true)}}>
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      className="h-6 w-6"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        strokeLinecap="round"
                        strokeLinejoin="round"
                        strokeWidth={2}
                        d="M5.121 17.804A13.937 13.937 0 0112 16c2.5 0 4.847.655 6.879 1.804M15 10a3 3 0 11-6 0 3 3 0 016 0zm6 2a9 9 0 11-18 0 9 9 0 0118 0z"
                      />
                    </svg>
                    <span className="text-sm  ml-2">Account</span>
                  </button>
                </li>
              </ul>
            </nav>
          </div>
        </aside>
      )}

      <main className="flex-grow flex flex-col min-h-screen w-full">
        <header className="bg-white border-b h-10 flex items-center justify-center">
          {isClosed ? (
            <button
              tabIndex={1}
              className="w-10 p-1"
              aria-label="Open menu"
              title="Open menu"
              onClick={() => setClosed(false)}
            >
              <svg
                aria-hidden="true"
                fill="none"
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth="2"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path d="M4 6h16M4 12h16M4 18h16"></path>
              </svg>
            </button>
          ) : (
            <button
              tabIndex={1}
              className="w-10 p-1"
              aria-label="Close menu"
              title="Close menu"
              onClick={() => setClosed(true)}
            >
              <svg
                fill="none"
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth="2"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          )}

          <div className="flex flex-grow items-center px-3">
            <h1 className="text-lg col-start-1 col-end-3">{!isHome ? "Home" : "Account"}</h1>
            <div className="flex-grow"></div>
            <a
              href="https://docs.symbolplatform.com/ja/"
              target="_blank"
              rel="noopener noreferrer"
            >
              <img src={Symbol} alt="symbol" className="col-end-7 col-span-2" />
            </a>
            <a
              href="https://nemtus.com/"
              target="_blank"
              rel="noopener noreferrer"
            >
              <img src={NemTus} alt="nemtus" className="w-6 m-2" />
            </a>
          </div>
        </header>
        {isHome && <CreateFromPrivateKey></CreateFromPrivateKey>}
        {!isHome && <Home></Home>}
      </main>
    </div>
  )
}

export default SideBar
```

実装イメージとして
以下のイメージ図を参考にしていただけますと幸いです。

![こんな感じのコンポーネントをイメージしています](/images/react-articles/component.png)

:::

## まとめ

お疲れ様でした。

色々いっぱい解説したので、まずはポイントを

1. 環境構築は面倒なので休憩しながらやりましょう。（慣れるまでの辛抱です）
2. 実際にアプリケーションを作るとSymbolの実装は結構少ないです（だからなんか寂しいんです）
3. どちらかというとReactやTailwindCSSの方が面倒です（慣れるまでの辛抱です）
4. 本人のペースを崩さす勉強していってください（他人のマウントを相手している時間はあなたにはありません。）

となります。
いや〜疲れましたね。ここまでやってちょっとわかった！！となってもらえれば嬉しいです。

結局のところReactやTypescriptの解説が多かった気がしますが、
要するにデータはsymbol-sdkで取得して、表示はReactでといったところです。

Reactの解説が多かったのはsymbol-sdkの解説がそこまで不要なほどに作りがいいと思っています。（なのでちょっと寂しいのですが）

解説量が多い=いい解説

ではなく、

この記事を読んでアプリを作ってみる人が増える=いい解説

と思っていますので、ぜひとも何かしら作ってみるチャレンジをしていただけますと幸いです。

最初はコピペでもいいんです。とにかく楽しんでもらえると幸いです。

疲れた時は、音楽でも聞いてリラックスしましょう。

[ラビィッ！！](https://www.youtube.com/watch?v=RiO8a9ErCBg)

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
- 略歴
  - 障がい者向けアプリケーションを作ろうとして、頑張った結果ブロックチェーンの企業で働いています。
- SNS
  - twitter: [https://twitter.com/salaryman_tousi](https://twitter.com/kazumasamatsumo)
