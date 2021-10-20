---
title: "Ionic(Vue)でSymbolモバイルアプリを作ってみる その1"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blockchain", "symbol", "ionic", "typescript", "vue"]
published: false
---

# Ionic(vue)でSymbolモバイルアプリを作ってみる その1

## 概要

このシリーズでは、Ionic(Vue)を使ってモバイルアプリでSymbolブロックチェーンにアクセスする方法を紹介します。
現在、SymbolのライブラリはJavaScript/TypeScript版、Java版、Python版がリリースされている一方、Swift版やDart版が出ていないため、2021年10月現在でiOSやAndroidのモバイルアプリを作ろうとした場合、Ionic等で所謂ガワアプリを作って対応することになります。
この記事ではIonic(Vue)の導入からSymbol-sdk(JavaScript/TypeScript版)を導入し、転送トランザクションを送信するところまでを紹介します。

この記事のソースコードは[こちら](https://github.com/nemtus/symbol-sample-ionic-vue/tree/main/chapter1)を参照してください。

## Ionicとは

IonicはAngularやReact、Vueといったフレームワークと併用することでモバイルアプリやデスクトップアプリを作ることができるオープンソースのUIツールキットです。
公式ドキュメント: https://ionicframework.jp/docs/


## この記事のサンプルコードの動作環境

- Ionic CLI : 6.17.1
- Ionic Framework : @ionic/vue 5.8.3
- NodeJS : v14.16.1 (/usr/local/bin/node)
- npm    : 7.24.1
- OS     : macOS Big Sur (11.4)


## Ionic CLIの導入

以下のコマンドを実行して、グローバルにIonic CLIを導入します

```sh
$ npm install -g @ionic/cli@latest
```

## プロジェクトの作成

以下のコマンドを実行して、Ionicのプロジェクトを作成します。

```sh
$ ionic start ionocVueSymbol01 blank --type vue 
```

プロジェクトを作成したらプロジェクトのディレクトリに移動します

```sh
$ cd ionocVueSymbol01
```

以下のコマンドでプロジェクトを実行することができます

```sh
$ ionic serve
```

ブラウザで `http://localhost:8100/` にアクセスするとプロジェクトのページが表されます。

## symbol-sdkの導入

プロジェクトのルートディレクトリで以下のコマンドを実行して、symbol-sdkを導入しましょう

```
$ npm install symbol-sdk rxjs
```

## トランザクションを送信してみる

実際にsymbol-sdkを使ってトランザクションを送信してみたいと思います。
まずは、プロジェクトを作った時に作成された `src/view/Home.vue` を以下のように書き換えてみましょう。

```typescript:src/view/Home.vue
<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Send Transaction</ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Send Transaction</ion-title>
        </ion-toolbar>
      </ion-header>
    
      <div id="container">
        <ion-button @click="onClickSendTransaction">Send Transaction</ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonButton,
  loadingController,
  toastController
} from '@ionic/vue';
import { defineComponent } from 'vue';
import { sendTransaction } from '@/libs/SymbolService';
  
export default defineComponent({
  name: 'Home',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
  },

  data() {
    return {
      loading: {} as HTMLIonLoadingElement,
    }
  },

  methods: {
    async showloading() {
      console.log();
      this.loading = await loadingController.create({
        message: '送信中...'
      });
      await this.loading.present();
    },

    async dissmissLoading() {
      this.loading.dismiss();
    },

    async showToast(msg: string) {
      const toast = await toastController.create({
        message: msg,
        duration: 2000,
      });
      
      toast.present();
    },

    async onClickSendTransaction() {
      try {
        await this.showloading();
        const transaction = await sendTransaction();
        console.log(transaction);
        await this.showToast('送信が完了しました');
      } catch (err) {
        console.error(err);
        await this.showToast(err as string);
      } finally {
        await this.dissmissLoading();
      }
    }
  }
});
</script>

<style scoped>
<!-- 中略(変更なし) -->
</style>
```

これで、画面の中央に `Send Transaction` というボタンが表示され、これをクリックすると `onClickSendTransaction()` というメソッドがコールされ、トランザクションを送信する処理が実行されます。
トランザクションの処理を待っている間はインジケータが回り、終わるかエラーになるとその結果がトーストで表示されるようになっています。
実際のトランザクションの送信処理は `sendTransaction()` を呼んで実行しています。
今度は、`src/libs/SymbolService.ts` を作成し、`sendTransaction()` の処理を書いていきましょう。

`SymbolService.ts` は下記の通りとなります。

```typescript:SymbolService.ts
import {
  Account,
  Address,
  Deadline,
  NetworkType,
  PlainMessage,
  RepositoryFactoryHttp,
  TransactionService,
  TransferTransaction
} from 'symbol-sdk';


const nodeUrl = 'https://sym-test.opening-line.jp:3001';
const networkType = NetworkType.TEST_NET;
const networkGenerationHash = '3B5E1FA6445653C971A50687E75E6D09FB30481055E3990C84B25E9222DC1155'
const epochAdjustment = 1616694977;

const repoFactory = new RepositoryFactoryHttp(nodeUrl, {
  websocketUrl: `${nodeUrl.replace('http', 'ws')}/ws`,
  websocketInjected: WebSocket,
});

const senderPrivateKey = process.env.VUE_APP_SENDER_PRIVATE_KEY;
const senderAccount = Account.createFromPrivateKey(senderPrivateKey, networkType);

// faucet address
const targetAddress = 'TCLQ3QKUFV6I35FVDXVMB7X4CWI3FLAOVQGNKCQ';

export async function sendTransaction() {
  const transferTransaction = TransferTransaction.create(
    Deadline.create(epochAdjustment),
    Address.createFromRawAddress(targetAddress),
    [NetworkCurrencies.PUBLIC.currency.createRelative(1)],
    PlainMessage.create('hello ionic vue'),
    networkType,
  ).setMaxFee(100);

  const signedTransaction = senderAccount.sign(transferTransaction, networkGenerationHash);

  const transactionRepo = repoFactory.createTransactionRepository();
  const receiptRepo = repoFactory.createReceiptRepository();
  const transactionService = new TransactionService(transactionRepo, receiptRepo);
  const listener = repoFactory.createListener();
  await listener.open();
  try {
    const transaction = await transactionService.announce(signedTransaction, listener).toPromise();
    return transaction;
  } catch(err) {
    console.error(err);
    throw(err);
  } finally {
    listener.close();
  }
}
```

`SymbolService.ts` にトランザクションの送信処理をまとめていますが、この中身を細かく見ていこうと思います。


```typescript
const nodeUrl = 'https://sym-test.opening-line.jp:3001';
const networkType = NetworkType.TEST_NET;
const networkGenerationHash = '3B5E1FA6445653C971A50687E75E6D09FB30481055E3990C84B25E9222DC1155'
const epochAdjustment = 1616694977;
```

まず、上記の最初の4行で接続するネットワークの基本情報を設定しています。
`nodeUrl` は接続するSymbolのRESTゲートウェイを指定しています。ここではOpening Line社のテストネットのノードに接続していますが、Symbolではメインネットで1000台以上、テストネットで50台程度のノードが動いており、これらノードは協調して動いておりどのノードにつないでも構いません。
`networkType` はここでは `NetworkType.TEST_NET` を指定しています。メインネットの場合は `NetworkType.MAIN_NET` を指定します。
`networkGenerationHash` はネットワーク固有の値でリプレイ攻撃を防ぐため、トランザクションの署名をする際にこの値が必要となってきます。
Symbolではネットワーク毎の基準時間が定めたれており、UNIX時間との差分を `epochAdjustment` で保持しています。この値は、トランザクションの有効期限を設定する際に必要となります。

networkGenerationHashやepochAdjustmentは接続先のノードの `/network/properties` で確認することができます。

```typescript
const repoFactory = new RepositoryFactoryHttp(nodeUrl, {
  websocketUrl: `${nodeUrl.replace('http', 'ws')}/ws`,
  websocketInjected: WebSocket,
});
```

`RepositoryFactoryHttp` はsymbol-sdkで各REST APIに接続するためのクライアントを作成するためのインスタンスです。
接続先のノードを指定すると共に、WebSocketのための設定をオプションを追加しています。
`websocketUrl` は `ws(s)://(接続先ノード):(3000|3001)/ws` になるように
`websocketInjected` はデフォルトではnodeのものになっているため、ブラウザのものに設定しています。

```typescript
const senderPrivateKey = process.env.VUE_APP_SENDER_PRIVATE_KEY;
const senderAccount = Account.createFromPrivateKey(senderPrivateKey, networkType);
```

トランザクションを送信するアカウントの秘密鍵とそれを基に `Account` クラスを作成しています。
秘密鍵は今回のサンプルでは`.env`に保存していますが、秘密鍵が漏れるとアカウントのコントロールを奪われることになるため、実際の運用の際に慎重に取り扱ってください。

ここから`sendTransaction()` の中身を見ていきたいと思います。

```typescript
const transferTransaction = TransferTransaction.create(
    Deadline.create(epochAdjustment),
    Address.createFromRawAddress(targetAddress),
    [NetworkCurrencies.PUBLIC.currency.createRelative(1)],
    PlainMessage.create('hello ionic vue'),
    networkType,
  ).setMaxFee(100);
```

まず最初に転送トランザクションのインスタンスを作成します。
Symbolには何種類かのトランザクションがありますが、転送トランザクションはその中でもよく使われるトランザクションで、Symbolの基軸通貨のxymなどのモザイクを送信します。
それでは転送トランザクションの中身を見ていきましょう。
`Deadline.create(epochAdjustment)` ではトランザクションの有効期限を設定します。デフォルトではこのトランザクションのインスタンスを作成してから2時間となっています。
`Address.createFromRawAddress(targetAddress)`は送信先のアドレスを指定します。
3つめの引数では送信するモザイクを指定します。引数が配列になっていることからも分かりますが、1回のトランザクションで複数のモザイクを送ることができます。
`NetworkCurrencies.PUBLIC.currency` はxymを指定するコンビニエンスメソッドです。
`NetworkCurrencies.PUBLIC.currency.createRelative(1)`とすることで、1xymを送信すると指定しています。
`PlainMessage.create('hello ionic vue')` は転送トランザクションに添付するメッセージを設定しています。これは平文で`hello ionic vue`というメッセージを送っています。
5つ目の引数ではネットワークのタイプを指定します。
`setMaxFee`はトランザクションの最大手数料を指定しています。
Symbolのパブリックチェーンでは手数料を支払う必要がありますが、その手数料は `トランザクションのバイト数 * トランザクションが取り込まれたブロックの手数料倍率` となっています。
`setMaxFee`は引数で何倍の手数料倍率まで支払うかを設定します。この例では`100倍`まで支払うということになります。

```typescript
const signedTransaction = senderAccount.sign(transferTransaction, networkGenerationHash);
```

トランザクションを作ったら、それをアカウントの秘密鍵とネットワーク固有のハッシュで署名します。
これでネットワークにモザイク転送のトランザクションを送る準備ができました。

```typescript
const transactionRepo = repoFactory.createTransactionRepository();
const receiptRepo = repoFactory.createReceiptRepository();
const transactionService = new TransactionService(transactionRepo, receiptRepo);
const listener = repoFactory.createListener();
```

上記のコードは作成したトランザクションをネットワークにアナウンスするためのRESTAPIとWebSoketに接続するためのクライアントを作成しています。

```typescript
await listener.open();
try {
    const transaction = await transactionService.announce(signedTransaction, listener).toPromise();
    return transaction;
  } catch(err) {
    console.error(err);
    throw(err);
  } finally {
    listener.close();
  }
}
```

上記がトランザクションをネットワークに送信する処理と送信したトランザクションの結果を監視するコードとなっています。
Symbolのトランザクションの処理結果は非同期で返ってくるため、最初にWebSocketに接続します。
`transactionService.announce(signedTransaction, listener)` はトランザクションをネットワークにアナウンスとトランザクションの結果の監視を1まとめにしたコンビニエンスメソッドです。
これ1つでトランザクションがブロックに取り込まれるか何らかのエラーが発生したかを監視することができます。
Symbolのトランザクションは非同期で処理されるため、トランザクションがブロックに取り込まれたか、エラーが発生したかはWebSocketもしくは別なAPIを叩いて確認する必要がありますが、このメソッドはそれをひとまとめにしています。
`transactionService.announce(signedTransaction, listener)` は本来rxjsで処理するのですが、今回の例では簡便化のためpromise化して、async/awaitで処理しています。

トランザクションの結果は、トランザクションがブロックに取り込まれ承認されると返り値としてトランザクションのデータが返ります。Symbolのブロック生成間隔は30秒程度なので承認されるまでそのくらいの時間がかかります。
残高不足なのでトランザクションが受け入れられなかった場合は、catch節に流れ、エラー内容が引数`err`に入ります。
成否にかかわらず最後にWebSoketを閉じます。

以上がトランザクションを送信する基本的な流れとなります。

## ネイティブアプリとしてビルドする

IonicはiOSアプリやAndroidアプリとしてビルドすることが可能です。
ネイティブアプリとしてビルドする場合は以下の様にします。

プロジェクトにcapacitorを追加する

```sh
$ ionic integrations enable capacitor
```

ビルドしてから利用するプラットフォームを選択する

```sh
$ ionic build
$ ionic cap add ios
$ ionic cap add android
```

iOS/Androidのプロジェクトビルド、実行するにはXCodeやAndroid Studioを利用します。

```sh
ionic cap open ios
ionic cap open android
```

## まとめ

今回は、Ionic(Vue.js)を使って、モザイクを転送するトランザクションを送信するWebアプリを作成し、それをネイティブアプリにする流れまでを紹介しました。
Ionicを活用することで比較的容易にSymbolのブロックチェーンを活用したネイティブアプリを作ることができることが伝われば幸いです。
次回はQRコードリーダを組み込み、より活用できるモバイルアプリを作っていければと思います。

## 最後に

もしNEMTUSに対しNEMやSymbol関連記事の寄稿や、サンプルとして公開したアプリについて何かありましたら、以下GitHubにて記事やサンプルアプリを公開しておりますので、お気軽にIssueやPull Request等、連携くださいますと幸いです。

- [https://github.com/nemtus/](https://github.com/nemtus/)

NEMTUSとして、NEM, Symbolに関する様々な技術情報を継続的に発信していきたいと考えていますので、今後ともどうぞよろしくお願いします。

## 記事作成者

- 名前
  - 岡田和也 (Daoka)
- 所属
  - 株式会社Opening Line テクニカルディレクター
    - [https://www.opening-line.co.jp/](https://www.opening-line.co.jp/)
  - NPO法人NEMTUS 理事
    - [https://nemtus.com/](https://nemtus.com/)
- 略歴
  - 株式会社Opening LineでNEM/Symbolを作ったアプリケーションの開発などをおこないつつ、Opening LineやNEMTUSとしてNEM/Symbolの普及活動にも従事。Symbolリリース前のテストでは所謂Daoka砲で不具合を洗い出すなどの貢献した。
- SNS
  - twitter: [https://twitter.com/DaokaTrade](https://twitter.com/DaokaTrade)
  - GitHub: [https://github.com/daoka](https://github.com/daoka)

