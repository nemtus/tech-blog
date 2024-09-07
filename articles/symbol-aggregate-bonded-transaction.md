---
title: "Symbolブロックチェーンでのアグリゲートボンデッドトランザクション送信サンプルコードの紹介"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blockchain", "symbol", "typescript", "javascript", "npm"]
published: true
publication_name: "nemtus_official"
---

## 要約

この記事では、[@nemtus/symbol-sdk-typescript](https://github.com/nemtus/symbol/tree/dev/sdk/javascript)を使って、Symbolブロックチェーンでアグリゲートボンデッドトランザクション(Aggregate Bonded Transaction)を送信するためのサンプルコードを紹介します。

詳細については、記事内のサンプルコードをご参照ください。

## アグリゲートボンデッドトランザクション(Aggregate Bonded Transaction)とは

先日公開した以下記事では、シンプルなトークン送信トランザクション(Transfer Transaction)のサンプルコードを紹介しました。

https://zenn.dev/nemtus/articles/nemtus-symbol-sdk-typescript

Symbolブロックチェーンでは、そのようなシンプルなトークン送信トランザクション以外にも、様々な表現力を持ったトランザクションの種類があり、それらを組み合わせて、トランザクションの送受信のみで様々な挙動をブロックチェーン上でシンプルかつ力強く表現できるという特徴があります。

その組み合わせ方の一つとして、アグリゲートボンデッドトランザクション(Aggregate Bonded Transaction)というトランザクションの種類があります。

これは、複数のトランザクション(上限100個のトランザクション)を、一部、連署に必要な署名数が揃っていない状態でネットワークに送信し、連署人がネットワークに署名を送り、連署が揃ったら実行されるというトランザクションです。連署が揃わない状態のままトランザクションの期限が切れると、トランザクションは実行されず、なかったことになります。

このトランザクションを応用することで、以下のようなことを容易に実現できます。

- 信頼できる第三者を介することなく、取引の当事者のみによるトランザクション送信や署名送信だけでトラストレスに取引を実行
- 複数の取引を一度にまとめることでトランザクションの送信回数や手数料を削減
- 複数の署名が無いとトランザクションが実行されないマルチシグアカウントでの安全な取引

### アグリゲートボンデッドトランザクションとセットで必要になるハッシュロックトランザクション(Hash Lock Transaction)との関係について

アグリゲートボンデッドトランザクションはその自由度の高さから、相手の資産を奪い取るためにトランザクションの署名要求をするような、あるいは正当な理由がない場合は詐欺になるようなトランザクションを送信することもできます。

そういった事態の乱発を防ぐため、送信者に対し、リスクを負わずにはアグリゲートボンデッドトランザクションを送信できないよう、期限までに相手が署名しなかった場合には、予め確保された担保をネットワークの特定のアカウントが没収する仕組みが導入されています。

アグリゲートボンデッドトランザクションを送信するために、このような担保を一時的にネットワークに預けるトランザクションの種類がハッシュロックトランザクションです。

:::message
ハッシュロックトランザクションという名前を聞くと、ブロックチェーンに詳しい方はHTLC(=Hashed Time Lock Contract)を連想される方が多いかもしれませんが、SymbolブロックチェーンでのHTLC的なものはシークレットロックトランザクションとシークレットプルーフトランザクションというトランザクションがあり、SymbolブロックチェーンのハッシュロックトランザクションはHTLCとは異なるものであるということにご注意ください。
:::

対象となるアグリゲートボンデッドトランザクションのトランザクションハッシュをトランザクション送信前に記録しておき、そのトランザクションハッシュと、担保となる10XYMを指定してハッシュロックトランザクションを送信し、そのハッシュロックトランザクションが承認された後に、アグリゲートボンデッドトランザクションを送信するという流れになります。

アグリゲートボンデッドトランザクションがネットワークにアナウンスされると、署名を行う必要がある人のウォレット上に署名要求の通知が届き、そこから署名を行うと、必要な署名が揃ったタイミングでアグリゲートボンデッドトランザクションが承認されます。アグリゲートボンデッドトランザクションが承認されたらハッシュロックトランザクションは自動的にキャンセルされ、担保となっていた10XYMは送信者に戻って来ます。

必要な署名が揃わなかった場合はアグリゲートボンデッドトランザクションは実行されず、ハッシュロックトランザクションの期限が切れると、担保となっていた10XYMはネットワークの特定のアカウントに没収されてしまいます。

## アグリゲートボンデッドトランザクションの具体例

### 割り勘

例えば、Symbolブロックチェーン上でのトークンを使った決済を受け付けているお店での、割り勘的なシチュエーションを考えてみます。

- お店側としては、一度のオペレーションでさっと全員分支払ってほしいはずです。10人など大人数の場合、各個人毎に支払ってもらうようなオペレーションは手間がかかるので避けたいでしょう。
- 一方、団体のお客さん側でも、一人当たり何XYMずつ支払うといった場合には、待ち時間もあるし、誰かが集めてから支払う場合は、その誰かが一時的にお金を預かるため、その人が信頼できる人である必要がある等、お客さんの側でも面倒です。(とても嫌な想像ですが、とりまとめた人が、受け取ったアドレスからトークンの送信がうまくできないと主張し出して、結局誰か別の人が払い、最初にとりまとめた人がそのまま帰って、後日、立て替え分を返さず、バックレるとかのリスクがあるわけです。)

ここでは団体の人数は3人として、3人のお客さんからお店へ一つのアグリゲートボンデッドトランザクションで一度に支払いするサンプルコードを紹介します。

:::message
ここでは、説明しやすいように、単純な具体例を元にアグリゲートボンデッドトランザクションをご説明しますが、実社会では一連の取引を一括実行したいというニーズは数多くあり、様々な応用の可能性があると思います。ぜひ皆様にて実ビジネスでの活用方法を考えてみてください。
:::

登場人物

1. `SIGNER_1` ... 団体のお客さんその1: 店主に対してこの人がアグリゲートボンデッドトランザクションを送る
2. `SIGNER_2` ... 団体のお客さんその2: `SIGNER_1`に公開鍵を事前に伝えている
3. `SIGNER_3` ... 団体のお客さんその3: `SIGNER_1`に公開鍵を事前に伝えている
4. `RECIPIENT` ... 店主

送信するトランザクション概要

- アグリゲートボンデッドトランザクション
  - 内部トランザクション1: `SIGNER_1`から`RECIPIENT`への送金
  - 内部トランザクション2: `SIGNER_2`から`RECIPIENT`への送金
  - 内部トランザクション3: `SIGNER_3`から`RECIPIENT`への送金
- ハッシュロックトランザクション

## アグリゲートボンデッドトランザクションのサンプルコードの紹介

### Setup

前回記事と同様の部分は折り畳み状態にしておきますので、必要に応じて展開してご参照ください。

:::details 前回記事と同様の環境構築手順

#### Node.jsのインストール

この方法は省略します。

#### 適当なディレクトリ(ここでは例として`symbol-sdk-typescript-sample-1`)を作って初期化

```shell
~/$ mkdir symbol-sdk-typescript-sample-1
~/$ cd symbol-sdk-typescript-sample-1
~/symbol-sdk-typescript-sample-1$ npm init -y

```

#### 必要なパッケージをインストールする

ここでは以下のパッケージを使ってトランザクションを送信します。

1. typescript, ts-node ... TypeScriptで色々やるにはtypescriptは必須、ts-nodeは手軽な実行のために使う
2. REST APIでSymbolブロックチェーンの情報を参照したりトランザクションを送信したり等 ... [@nemtus/symbol-sdk-openapi-generator-typescript-axios](https://www.npmjs.com/package/@nemtus/symbol-sdk-openapi-generator-typescript-axios)
3. 今回作成したTypeScript向けSDK(アカウント情報のハンドリング、トランザクションデータ作成、署名等) ... [@nemtus/symbol-sdk-typescript](https://www.npmjs.com/package/@nemtus/symbol-sdk-typescript)
4. WebSocketを使用するため ... [ws](https://www.npmjs.com/package/ws)
5. REST API clientで使用するため ... [axios](https://www.npmjs.com/package/axios)
6. 秘密鍵をソースコードに直接書かずに環境変数的に扱うため ... [dotenv](https://www.npmjs.com/package/dotenv)

以下コマンドでインストールします。

```bash
~/symbol-sdk-typescript-sample-1$ npm install -D typescript ts-node @types/ws
~/symbol-sdk-typescript-sample-1$ npm install @nemtus/symbol-sdk-typescript @nemtus/symbol-sdk-openapi-generator-typescript-axios axios ws dotenv

```

#### `tsconfig.json`ファイルを作成

以下コマンドを実行してTypeScriptでのコードをコンパイルするための設定ファイル`tsconfig.json`の雛型を生成します。

```bash
~/symbol-sdk-typescript-sample-1$ npx tsc --init

```

ES ModulesやBigInt等のJavaScriptとしては新しめの書き方が使えるように、`"target": "es2016"`を`"target": "esnext"`に変更しておきます。

:::

#### テスト用アカウントの作成と環境変数へのセット

Symbolブロックチェーンの公式ウォレットを以下リンクからダウンロードして、テストネットで以下のようにテスト用のアカウントを3個作ってフォーセットからテスト用のトークンを取得しましょう。そして、そのアカウントの秘密鍵、公開鍵、アドレスを記録しておきましょう。

[https://github.com/symbol/desktop-wallet/releases](https://github.com/symbol/desktop-wallet/releases)

記録できたら、以下のように`.env`ファイルに秘密鍵を記入しておきましょう。

```shell:.env
SIGNER_1_PRIVATE_KEY ="PUT_YOUR_PRIVATE_KEY_HERE";
SIGNER_2_PRIVATE_KEY ="PUT_YOUR_PRIVATE_KEY_HERE";
SIGNER_3_PRIVATE_KEY ="PUT_YOUR_PRIVATE_KEY_HERE";

```

:::message alert
ブロックチェーンの世界において、秘密鍵は、そのアカウントの全てを実行できる最強かつ唯一の権限を持っていると言えるでしょう。秘密鍵が漏洩するとそのアカウントのあらゆる資産や秘密情報を失うことになるので、秘密鍵の管理は可能な限り慎重を期してください。

今回はテストネットにて使い捨てのアカウントを作って試すという前提で、dotenvを用いた簡易的な環境変数のような形で秘密鍵を使っていますが、メインネットや本番環境のような実際に価値がある資産と紐づいたネットワーク上でのアカウント管理では、さらに厳重な、適切な方法で、秘密鍵を保護することを強く推奨します。
:::

### SampleCode

これで環境が整いました。以下ファイルを作成し、`npx ts-node send-aggregate-bonded-transaction-to-transfer-from-multi-accounts.ts`で実行してみましょう。

```typescript:send-aggregate-bonded-transaction-to-transfer-from-multi-accounts.ts
import { SymbolFacade } from "@nemtus/symbol-sdk-typescript/esm/facade/SymbolFacade";
import { PrivateKey } from "@nemtus/symbol-sdk-typescript/esm/CryptoTypes";
import { KeyPair } from "@nemtus/symbol-sdk-typescript/esm/symbol/KeyPair";
import {
  Cosignature,
  Signature,
  PublicKey as PublicKeyModel,
} from "@nemtus/symbol-sdk-typescript/esm/symbol/models";
import { hexToUint8 } from "@nemtus/symbol-sdk-typescript/esm/utils/converter";
import {
  Configuration,
  NetworkRoutesApi,
  TransactionGroupEnum,
  TransactionRoutesApi,
  TransactionRoutesApiAnnounceCosignatureTransactionRequest,
  TransactionStatusDTO,
  TransactionStatusRoutesApi,
} from "@nemtus/symbol-sdk-openapi-generator-typescript-axios";
import WebSocket from "ws";
import "dotenv/config";

// テストネットのノードを指定
const NODE_DOMAIN = "symbol-test.next-web-technology.com";

// 送信先アドレス ... 受取者のアドレス: 今回の場合は店主のアドレスとして仮にFaucetアドレスを指定
const recipientAddressString = "TDMYLKCTEVPSRPTG4UXW47IQPCYNLW2OVWZMLGY";

(async () => {
  // epochAdjustment, networkCurrencyMosaicIdの取得のためNetworkRoutesApi.getNetworkPropertiesを呼び出す
  const configurationParameters = {
    basePath: `http://${NODE_DOMAIN}:3000`,
  };
  const configuration = new Configuration(configurationParameters);
  const networkRoutesApi = new NetworkRoutesApi(configuration);
  const networkPropertiesDTO = (await networkRoutesApi.getNetworkProperties())
    .data;

  // epochAdjustmentのレスポンス値は文字列でsが末尾に含まれるため除去してnumberに変換する
  const epochAdjustmentOriginal = networkPropertiesDTO.network.epochAdjustment!;
  const epochAdjustment = parseInt(epochAdjustmentOriginal.replace(/s/g, ""));

  // networkCurrencyMosaicIdのレスポンス値はhex文字列で途中に'が含まれるため除去してBigIntに変換する
  const networkCurrencyMosaicIdOriginal =
    networkPropertiesDTO.chain.currencyMosaicId!;
  const networkCurrencyMosaicId = BigInt(
    networkCurrencyMosaicIdOriginal.replace(/'/g, "")
  );

  // facadeの中に指定するtestnet等のネットワーク名を取得するためNetworkRoutesApi.getNetworkTypeを呼び出す
  const networkTypeDTO = (await networkRoutesApi.getNetworkType()).data!;
  const networkName = networkTypeDTO.name;

  // ネットワーク名を指定してSDKを初期化
  const facade = new SymbolFacade(networkName);

  // アグリゲートボンデッドトランザクションを送信するアカウント関連データを作成(団体のお客さんその1のアカウント: 店主に対してこのアカウントからアグリゲートボンデッドトランザクションを送信する)
  const signer1PrivateKeyString = process.env.SIGNER_1_PRIVATE_KEY!;
  const signer1PrivateKey = new PrivateKey(signer1PrivateKeyString);
  const signer1KeyPair = new KeyPair(signer1PrivateKey);
  const signer1PublicKeyString = signer1KeyPair.publicKey.toString();
  const signer1AddressString = facade.network
    .publicKeyToAddress(signer1KeyPair.publicKey)
    .toString();

  // アグリゲートボンデッドトランザクションの連署するアカウント関連データを作成(団体のお客さんその2のアカウント)
  const signer2PrivateKeyString = process.env.SIGNER_2_PRIVATE_KEY!;
  const signer2PrivateKey = new PrivateKey(signer2PrivateKeyString);
  const signer2KeyPair = new KeyPair(signer2PrivateKey);
  const signer2PublicKeyString = signer2KeyPair.publicKey.toString();
  const signer2AddressString = facade.network
    .publicKeyToAddress(signer2KeyPair.publicKey)
    .toString();

  // アグリゲートボンデッドトランザクションの連署するアカウント関連データを作成(団体のお客さんその3のアカウント)
  const signer3PrivateKeyString = process.env.SIGNER_3_PRIVATE_KEY!;
  const signer3PrivateKey = new PrivateKey(signer3PrivateKeyString);
  const signer3KeyPair = new KeyPair(signer3PrivateKey);
  const signer3PublicKeyString = signer3KeyPair.publicKey.toString();
  const signer3AddressString = facade.network
    .publicKeyToAddress(signer3KeyPair.publicKey)
    .toString();

  // deadlineの計算(2時間で設定しているが変更可能、ただし遠すぎるとエラーになる)
  const now = Date.now();
  const deadline = BigInt(now - epochAdjustment * 1000 + 2 * 60 * 60 * 1000);

  // 内部トランザクション1のデータ生成 ... 団体のお客さんその1から店主への100XYM送金
  const embeddedTransaction1 = facade.transactionFactory.createEmbedded({
    type: "transfer_transaction",
    signerPublicKey: signer1PublicKeyString,
    recipientAddress: recipientAddressString,
    mosaics: [{ mosaicId: networkCurrencyMosaicId, amount: 100000000n }],
  });

  // 内部トランザクション2のデータ生成 ... 団体のお客さんその2から店主への100XYM送金
  const embeddedTransaction2 = facade.transactionFactory.createEmbedded({
    type: "transfer_transaction",
    signerPublicKey: signer2PublicKeyString,
    recipientAddress: recipientAddressString,
    mosaics: [{ mosaicId: networkCurrencyMosaicId, amount: 100000000n }],
  });

  // 内部トランザクション3のデータ生成 ... 団体のお客さんその3から店主への100XYM送金
  const embeddedTransaction3 = facade.transactionFactory.createEmbedded({
    type: "transfer_transaction",
    signerPublicKey: signer3PublicKeyString,
    recipientAddress: recipientAddressString,
    mosaics: [{ mosaicId: networkCurrencyMosaicId, amount: 100000000n }],
  });

  // 内部トランザクションを配列にまとめる
  const embeddedTransactions = [
    embeddedTransaction1,
    embeddedTransaction2,
    embeddedTransaction3,
  ];

  // 内部トランザクションをまとめたハッシュ値を計算
  const embeddedTransactionsHash = (
    facade.constructor as any
  ).hashEmbeddedTransactions(embeddedTransactions);

  // アグリゲートボンデッドトランザクションのデータ生成
  const aggregateBondedTransaction = facade.transactionFactory.create({
    type: "aggregate_bonded_transaction",
    signerPublicKey: signer1PublicKeyString,
    deadline,
    transactionsHash: embeddedTransactionsHash,
    transactions: embeddedTransactions,
  });

  // アグリゲートボンデッドトランザクションの必要署名数を指定して手数料設定 ... 送信先ノードの設定によるがノードのデフォルト設定値100なら基本的に足りないことはないと思う
  const feeMultiplier = 100;
  const signerNumber = 3;
  (aggregateBondedTransaction as any).fee.value = BigInt(
    ((aggregateBondedTransaction as any).size + Signature.SIZE * signerNumber) *
      feeMultiplier
  );

  // アグリゲートボンデッドトランザクションへのSigner1の署名
  const aggregateBondedTransactionSignatureBySigner1 = facade.signTransaction(
    signer1KeyPair,
    aggregateBondedTransaction
  );
  (aggregateBondedTransaction as any).signature = new Signature(
    aggregateBondedTransactionSignatureBySigner1.bytes
  );

  // 各ネットワーク固有のgenerationHashSeedを設定
  (aggregateBondedTransaction as any).network.generationHashSeed =
    facade.network;

  // トランザクションのハッシュを計算 ... 連署者が連署する際や、トランザクションの承認状態を後でWebSocketで確認する時などに必要
  const aggregateBondedTransactionHash = facade.hashTransaction(
    aggregateBondedTransaction
  );
  console.log(aggregateBondedTransactionHash.toString());
  console.log(
    `https://testnet.symbol.fyi/transactions/${aggregateBondedTransactionHash.toString()}`
  ); // デバッグ時に確認しやすいよう、ブロックエクスプローラーの該当ページを表示しておく

  // アグリゲートボンデッドトランザクション送信時にはこのデータを使う
  const aggregateBondedTransactionPayload = (
    facade.transactionFactory.constructor as any
  ).attachSignature(
    aggregateBondedTransaction,
    aggregateBondedTransactionSignatureBySigner1
  );

  // ハッシュロックトランザクションのデータ生成 ... アグリゲートボンデッドトランザクションのハッシュ値を指定してそのトランザクションの実行を必要な署名が揃うまでロックして待機させるトランザクションと解釈するとイメージしやすい
  const hashLockTransaction = facade.transactionFactory.create({
    type: "hash_lock_transaction",
    signerPublicKey: signer1PublicKeyString,
    deadline,
    hash: aggregateBondedTransactionHash.toString(),
    mosaic: { mosaicId: networkCurrencyMosaicId, amount: 10000000n },
    duration: BigInt(((24 * 60 * 60) / 30) * 2), // 2日間に1ブロック30秒の想定で何ブロックかの想定ブロック数
  });

  // ハッシュロックトランザクションの手数料を設定
  (hashLockTransaction as any).fee.value = BigInt(
    (hashLockTransaction as any).size * feeMultiplier
  );

  // ハッシュロックトランザクションの署名
  const hashLockTransactionSignature = facade.signTransaction(
    signer1KeyPair,
    hashLockTransaction
  );
  (hashLockTransaction as any).signature = new Signature(
    hashLockTransactionSignature.bytes
  );
  (hashLockTransaction as any).network.generationHashSeed = facade.network;
  const hashLockTransactionHash = facade.hashTransaction(hashLockTransaction);

  // ハッシュロックトランザクションのペイロードを生成
  const hashLockTransactionPayload = (
    facade.transactionFactory.constructor as any
  ).attachSignature(hashLockTransaction, hashLockTransactionSignature);

  console.log(hashLockTransactionHash.toString());
  console.log(
    `https://testnet.symbol.fyi/transactions/${hashLockTransactionHash.toString()}`
  );

  // 1 confirmation以外の場合の設定
  const confirmationHeight = 6; // 6confで確認と見なす場合
  let hashLockTransactionHeight = 0;
  let aggregateBondedTransactionHeight = 0;
  let blockHeight = 0;
  let finalizedBlockHeight = 0;

  // WebSocketでトランザクション送信時の各種イベントに応じた処理を事前定義しておく必要がある
  const ws = new WebSocket(`wss://${NODE_DOMAIN}:3001/ws`);

  ws.on("open", () => {
    console.log("connection open");
  });

  ws.on("close", () => {
    console.log("connection closed");
  });

  ws.on("message", async (msg: any) => {
    const res = JSON.parse(msg);
    if ("uid" in res) {
      console.log(`uid : ${res.uid}`);

      // SIGNER_1の部分トランザクションが追加されるのを監視
      const partialForSigner1Body = `{"uid": "${res.uid}", "subscribe": "partialAdded/${signer1AddressString}"}`;
      console.log(partialForSigner1Body);
      ws.send(partialForSigner1Body);

      // SIGNER_2の部分トランザクションが追加されるのを監視
      const partialForSigner2Body = `{"uid": "${res.uid}", "subscribe": "partialAdded/${signer2AddressString}"}`;
      console.log(partialForSigner2Body);
      ws.send(partialForSigner2Body);

      // SIGNER_3の部分トランザクションが追加されるのを監視
      const partialForSigner3Body = `{"uid": "${res.uid}", "subscribe": "partialAdded/${signer3AddressString}"}`;
      console.log(partialForSigner3Body);
      ws.send(partialForSigner3Body);

      // SIGNER_1のトランザクションが未承認状態になったのを監視
      const unconfirmedBody = `{"uid": "${res.uid}", "subscribe": "unconfirmedAdded/${signer1AddressString}"}`;
      console.log(unconfirmedBody);
      ws.send(unconfirmedBody);

      // SIGNER_1のトランザクションが承認されるの監視
      const confirmedBody = `{"uid": "${res.uid}", "subscribe": "confirmedAdded/${signer1AddressString}"}`;
      console.log(confirmedBody);
      ws.send(confirmedBody);

      // SIGNER_1のトランザクションがエラーになったのを監視
      const statusBody = `{"uid": "${res.uid}", "subscribe": "status/${signer1AddressString}"}`;
      console.log(statusBody);
      ws.send(statusBody);

      // 新しいブロックを監視
      const blockBody = `{"uid": "${res.uid}", "subscribe": "block"}`;
      console.log(blockBody);
      ws.send(blockBody);

      // ファイナライズされたブロックを監視
      const finalizedBlockBody = `{"uid": "${res.uid}", "subscribe": "finalizedBlock"}`;
      console.log(finalizedBlockBody);
      ws.send(finalizedBlockBody);
    }

    // ハッシュロックトランザクションが未承認になったときに発火
    if (
      res.topic === `unconfirmedAdded/${signer1AddressString}` &&
      res.data.meta.hash === hashLockTransactionHash.toString()
    ) {
      console.log("hashLockTransaction unconfirmed");
    }

    // ハッシュロックトランザクションが承認されたときに発火
    if (
      res.topic === `confirmedAdded/${signer1AddressString}` &&
      res.data.meta.hash === hashLockTransactionHash.toString()
    ) {
      console.log(
        `hashLockTransaction confirmed at block ${res.data.meta.height}`
      );
      hashLockTransactionHeight = parseInt(res.data.meta.height);

      // ハッシュロックトランザクションが承認されたら、アグリゲートボンデッドトランザクションをアナウンス
      console.log("announce aggregateBondedTransaction");
      try {
        const transactionRoutesApi = new TransactionRoutesApi(configuration);
        console.log(aggregateBondedTransactionPayload);
        const response = await transactionRoutesApi.announcePartialTransaction({
          transactionPayload: aggregateBondedTransactionPayload,
        });
        console.log(response.data);
      } catch (error) {
        console.error(error);
      }
    }

    // SIGNER_2への部分トランザクションが検知されたとき(≒署名要求が届いたとき)に発火
    if (
      res.topic === `partialAdded/${signer2AddressString}` &&
      res.data.meta.hash === aggregateBondedTransactionHash.toString()
    ) {
      console.log(`partialAdded for SIGNER_2 ${signer2AddressString}`);

      // WebSocketからのメッセージの中にどのようなトランザクションかという情報が含まれているので、連署の際には署名対象のハッシュのアグリゲートボンデッドトランザクションがどのような内容か、適切な確認を行った上で連署することがとても大事です。
      console.dir(res, { depth: null });

      // 連署の用の署名データを作成しネットワークにアナウンスする ... コードを書かずともウォレットからも実行可能だがサンプルコードの紹介意図でここに処理を書いておく
      console.log("cosign aggregateBondedTransaction by SIGNER_2");
      try {
        // signer2による連署作成 ... アグリゲートボンデッドトランザクションのハッシュのバイナリに対して署名する
        const signatureBySigner2 = new Signature(
          signer2KeyPair.sign(
            hexToUint8(aggregateBondedTransactionHash.toString())
          ).bytes
        );
        const cosignatureBySigner2 = new Cosignature();
        cosignatureBySigner2.signerPublicKey = new PublicKeyModel(
          signer2KeyPair.publicKey.bytes
        );
        cosignatureBySigner2.signature = signatureBySigner2;

        const transactionRoutesApi = new TransactionRoutesApi(configuration);
        const transactionRoutesApiAnnounceCosignatureTransactionRequest: TransactionRoutesApiAnnounceCosignatureTransactionRequest =
          {
            cosignature: {
              parentHash: aggregateBondedTransactionHash.toString(),
              signature: cosignatureBySigner2.signature.toString(),
              signerPublicKey: cosignatureBySigner2.signerPublicKey.toString(),
              version: cosignatureBySigner2.version.toString(),
            },
          };
        const response =
          await transactionRoutesApi.announceCosignatureTransaction(
            transactionRoutesApiAnnounceCosignatureTransactionRequest
          );
        console.log(response.data);
      } catch (error) {
        console.error(error);
      }
    }

    // SIGNER_3への部分トランザクションが検知されたとき(≒署名要求が届いたとき)に発火
    if (
      res.topic === `partialAdded/${signer3AddressString}` &&
      res.data.meta.hash === aggregateBondedTransactionHash.toString()
    ) {
      console.log(`partialAdded for SIGNER_3 ${signer3AddressString}`);

      // WebSocketからのメッセージの中にどのようなトランザクションかという情報が含まれているので、連署の際には署名対象のハッシュのアグリゲートボンデッドトランザクションがどのような内容か、適切な確認を行った上で連署することがとても大事です。
      console.dir(res, { depth: null });

      // 連署の用の署名データを作成しネットワークにアナウンスする
      console.log("cosign aggregateBondedTransaction by SIGNER_3");
      try {
        // signer3による連署作成 ... アグリゲートボンデッドトランザクションのハッシュのバイナリに対して署名する
        const signatureBySigner3 = new Signature(
          signer3KeyPair.sign(
            hexToUint8(aggregateBondedTransactionHash.toString())
          ).bytes
        );
        const cosignatureBySigner3 = new Cosignature();
        cosignatureBySigner3.signerPublicKey = new PublicKeyModel(
          signer3KeyPair.publicKey.bytes
        );
        cosignatureBySigner3.signature = signatureBySigner3;

        const transactionRoutesApi = new TransactionRoutesApi(configuration);
        const requestParameters: TransactionRoutesApiAnnounceCosignatureTransactionRequest =
          {
            cosignature: {
              parentHash: aggregateBondedTransactionHash.toString(),
              signature: cosignatureBySigner3.signature.toString(),
              signerPublicKey: cosignatureBySigner3.signerPublicKey.toString(),
              version: cosignatureBySigner3.version.toString(),
            },
          };
        const response =
          await transactionRoutesApi.announceCosignatureTransaction(
            requestParameters
          );
        console.log(response.data);
      } catch (error) {
        console.error(error);
      }
    }

    // アグリゲートボンデッドトランザクションが未承認になったときに発火
    if (
      res.topic === `unconfirmedAdded/${signer1AddressString}` &&
      res.data.meta.hash === aggregateBondedTransactionHash.toString()
    ) {
      console.log("aggregateBondedTransaction unconfirmed");
    }

    // アグリゲートボンデッドトランザクションが承認されたときに発火
    if (
      res.topic === `confirmedAdded/${signer1AddressString}` &&
      res.data.meta.hash === aggregateBondedTransactionHash.toString()
    ) {
      console.log("aggregateBondedTransaction confirmed");
      aggregateBondedTransactionHeight = parseInt(res.data.meta.height);
    }

    // ブロック生成時に発火
    if (res.topic === `block`) {
      console.log("block");
      blockHeight = parseInt(res.data.block.height);
    }

    // ブロックのファイナライズ時に発火
    if (res.topic === `finalizedBlock`) {
      console.log("finalizedBlock");
      console.log(res);
      finalizedBlockHeight = parseInt(res.data.height);
    }

    // トランザクションがエラーになったときに発火
    if (
      res.topic === `status/${signer1AddressString}` &&
      res.data.hash === aggregateBondedTransactionHash.toString()
    ) {
      console.log(res.data.code);
      ws.close();
    } else {
      console.log(res);
    }

    // confirmationHeightブロック後に監視終了
    if (
      aggregateBondedTransactionHeight !== 0 &&
      aggregateBondedTransactionHeight + confirmationHeight - 1 <= blockHeight
    ) {
      console.log(
        `${confirmationHeight} blocks confirmed. transactionHeight is ${aggregateBondedTransactionHeight} blockHeight is ${blockHeight}.`
      );

      // トランザクションの状態を確認し監視終了
      try {
        const transactionStatusRoutesApi = new TransactionStatusRoutesApi(
          configuration
        );
        const transactionStatusDTO: TransactionStatusDTO = (
          await transactionStatusRoutesApi.getTransactionStatus({
            hash: aggregateBondedTransactionHash.toString(),
          })
        ).data;
        if (transactionStatusDTO.group === TransactionGroupEnum.Confirmed) {
          // トランザクションの確認後の処理をここに書く
          ws.close();
        } else if (
          transactionStatusDTO.group === TransactionGroupEnum.Unconfirmed
        ) {
          // 未承認に再度戻っていた場合は監視をそのまま続ける
          console.log("rollback detected. transaction is unconfirmed.");
        } else {
          // トランザクション未承認のまま消えていた場合の処理をここに書く ... 例. 状態確認して再アナウンス等
          console.log("rollback detected. transaction disappeared.");
          ws.close();
        }
      } catch (err) {
        console.error(err);
        ws.close();
      }
    } else {
      console.log(
        `wait for ${confirmationHeight} blocks. transactionHeight is ${aggregateBondedTransactionHeight} blockHeight is ${blockHeight}.`
      );
    }

    // finalizedBlockHeightが対象ブロックを追い越した後に監視終了
    if (
      aggregateBondedTransactionHeight !== 0 &&
      aggregateBondedTransactionHeight <= finalizedBlockHeight
    ) {
      console.log(
        `${finalizedBlockHeight} block finalized. transactionHeight is ${aggregateBondedTransactionHeight} blockHeight is ${blockHeight}.`
      );

      // トランザクションの状態を確認し監視終了
      try {
        const transactionStatusRoutesApi = new TransactionStatusRoutesApi(
          configuration
        );
        const transactionStatusDTO: TransactionStatusDTO = (
          await transactionStatusRoutesApi.getTransactionStatus({
            hash: aggregateBondedTransactionHash.toString(),
          })
        ).data;
        if (transactionStatusDTO.group === TransactionGroupEnum.Confirmed) {
          // トランザクションの確認後の処理をここに書く
          ws.close();
        } else if (
          transactionStatusDTO.group === TransactionGroupEnum.Unconfirmed
        ) {
          // 未承認に再度戻っていた場合は監視をそのまま続ける
          console.log("rollback detected. transaction is unconfirmed.");
        } else {
          // トランザクション未承認のまま消えていた場合の処理をここに書く ... 例. 状態確認して再アナウンス等
          console.log("rollback detected. transaction disappeared.");
          ws.close();
        }
      } catch (err) {
        console.error(err);
        ws.close();
      }
    } else {
      console.log(
        `wait for finalized block. transactionHeight is ${aggregateBondedTransactionHeight} blockHeight is ${blockHeight}.`
      );
    }
  });

  // ハッシュロックトランザクションのアナウンス実行
  try {
    const transactionRoutesApi = new TransactionRoutesApi(configuration);
    console.log(hashLockTransactionPayload);
    const response = await transactionRoutesApi.announceTransaction({
      transactionPayload: hashLockTransactionPayload,
    });
    console.log(response.data);
  } catch (err) {
    console.error(err);
  }
})();

```

実行すると、以下のような流れで処理が行われます。

1. アグリゲートボンデッドトランザクションのデータを生成し、アグリゲートボンデッドトランザクションのハッシュを記録しておきます。
2. ハッシュロックトランザクションのデータを、アグリゲートボンデッドトランザクションのハッシュを指定して、生成します。
3. ハッシュロックトランザクションを送信します。
4. ハッシュロックトランザクションが承認されたら、アグリゲートボンデッドトランザクションを送信します。
5. アグリゲートボンデッドトランザクションが送信されたら、連署が必要なアカウント(今回の場合`SIGNER_1`, `SIGNER_2`)にWebSocketから通知が届くので、トランザクションの内容を確認して、連署を送信します。実運用においては各連署者が各自のWalletで連署するのが自然でしょうか。(また、何に対する署名データを作る必要があるのかという点が技術的に重要で、アグリゲートボンデッドトランザクションのハッシュのバイナリ値に対する署名を連署データとして作成する必要があります。また、連署の送信はトランザクションの送信ではなく、単純にREST APIのエンドポイントを呼び出しているだけで、手数料がかからないという特徴もあります。)
6. 必要な連署が全て揃ったらアグリゲートボンデッドトランザクションが承認され、アグリゲートボンデッドトランザクションの内部トランザクションが一括実行され、
7. ハッシュロックトランザクションでロックされていた送信者が担保にしていた10XYMが送信者に戻ります。

サンプルコードにはコメントを比較的多めに書いておいたので、どこで何をしているかはある程度確認できるかなと思います。もし上手く行かないところがあったら、GitHubや本記事末尾で紹介しているDiscord等でお気軽にコメントください。

:::details 実行時のログの例

```shell
~/symbol-sdk-typescript-sample-1$ npx ts-node send-aggregate-bonded-transaction-to-transfer-from-multi-accounts.ts 

98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9
https://testnet.symbol.fyi/transactions/98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9
B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF
https://testnet.symbol.fyi/transactions/B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF
{"payload": "B80000000000000044C688AD15A7AD2D6073876654CF979F360ECEA76DECB8CC9F33FB2920111C5100F46146FD8A377F414821AA03C0332A6D34FA5A4677C0D44DDE8EF5D11F850CECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D0680000000001984841E04700000000000002B4C3E305000000C8B6532DDB16843A8096980000000000801600000000000098A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9"}
{ message: 'packet 9 was pushed to the network via /transactions' }
connection open
uid : RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "partialAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "partialAdded/TD2UFKX6Z2GSA2DF2UJ5NZVKCT3ZXMVTRYD6CRQ"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "partialAdded/TAB2CPNR4HZO7RDEWXXY43ZF2ZQWCLZLWARG35I"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "unconfirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "confirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "status/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "block"}
{"uid": "RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS", "subscribe": "finalizedBlock"}
{ uid: 'RINGAAGXI2YPIBEMJF2B6EN4A5MCQLTS' }
wait for 6 blocks. transactionHeight is 0 blockHeight is 0.
wait for finalized block. transactionHeight is 0 blockHeight is 0.
hashLockTransaction unconfirmed
{
  topic: 'unconfirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: '44C688AD15A7AD2D6073876654CF979F360ECEA76DECB8CC9F33FB2920111C5100F46146FD8A377F414821AA03C0332A6D34FA5A4677C0D44DDE8EF5D11F850C',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16712,
      maxFee: '18400',
      deadline: '25296090114',
      mosaicId: '3A8416DB2D53B6C8',
      amount: '10000000',
      duration: '5760',
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9'
    },
    meta: {
      hash: 'B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF',
      merkleComponentHash: 'B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF',
      height: '0'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 0.
wait for finalized block. transactionHeight is 0 blockHeight is 0.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '19022EBDA8361022B70701395C5D4DA78E42D5919D3725C5CA8DBC2F3D15BD594EA0CFF47D548B04510D8D90EA9A42B28401821F185E874F12286D8625272A0C',
      signerPublicKey: '9DE9407F87FBCB9E3D1CF3C257A71CD401605DDE19BED8775A012E635D38D7D3',
      version: 1,
      network: 152,
      type: 33091,
      height: '693868',
      timestamp: '25288904639',
      difficulty: '10000000000000',
      proofGamma: '6326E725FCAFD4F5D9A790A92D541AE1B4BA5E905DEB5B65003054B72A400B29',
      proofVerificationHash: '25171A9FD117BC582D672DEA4D5B5C44',
      proofScalar: 'DF66530C9300BC0483510D81D69D80FA3CECBA19C4E634632C16601BCEBE2101',
      previousBlockHash: '26F8F11B686EFB7D3DD608BDB9FF78A88391C2871A2F86B38BE38FC73B673603',
      transactionsHash: 'B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF',
      receiptsHash: '6A85112D64D62E8C28EFD647A2B2B125F1693BE7C4EE92F5505F05B6F2AEDC55',
      stateHash: 'C643EDD54C7206630890B1883C5A24383B4FD2430BF311FC6070DD6929179CDC',
      beneficiaryAddress: '988C91EC5E79836DD1C549E3AAAEACFD02F69A364F809BB2',
      feeMultiplier: 100
    },
    meta: {
      hash: 'DF2F5D81834EA17CEC1445C96C8FB8CE43FBF048A42BC4D92C5FCC80C6E481A1',
      generationHash: 'FD024F4C9D97BCE3748EB8F104529ACC4CD9CDE2DD0DB51837238A15BFE48264'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693868.
wait for finalized block. transactionHeight is 0 blockHeight is 693868.
hashLockTransaction confirmed at block 693868
announce aggregateBondedTransaction
{"payload": "C801000000000000C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068000000000198414220FD00000000000002B4C3E305000000F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F20010000000000006000000000000000ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068000000000198544198D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B0000010000000000C8B6532DDB16843A00E1F50500000000600000000000000058363286FB1830AB9C8A6716EB6522B4AD6F566FBD9EEE19D1C49B94EF1BAED8000000000198544198D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B0000010000000000C8B6532DDB16843A00E1F50500000000600000000000000055A1685BBB2F41E85ACEA4AA940B70D4CD653F82FF8B97586B380DE42595B4CF000000000198544198D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B0000010000000000C8B6532DDB16843A00E1F50500000000"}
{
  message: 'packet 256 was pushed to the network via /transactions/partial'
}
{
  topic: 'confirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: '44C688AD15A7AD2D6073876654CF979F360ECEA76DECB8CC9F33FB2920111C5100F46146FD8A377F414821AA03C0332A6D34FA5A4677C0D44DDE8EF5D11F850C',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16712,
      maxFee: '18400',
      deadline: '25296090114',
      mosaicId: '3A8416DB2D53B6C8',
      amount: '10000000',
      duration: '5760',
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9'
    },
    meta: {
      hash: 'B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF',
      merkleComponentHash: 'B67D4933074D2D42FEFDA88936F2870E69D0918991C2DC153763B7A2BDC95BAF',
      height: '693868'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693868.
wait for finalized block. transactionHeight is 0 blockHeight is 693868.
partialAdded for SIGNER_3 TAB2CPNR4HZO7RDEWXXY43ZF2ZQWCLZLWARG35I
{
  topic: 'partialAdded/TAB2CPNR4HZO7RDEWXXY43ZF2ZQWCLZLWARG35I',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [
        {
          transaction: {
            signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
            version: 1,
            network: 152,
            type: 16724,
            recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
            mosaics: [ { id: '3A8416DB2D53B6C8', amount: '100000000' } ]
          }
        },
        {
          transaction: {
            signerPublicKey: '58363286FB1830AB9C8A6716EB6522B4AD6F566FBD9EEE19D1C49B94EF1BAED8',
            version: 1,
            network: 152,
            type: 16724,
            recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
            mosaics: [ { id: '3A8416DB2D53B6C8', amount: '100000000' } ]
          }
        },
        {
          transaction: {
            signerPublicKey: '55A1685BBB2F41E85ACEA4AA940B70D4CD653F82FF8B97586B380DE42595B4CF',
            version: 1,
            network: 152,
            type: 16724,
            recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
            mosaics: [ { id: '3A8416DB2D53B6C8', amount: '100000000' } ]
          }
        }
      ]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '0000000000000000000000000000000000000000000000000000000000000000',
      height: '0'
    }
  }
}
cosign aggregateBondedTransaction by SIGNER_3
partialAdded for SIGNER_2 TD2UFKX6Z2GSA2DF2UJ5NZVKCT3ZXMVTRYD6CRQ
{
  topic: 'partialAdded/TD2UFKX6Z2GSA2DF2UJ5NZVKCT3ZXMVTRYD6CRQ',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [
        {
          transaction: {
            signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
            version: 1,
            network: 152,
            type: 16724,
            recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
            mosaics: [ { id: '3A8416DB2D53B6C8', amount: '100000000' } ]
          }
        },
        {
          transaction: {
            signerPublicKey: '58363286FB1830AB9C8A6716EB6522B4AD6F566FBD9EEE19D1C49B94EF1BAED8',
            version: 1,
            network: 152,
            type: 16724,
            recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
            mosaics: [ { id: '3A8416DB2D53B6C8', amount: '100000000' } ]
          }
        },
        {
          transaction: {
            signerPublicKey: '55A1685BBB2F41E85ACEA4AA940B70D4CD653F82FF8B97586B380DE42595B4CF',
            version: 1,
            network: 152,
            type: 16724,
            recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
            mosaics: [ { id: '3A8416DB2D53B6C8', amount: '100000000' } ]
          }
        }
      ]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '0000000000000000000000000000000000000000000000000000000000000000',
      height: '0'
    }
  }
}
cosign aggregateBondedTransaction by SIGNER_2
{
  topic: 'partialAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [Array]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '0000000000000000000000000000000000000000000000000000000000000000',
      height: '0'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693868.
wait for finalized block. transactionHeight is 0 blockHeight is 693868.
{
  message: 'packet 257 was pushed to the network via /transactions/cosignature'
}
{
  topic: 'partialAdded/TD2UFKX6Z2GSA2DF2UJ5NZVKCT3ZXMVTRYD6CRQ',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [Array]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '0000000000000000000000000000000000000000000000000000000000000000',
      height: '0'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693868.
wait for finalized block. transactionHeight is 0 blockHeight is 693868.
{
  message: 'packet 257 was pushed to the network via /transactions/cosignature'
}
{
  topic: 'partialAdded/TAB2CPNR4HZO7RDEWXXY43ZF2ZQWCLZLWARG35I',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [Array]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '0000000000000000000000000000000000000000000000000000000000000000',
      height: '0'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693868.
wait for finalized block. transactionHeight is 0 blockHeight is 693868.
aggregateBondedTransaction unconfirmed
{
  topic: 'unconfirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [Array],
      cosignatures: [Array]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '86695BED8B1AF7C2C4A56D0115BB1DC91C2EA9FEC32EE38658B2C8D708A4FCD0',
      height: '0'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693868.
wait for finalized block. transactionHeight is 0 blockHeight is 693868.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '7A4DA33EAE8FFBB4192A3BFD3E19B4E9B005D9891473F82749C20E4CECD0170625A6962B67F3ED3460F2CF728BA285AC85886AAE872335ACAB12C031F702EC01',
      signerPublicKey: 'B0F840307A8D8A3327DEDEAB4D0BC4EC8D1420F6E7E849F562852789465E6598',
      version: 1,
      network: 152,
      type: 33091,
      height: '693869',
      timestamp: '25288943343',
      difficulty: '10000000000000',
      proofGamma: 'CF628C7C1C54C7DE7D4842DF70C5F87573F12E73055217FDA800CE8A74267485',
      proofVerificationHash: '408B6CEAD0CE90A761EACB939389A3FB',
      proofScalar: 'A3E7CCA5FA1629C4E8EAAFEACEB486ED3562F009319FCC6EB6D03C5DB825FE07',
      previousBlockHash: 'DF2F5D81834EA17CEC1445C96C8FB8CE43FBF048A42BC4D92C5FCC80C6E481A1',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'DBA5101561A0B5805FFB1C4741C9876ACC2CC70543AE7B7F01D91528B7CB390F',
      stateHash: 'E87D14EEDAA404AA8B9782452E73944CFF068D22AFA7199767929B635F4F2B6B',
      beneficiaryAddress: '988DE70F51DEE436904E8D0D41FADE0D49758981293DAB08',
      feeMultiplier: 0
    },
    meta: {
      hash: '6B65F77C76444E49193C7760BB4EA02C5F5BD05B0086BEE076CE11C0A3FE7103',
      generationHash: 'E7799757CBF4E1A180994716FAFCDCF086F147F75C09627C598E696742288799'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693869.
wait for finalized block. transactionHeight is 0 blockHeight is 693869.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '537B3658DE1C8FF2DF0C8E71EB7E89E56C8F9D532D8EB9DE525237BF5CF44C0DB999046BEDE07DBD96E9AA5695D01E11084505B5E1B8E759D539135277887D0D',
      signerPublicKey: '531A93CCA6BC5D1FC50BB63D9E6ABBE558A7F96729EB31658225FBBC90900AF8',
      version: 1,
      network: 152,
      type: 33091,
      height: '693870',
      timestamp: '25288985688',
      difficulty: '10000000000000',
      proofGamma: '4A9A54EC63914C49C0E10BA5962FA08486D8CE5AF920A3DF119198E0FF65B0E9',
      proofVerificationHash: '60F7C179A65E48F2AC1C9594B73853C4',
      proofScalar: 'A546FE70D4F1CA0D69BDA6F47861C1DD4AB380D41759785CA5E6C622B72E840C',
      previousBlockHash: '6B65F77C76444E49193C7760BB4EA02C5F5BD05B0086BEE076CE11C0A3FE7103',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '5393AAAA31A06A9FA0A3876B734DEA6214A4FE182E2E87302D147E2A528B0F2E',
      stateHash: 'B1C6308CE6C181D4BD6F61C3D4534160C70FB385BF643B771882E0603C409DAE',
      beneficiaryAddress: '988C91EC5E79836DD1C549E3AAAEACFD02F69A364F809BB2',
      feeMultiplier: 0
    },
    meta: {
      hash: '5A2183A86DB69F356912BEDFBD4C27F7D23480BF24B943B1523AAEA14ACCB513',
      generationHash: 'F13523E5A56808B0E527EE5D7EA3DAFD94D910BAE0D153C7B5B170DFFAC8B095'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693870.
wait for finalized block. transactionHeight is 0 blockHeight is 693870.
block
{
  topic: 'block',
  data: {
    block: {
      signature: 'A30461882534DD74D2E4907F7D29BFBF82EBAABC4F7F346BA1B912A9AA68887B6EB0297D8042F0C8B24F41558943F0AAE6A48EA34375B6A6E63AF0DD63692C09',
      signerPublicKey: 'DC20B243B63246C9E75E4FB5ED236513A005454393E93C8A4CE6EDEE323C2DDB',
      version: 1,
      network: 152,
      type: 33091,
      height: '693871',
      timestamp: '25289018682',
      difficulty: '10000000000000',
      proofGamma: 'CBCA7156E5CECE836351F510FDE7D2736809EF7C8B72FE91439271FED30C7160',
      proofVerificationHash: '25F40637950BEE1E192854327622CE80',
      proofScalar: '9691DD883CD71A6B018185A9997BA6A8A0F2A71BF83597961572E63088E39A09',
      previousBlockHash: '5A2183A86DB69F356912BEDFBD4C27F7D23480BF24B943B1523AAEA14ACCB513',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '6BF3137C55064F0D4C152A24771E048977190706B8F120F23FC3DD3829E97481',
      stateHash: '8FA1B7DA5DD0843E57FA7440EF43541068949FB6F5982B83AAE260B36E6AFB98',
      beneficiaryAddress: '986C859DE41BDF90D47B08CC602654AA2EDCE5617260C694',
      feeMultiplier: 0
    },
    meta: {
      hash: 'A2D99BE9908E0D1DA8897B6F34500D0C4D0917EA69845049AE7C13EBAFE2CA89',
      generationHash: 'F3BF4D8326B53E6E90CE318B73A589E6575625B37D51855174F96F24EAA81262'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693871.
wait for finalized block. transactionHeight is 0 blockHeight is 693871.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '5428300B635A251CC97032FEB563223D7F4817C7540E66967769D2E3DBB01FA13971FBBB461D9FEA95E3FF086EE91D0E815731D8D474AFF7F35F5FF7873A8206',
      signerPublicKey: 'BF2EAFD7C2B1E84C814B797332CA10E82CC3E3C1E7BC8ACC4640E7FD33C90A2C',
      version: 1,
      network: 152,
      type: 33091,
      height: '693872',
      timestamp: '25289050364',
      difficulty: '10000000000000',
      proofGamma: '8D4015CC056B97E32A8CD3E0F987FBF5581F931B1D66A7E2078782F388756E27',
      proofVerificationHash: '5FA64036FB415D5DAD3DCE2A0466D39E',
      proofScalar: '4220AA76FF1DB157883E9FFD0A3626D1E568750821B88FB7BAB7887A16DDFB00',
      previousBlockHash: 'A2D99BE9908E0D1DA8897B6F34500D0C4D0917EA69845049AE7C13EBAFE2CA89',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '55917278DD7CA2D0BAEF12D5DD072CC929A2187AAC78E239E043A1F4198AC5DF',
      stateHash: 'FDB3F333FE62D6CCEE15EA72286193316AE6B67700D3E2E8ACF3B4A0AF018673',
      beneficiaryAddress: '985F693ED8D58BC10F3E428E600E17F704BE042B03198B2A',
      feeMultiplier: 0
    },
    meta: {
      hash: '50AFE35B27631B4EE9B411E8E49EFA344664A195925C3231D5538058AED5DBD9',
      generationHash: 'F63D64211F25BC5E5B0CFCCF5748F5CFAAEBDCA113BEAC5D6CB0B1159B08F15D'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693872.
wait for finalized block. transactionHeight is 0 blockHeight is 693872.
block
{
  topic: 'block',
  data: {
    block: {
      signature: 'E53CBEDF69050B6041B684566E65A8EEFC38CB8D7DFA622DE54547E1AD18A54147A8CA7F738FFAE5C00E819B214224EEBC2FF3DD7CEF0D7D2282E9C1999C8506',
      signerPublicKey: 'F9A5A66EBE9AD5EAAAF07AF208BF33FB0B24C2049EFF99B65AD1BB46809852DE',
      version: 1,
      network: 152,
      type: 33091,
      height: '693873',
      timestamp: '25289076508',
      difficulty: '10000000000000',
      proofGamma: '3A82FF49F2C3C57FD5FC504E29BABF41A2A40F9985D62159AB64BE2D798D124C',
      proofVerificationHash: '228A287B5A31D4237FFDF2FF59448A28',
      proofScalar: 'D928496462CD67DF780A10382E697FFFD9F2EB20C2A6281FE3AAFFFC6A6BF406',
      previousBlockHash: '50AFE35B27631B4EE9B411E8E49EFA344664A195925C3231D5538058AED5DBD9',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'F426FEC961E15E6F1A76117DA8363DE4064391816B024B5134F6BA19DCAF5DC0',
      stateHash: 'D068F382DCEA882E4BAEBBA50BC1FE894C5ECF99FECB53003DDD5019445E9614',
      beneficiaryAddress: '98D3970759CB13CD9E6BD7FBEBD7F909F34ADA4E079B1A4D',
      feeMultiplier: 0
    },
    meta: {
      hash: '1BE1875AD73B683DF9DC69951CDF889C19722B77540830F64BA4E932D7F96A0F',
      generationHash: 'FA8E79E1F6C1BEC0094CFDA8D9EC28096B485E28BC74B999DB34D6837B216A55'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693873.
wait for finalized block. transactionHeight is 0 blockHeight is 693873.
block
{
  topic: 'block',
  data: {
    block: {
      signature: 'BC21468F9FDA1BEB8AA8D4F6BC865269A90EE7561B7B51A13210E7394CC2C06F245673BFE18788AA4C0164B04F74E4A59DB57EC920E2123A4F7884A6B886100F',
      signerPublicKey: 'F9A5A66EBE9AD5EAAAF07AF208BF33FB0B24C2049EFF99B65AD1BB46809852DE',
      version: 1,
      network: 152,
      type: 33091,
      height: '693874',
      timestamp: '25289111539',
      difficulty: '10000000000000',
      proofGamma: '03161681AE06594F6E757F6F5E1F498A551E8C82791F76C124ABE8F22341183E',
      proofVerificationHash: 'E041165D0EEECCC6E3043A5419511B82',
      proofScalar: '8FFCFC7AB9C47C0B1736DCE31437B7C69AB23F7508CDD6018C02DDBD7A310B04',
      previousBlockHash: '1BE1875AD73B683DF9DC69951CDF889C19722B77540830F64BA4E932D7F96A0F',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'F426FEC961E15E6F1A76117DA8363DE4064391816B024B5134F6BA19DCAF5DC0',
      stateHash: '7FEB956B952AA6C73584645AA0C710BE6E23B57848A4375871ECCD11197FE8C7',
      beneficiaryAddress: '98D3970759CB13CD9E6BD7FBEBD7F909F34ADA4E079B1A4D',
      feeMultiplier: 0
    },
    meta: {
      hash: '3A114F48CC3E730B78A67FB25B9D18BC7470B2FC953F9B3BC38A6976B279B3CC',
      generationHash: 'EEEF8DFA3D0FDCCB989021EDF7F504C93CDCA0BC53811E2257BD681E66C8EAF5'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693874.
wait for finalized block. transactionHeight is 0 blockHeight is 693874.
finalizedBlock
{
  topic: 'finalizedBlock',
  data: {
    finalizationEpoch: 965,
    finalizationPoint: 34,
    height: '693860',
    hash: 'E8221959A8B4284A965925B48C14F4E863A28B5167A17F447F3D196301162192'
  }
}
{
  topic: 'finalizedBlock',
  data: {
    finalizationEpoch: 965,
    finalizationPoint: 34,
    height: '693860',
    hash: 'E8221959A8B4284A965925B48C14F4E863A28B5167A17F447F3D196301162192'
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693874.
wait for finalized block. transactionHeight is 0 blockHeight is 693874.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '4DFDB853E49B036EF4F17550AFDC96D52ADE31CEFB5E9751C8D8D4526FC879B01DB57FD093F49D0CCA1064F90EB8D7756DB3A19ECCF6F42E1EE96B9949CE7104',
      signerPublicKey: 'F03EC7EAFDD0D2EE5A0DC6361785B3132917D1998809B694C7B6CC2379BD0760',
      version: 1,
      network: 152,
      type: 33091,
      height: '693875',
      timestamp: '25289150366',
      difficulty: '10000000000000',
      proofGamma: 'D1A28E090037FB981E39DB5C6B44B6CF185F6D5D108288B19539EFC43288B1DD',
      proofVerificationHash: '86A7747CF28F3A5949F62A264D33FBF5',
      proofScalar: '5EC116B64131E6804DD41840CAE2D1A0F81FB7F1F5DFA33AFB6E3E50F7879008',
      previousBlockHash: '3A114F48CC3E730B78A67FB25B9D18BC7470B2FC953F9B3BC38A6976B279B3CC',
      transactionsHash: '86695BED8B1AF7C2C4A56D0115BB1DC91C2EA9FEC32EE38658B2C8D708A4FCD0',
      receiptsHash: 'A12B3AAEA0C4D44774A49ECEE1B23170B178DE0E4FD1C1419FE54B3B8D079B4F',
      stateHash: '9272EEF8E1B71D61F37F241BDBB77057951F53B54FE043E770D74FE4E912B033',
      beneficiaryAddress: '98F77D3A3D15B5D7A535BCB61E36592874EB0CD93403A543',
      feeMultiplier: 97
    },
    meta: {
      hash: '09ED2CE05B9328BA3A78A26EAE620954DA58EBA986EC78C18E87ED8CF8597203',
      generationHash: 'E7708483E13441DCEBC5CE23EC62915A24AC2FF7A37BF9C6EC6BDA94B0C7762D'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 693875.
wait for finalized block. transactionHeight is 0 blockHeight is 693875.
aggregateBondedTransaction confirmed
{
  topic: 'confirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: 'C12B42EDEEC0C6B2BB6E74A84F6AA83AB052A4B68BA60EA0321C0697ED8E073813F500FEE371836FD8EFD606435E15E37E287CE4F963E763BBC20DE1C8C05208',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16961,
      maxFee: '64800',
      deadline: '25296090114',
      transactionsHash: 'F7EBE30A3ECCF3453A2B94E61D7E0F12784F29C5D15F106FEDFFAE32A9C53D1F',
      transactions: [Array],
      cosignatures: [Array]
    },
    meta: {
      hash: '98A12BE36DD7267802A15573263E7D1BAE321586F033DD52E6BC54DA484FB5A9',
      merkleComponentHash: '86695BED8B1AF7C2C4A56D0115BB1DC91C2EA9FEC32EE38658B2C8D708A4FCD0',
      height: '693875'
    }
  }
}
wait for 6 blocks. transactionHeight is 693875 blockHeight is 693875.
wait for finalized block. transactionHeight is 693875 blockHeight is 693875.
block
{
  topic: 'block',
  data: {
    block: {
      signature: 'D8770AA3040FCDFFA46B0FBBD2D14EB0733FD6703288DDACB7FF70BAFE42A03D8F753275E98B2BEFB2FB864429C16616A6705092C545319824080A9BFBE25301',
      signerPublicKey: 'B0F840307A8D8A3327DEDEAB4D0BC4EC8D1420F6E7E849F562852789465E6598',
      version: 1,
      network: 152,
      type: 33091,
      height: '693876',
      timestamp: '25289200539',
      difficulty: '10000000000000',
      proofGamma: '34B7E95DAD46909A8925E24B2BF1116F9F3CE608A85B713A1807BBF732B92CFF',
      proofVerificationHash: 'F9494C681031498EDFE188F05FCB4B1F',
      proofScalar: 'AA3EEDE4D6AB288DC8BFEFDFB69471E8442EA8541670DA7B640B37D4CA14D30E',
      previousBlockHash: '09ED2CE05B9328BA3A78A26EAE620954DA58EBA986EC78C18E87ED8CF8597203',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'DBA5101561A0B5805FFB1C4741C9876ACC2CC70543AE7B7F01D91528B7CB390F',
      stateHash: 'A6E784E82865D6E78BC1D2577053A8092EA422CEB27F07B22897F62460960CCE',
      beneficiaryAddress: '988DE70F51DEE436904E8D0D41FADE0D49758981293DAB08',
      feeMultiplier: 0
    },
    meta: {
      hash: '1C0C880CF31E67B39271023576E0FF5586F8BD5DBABE4F691C4DE970DD194FC2',
      generationHash: 'A64834E51B5913DB06A2F12C9E91262320ABF38B787B2A09283DCF98F4B80D48'
    }
  }
}
wait for 6 blocks. transactionHeight is 693875 blockHeight is 693876.
wait for finalized block. transactionHeight is 693875 blockHeight is 693876.
block
{
  topic: 'block',
  data: {
    block: {
      signature: 'F3E7C1288D31AFC5756E12900D39305CB92D01C29E1C20520AB4A8B5A01DAA8B9538012A1DBA87E354D2753EC51A07D851A79BEB6CA276BE15F10D5D737F770A',
      signerPublicKey: 'F9A5A66EBE9AD5EAAAF07AF208BF33FB0B24C2049EFF99B65AD1BB46809852DE',
      version: 1,
      network: 152,
      type: 33091,
      height: '693877',
      timestamp: '25289229636',
      difficulty: '10000000000000',
      proofGamma: '7B7C8BCC718BC6EC5271E227FF04D65E4198BB691761618B0C4878DA1B2DD345',
      proofVerificationHash: 'E2202EF5DFB97E68A25F8E0670711C07',
      proofScalar: '247B29022F1FC0F83CF26051074BC60560436A6F55413E1D8CC94125A8C7CD01',
      previousBlockHash: '1C0C880CF31E67B39271023576E0FF5586F8BD5DBABE4F691C4DE970DD194FC2',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'F426FEC961E15E6F1A76117DA8363DE4064391816B024B5134F6BA19DCAF5DC0',
      stateHash: 'EEB6EE3C10A6A8D58EF2DDF341BAA831F61B887F13FE8F2055266D107BD90B2B',
      beneficiaryAddress: '98D3970759CB13CD9E6BD7FBEBD7F909F34ADA4E079B1A4D',
      feeMultiplier: 0
    },
    meta: {
      hash: 'D477178C6AF2D1AC2583BD55490BCB3422DA4F0DA20580E11DCF001861881388',
      generationHash: 'F86290436ED12FBE19EF47451AD990B83594D2156840E2B162701D883B755DF9'
    }
  }
}
wait for 6 blocks. transactionHeight is 693875 blockHeight is 693877.
wait for finalized block. transactionHeight is 693875 blockHeight is 693877.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '7252E0012FFB9D9FFED722482974D0B5A9F0C1E59FC306E0E1C40AF61F6261B4B3F4FC5789F2383DFDEFD9282ACB36B0BC40896EBEF0148C50E4C725DBFF0309',
      signerPublicKey: '5813C012EA44F58687F808267D8D6B9A7478C01035C479AA90C5D67D0C4D75F6',
      version: 1,
      network: 152,
      type: 33091,
      height: '693878',
      timestamp: '25289262704',
      difficulty: '10000000000000',
      proofGamma: 'F04563C08E6D6583D3049A451A0BC6AF233D2C6B36508EFB22AC346DB8507B13',
      proofVerificationHash: '55D93E3D2772CD3946E257DFD6760E5B',
      proofScalar: '1AA3CF5C245C0E96FFF0E52658C8DD9536172607835C89B814E742EC6D8F340F',
      previousBlockHash: 'D477178C6AF2D1AC2583BD55490BCB3422DA4F0DA20580E11DCF001861881388',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '2DAF0DD38612C767C6B860BC63C5A2900F22CC8BF6DBB15AE3496E7076EC53E8',
      stateHash: 'B15D8BD2BB6D2AA85FB1EFFA3291A6C04F24C9934CCADDBB4520351E90276FD2',
      beneficiaryAddress: '981409D75596027DC6FC4D44242D13B01B3F26206E51A052',
      feeMultiplier: 0
    },
    meta: {
      hash: 'DA35A1AF121EC881ED3A0145EC3394D48382257C6EBC40B632D0023F6B005DDB',
      generationHash: 'FFFA4D8FC33DD1622E8A4B4A316DC0CC77265DF5DA2B064885273B324E70FF8E'
    }
  }
}
wait for 6 blocks. transactionHeight is 693875 blockHeight is 693878.
wait for finalized block. transactionHeight is 693875 blockHeight is 693878.
block
{
  topic: 'block',
  data: {
    block: {
      signature: 'F2EA9BC6FFAF9BDB0F5BFC37B404EBB03B9C6F55E15A9662953474A107D650F619F5E950D4CD9F13242D196EAECB237B4B15291FB4B71988A22212020EFD3D04',
      signerPublicKey: 'F9A5A66EBE9AD5EAAAF07AF208BF33FB0B24C2049EFF99B65AD1BB46809852DE',
      version: 1,
      network: 152,
      type: 33091,
      height: '693879',
      timestamp: '25289294690',
      difficulty: '10000000000000',
      proofGamma: '7FD1B4F9A77E9E61DBFBB3CCFD1E732495A7A6D44DB3423C916D9FCBB5867668',
      proofVerificationHash: 'EBE3F4AA6925A484BEAF2F580BE8EAF0',
      proofScalar: 'C5EECC9F67F296D1F1054484C5DE440192171FF1D7BD5BE3A06F0701AD402306',
      previousBlockHash: 'DA35A1AF121EC881ED3A0145EC3394D48382257C6EBC40B632D0023F6B005DDB',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'F426FEC961E15E6F1A76117DA8363DE4064391816B024B5134F6BA19DCAF5DC0',
      stateHash: '8ADB6CE1A3C821E45A322221131B4603E28B1325168082120A4076B7F26F8F2F',
      beneficiaryAddress: '98D3970759CB13CD9E6BD7FBEBD7F909F34ADA4E079B1A4D',
      feeMultiplier: 0
    },
    meta: {
      hash: 'F33AFC55BEB5AA6C1CD0EAFB75DADA226FFB81CF0EE890C2ECF4C84822EA83F1',
      generationHash: 'F5B0A71C9B08A8E3A921998454B5CFCF2E4FC15984B9C5D7FFC007A2912B6823'
    }
  }
}
wait for 6 blocks. transactionHeight is 693875 blockHeight is 693879.
wait for finalized block. transactionHeight is 693875 blockHeight is 693879.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '2ADD215F614B6A66729C38B85BD685A2E6FF576C274814F8E48E4732E9E16727A993893CBDF9744B3F04532F690D9F20D712A005783EFF932E18FFEBC2CB2F0F',
      signerPublicKey: 'BF2EAFD7C2B1E84C814B797332CA10E82CC3E3C1E7BC8ACC4640E7FD33C90A2C',
      version: 1,
      network: 152,
      type: 33091,
      height: '693880',
      timestamp: '25289327619',
      difficulty: '10000000000000',
      proofGamma: '2D99D1A4E47513A8034AAC21BA4342D8BAFEF6689E1023E5F72BF0B2574F7794',
      proofVerificationHash: 'A14E13413CF81F2C90CF898910E4F4CE',
      proofScalar: '2AF1E33F725291A0D4829B59257C92C7FB74350FEDE560EE3CD6A3A76D67EA04',
      previousBlockHash: 'F33AFC55BEB5AA6C1CD0EAFB75DADA226FFB81CF0EE890C2ECF4C84822EA83F1',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '55917278DD7CA2D0BAEF12D5DD072CC929A2187AAC78E239E043A1F4198AC5DF',
      stateHash: '3DBE9A825103D4AD5A7BDBE9B713F4FEB7A2597A6F58BCD3A2D5E3AB64A0C42B',
      beneficiaryAddress: '985F693ED8D58BC10F3E428E600E17F704BE042B03198B2A',
      feeMultiplier: 0
    },
    meta: {
      hash: '21B9D4A1B5B4A7898FE20EAFC85EA6228C5A8FB65F9D4EBA9DE8E0E2C0292DD9',
      generationHash: 'F46B96429010F88C6A1EB8494AFD5560189368AC604506C6D0850FFAE0159553'
    }
  }
}
6 blocks confirmed. transactionHeight is 693875 blockHeight is 693880.
wait for finalized block. transactionHeight is 693875 blockHeight is 693880.
connection closed
```

:::

## まとめ

Symbolブロックチェーンの大きな個性の一つであるアグリゲートボンデッドトランザクションの概要と、そのサンプルコードをご紹介しました。

アグリゲートボンデッドトランザクションの内部トランザクションとして、一括実行したいトランザクションを配列にして指定するだけで、多様なトランザクションを一括実行できる利便性を感じて頂けると幸いです。

実際の社会においては「本当はこれとこれとこれを1セットで実行したい」けれど「色々と制約があってある程度まとめてやらざるを得ない」とか「個別にやるしかないけど全体として抜け漏れなくやらないといけなくて大変」みたいな課題はかなりあるのではないかと思っています。

それら課題の解決に、アグリゲートボンデッドトランザクションの「関係者の署名が揃ったら全て一括で自動実行される」という仕組みがとても有効に働くのではないかと思います。

皆さまの身近な課題解決について考える際に、こういった仕組みがSymbolブロックチェーンにあることを思い出して頂ければ幸いです。

## 最後に

もしNEMTUSに対しNEMやSymbol関連記事の寄稿や、公開したSDKについて何かありましたら、以下GitHubにて記事やSDKを公開しておりますので、お気軽にDiscussionやIssueやPull Request等、連携くださいますと幸いです。どんな形のContributionも大歓迎です。

- [https://github.com/nemtus/](https://github.com/nemtus/)

NEMTUSとして、今後も継続的に、NEM, Symbolに関する様々な技術情報を継続的に発信していくとともにエコシステムへ貢献していきたいと考えていますので、今後ともどうぞよろしくお願いします。

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

## NEM/SymbolやNEMTUSの開発情報に興味がある方へ

NEM/Symbol関連の開発に興味のある方は、ぜひ日本の開発者向けコミュニティのDiscordにご参加ください。

- [Discord of nem Japan UserGroup](https://discord.gg/H85N3C8eZW)

NEMTUSにて開発している内容について、もしご興味ある方は、以下のNEMTUSのDiscordにてGitHubでの動きを流しているチャンネルもあるので、ぜひお気軽にご参加ください。

- [Discord of NEMTUS](https://discord.gg/5qKH2HWxe8)
