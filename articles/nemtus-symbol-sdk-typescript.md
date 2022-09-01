---
title: "symbol-sdkのJavaScript向け新npmパッケージの簡易的なTypeScript化とトランザクション送信サンプル紹介"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blockchain", "symbol", "typescript", "javascript", "npm"]
published: true
---

# SymbolブロックチェーンのJavaScript向け新公式SDKのTypeScript化及びトランザクション送信サンプルコードの紹介

## 要約

この記事では、[version 3以降のsymbol-sdkのJavaScript向けnpmパッケージ](https://github.com/symbol/symbol/tree/dev/sdk/javascript)を元に、[簡易的にTypeScript対応させたnpmパッケージ @nemtus/symbol-sdk-typescript](https://github.com/nemtus/symbol/tree/dev/sdk/javascript)を作成した方法と、それを使ってSymbolブロックチェーンでトランザクションを送信するためのサンプルコードを紹介します。

参考にさせてもらったのは以下リンク「.jsファイルから.d.tsファイルを生成する」の方法です。

https://www.typescriptlang.org/ja/docs/handbook/declaration-files/dts-from-js.html

詳細については、SDKのレポジトリやnpmパッケージの以下リンクをご参照ください。

- @nemtus/symbol-sdk-typescript ... 作成した簡易的なTypeScript対応済のnpmパッケージ
  - GitHub
    - [https://github.com/nemtus/symbol/tree/dev/sdk/javascript](https://github.com/nemtus/symbol/tree/dev/sdk/javascript)
  - npm
    - [https://www.npmjs.com/package/@nemtus/symbol-sdk-typescript](https://www.npmjs.com/package/@nemtus/symbol-sdk-typescript)

## 背景

先日公開した以下記事でも説明させてもらった通り、

https://zenn.dev/nemtus/articles/nemtus-symbol-sdk-openapi-generator-typescript

SymbolブロックチェーンのJavaScript向けSDKは、version2系のTypeScript/JavaScript向けSDKが非推奨となり、version3系のシンプルでコア領域にスコープを絞ったJavaScript向けSDKの開発とメンテナンスがコア開発チームによって進められています。

そのversion3系の新しいSDKはJavaScriptで書かれているため、素直にTypeScript向け環境で使おうとすると、`モジュール '***' またはそれに対応する型宣言が見つかりません。`といったエラーになります。TypeScriptが大幅に普及してきたモダンなフロントエンド開発の環境において、この状況はあまり好ましいものではありませんでした。

そこで、この状況を改善したく、何らかの方法でTypeScript化を試みるため、以下の方法を検討しました。

1. version2系のTypeScript向けSDKのフォークを行い、NEMTUSやコミュニティの有志の開発者で独自にメンテナンスを継続する
2. version3系のJavaScript向けSDKの実装を参考に、NEMTUSやコミュニティの有志の開発者で、TypeScript向けSDKの実装を行い、開発・メンテナンスを継続する
3. version3系のJavaScript向けSDKの実装をフォークしてそのまま使用し、NEMTUSやコミュニティの有志の開発者で、TypeScript向けに必要な型定義ファイル「.d.ts」を追加 or 自動生成して簡易的なTypeScript向けSDKの実装を行い、開発・メンテナンスを継続する

1については、重要と思われる依存先パッケージのメンテナンスの難易度が高く時間を溶かしそうで断念しました。

2についても、NEM/Symbolブロックチェーンのトランザクションのデータをどのようにバイナリ化して扱うかをDSL的に宣言してある部分への正確な理解が必要で、時間がかかりそうで難易度が高く、初手としては時間を溶かすだけになりそうで保留としました。

最終的に、コミュニティの有志の開発者の方々のアドバイスも頂き、3の方法で取り組むことにしました。TypeScript 3.7以降では、JSDocで書かれたコメントの情報を使って、JavaScriptのファイル群から型定義ファイルの「.d.ts」ファイルを生成できるようになっているとのことだったので、その方法で取り組んだところ、公式SDKにJSDocのコメントが比較的しっかり書かれていたこともあり、(型定義の不足はどうしてもあるものの、)TypeScript環境である程度素直に使えるSDKをリリースすることができました。

## .jsファイルから.d.tsファイルを生成する方法

細かい方法は以下資料をご参照ください。ポイントは`"allowJs": true,`, `"declaration": true,`です。

https://www.typescriptlang.org/ja/docs/handbook/declaration-files/dts-from-js.html

## 実際に作成した@nemtus/symbol-sdk-typescriptを使ってトランザクションを送信するサンプルコードの紹介

### Setup

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

#### テスト用アカウントの作成と環境変数へのセット

Symbolブロックチェーンの公式ウォレットを以下リンクからダウンロードして、テストネットで以下のようにテスト用のアカウントを1個作ってフォーセットからテスト用のトークンを取得しましょう。そして、そのアカウントの秘密鍵、公開鍵、アドレスを記録しておきましょう。

[https://github.com/symbol/desktop-wallet/releases](https://github.com/symbol/desktop-wallet/releases)

記録できたら、以下のように`.env`ファイルに秘密鍵を記入しておきましょう。

```shell:.env
SIGNER_1_PRIVATE_KEY ="PUT_YOUR_PRIVATE_KEY_HERE";

```

:::message alert
ブロックチェーンの世界において、秘密鍵は、そのアカウントの全てを実行できる最強かつ唯一の権限を持っていると言えるでしょう。秘密鍵が漏洩するとそのアカウントのあらゆる資産や秘密情報を失うことになるので、秘密鍵の管理は可能な限り慎重を期してください。

今回はテストネットにて使い捨てのアカウントを作って試すという前提で、dotenvを用いた簡易的な環境変数のような形で秘密鍵を使っていますが、メインネットや本番環境のような実際に価値がある資産と紐づいたネットワーク上でのアカウント管理では、さらに厳重な、適切な方法で、秘密鍵を保護することを強く推奨します。
:::

### SampleCode

これで環境が整いました。以下ファイルを作成し、`npx ts-node send-transfer-transaction.ts`で実行してみましょう。

```typescript:send-transfer-tx.ts
import { SymbolFacade } from "@nemtus/symbol-sdk-typescript/esm/facade/SymbolFacade";
import { PrivateKey } from "@nemtus/symbol-sdk-typescript/esm/CryptoTypes";
import { KeyPair } from "@nemtus/symbol-sdk-typescript/esm/symbol/KeyPair";
import { Signature } from "@nemtus/symbol-sdk-typescript/esm/symbol/models";
import {
  Configuration,
  NetworkRoutesApi,
  TransactionGroupEnum,
  TransactionRoutesApi,
  TransactionStatusDTO,
  TransactionStatusRoutesApi,
} from "@nemtus/symbol-sdk-openapi-generator-typescript-axios";
import WebSocket from "ws";
import "dotenv/config";

// テストネットのノードを指定
const NODE_DOMAIN = "symbol-test.next-web-technology.com";

// 送信先アドレス ... 今回はFaucetアドレスに送り返すことに
const faucetAddressString = "TDMYLKCTEVPSRPTG4UXW47IQPCYNLW2OVWZMLGY";

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

  // トランザクションを送信するアカウント関連データを作成
  const signer1PrivateKeyString = process.env.SIGNER_1_PRIVATE_KEY!;
  const signer1PrivateKey = new PrivateKey(signer1PrivateKeyString);
  const signer1KeyPair = new KeyPair(signer1PrivateKey);
  const signer1PublicKeyString = signer1KeyPair.publicKey.toString();
  const signer1AddressString = facade.network
    .publicKeyToAddress(signer1KeyPair.publicKey)
    .toString();

  // deadlineの計算(2時間で設定しているが変更可能、ただし遠すぎるとエラーになる)
  const now = Date.now();
  const deadline = BigInt(now - epochAdjustment * 1000 + 2 * 60 * 60 * 1000);

  // トランザクションのデータ生成 ... (例) 1XYM = 1000000μXYMをSIGNER_1からFAUCETへ送る
  const transaction = facade.transactionFactory.create({
    type: "transfer_transaction",
    signerPublicKey: signer1PublicKeyString,
    deadline,
    recipientAddress: faucetAddressString,
    mosaics: [{ mosaicId: networkCurrencyMosaicId, amount: 1000000n }],
  });

  // 手数料設定 ... 送信先ノードの設定によるがノードのデフォルト設定値100なら基本的に足りないことはないと思う
  const feeMultiplier = 100;
  (transaction as any).fee.value = BigInt(
    (transaction as any).size * feeMultiplier
  );

  // 署名
  const signature = facade.signTransaction(signer1KeyPair, transaction);
  (transaction as any).signature = new Signature(signature.bytes);

  // 各ネットワーク固有のgenerationHashSeedを設定
  (transaction as any).network.generationHashSeed = facade.network;

  // トランザクションのハッシュを計算 ... トランザクションの承認状態を後でWebSocketで確認する時などに必要
  const hash = facade.hashTransaction(transaction);
  console.log(hash.toString());
  console.log(`https://testnet.symbol.fyi/transactions/${hash.toString()}`); // デバッグ時に確認しやすいよう、ブロックエクスプローラーの該当ページを表示しておく

  // トランザクション送信時にはこのデータを使う
  const transactionPayload = (
    facade.transactionFactory.constructor as any
  ).attachSignature(transaction, signature);

  // 1 confirmation以外の場合の設定
  const confirmationHeight = 6; // 6confで確認と見なす場合
  let transactionHeight = 0;
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

      // ターゲットアドレスのトランザクションが未承認状態になったのを監視
      const unconfirmedBody = `{"uid": "${res.uid}", "subscribe": "unconfirmedAdded/${signer1AddressString}"}`;
      console.log(unconfirmedBody);
      ws.send(unconfirmedBody);

      // ターゲットアドレスのトランザクションが承認されるの監視
      const confirmedBody = `{"uid": "${res.uid}", "subscribe": "confirmedAdded/${signer1AddressString}"}`;
      console.log(confirmedBody);
      ws.send(confirmedBody);

      // ターゲットアドレスのトランザクションがエラーになったのを監視
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

    // トランザクションが未承認になったときに発火
    if (
      res.topic === `unconfirmedAdded/${signer1AddressString}` &&
      res.data.meta.hash === hash.toString()
    ) {
      console.log("transaction unconfirmed");
    }

    // トランザクションが承認されたときに発火
    if (
      res.topic === `confirmedAdded/${signer1AddressString}` &&
      res.data.meta.hash === hash.toString()
    ) {
      console.log("transaction confirmed");
      transactionHeight = parseInt(res.data.meta.height);
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
      res.data.hash === hash.toString()
    ) {
      console.log(res.data.code);
      ws.close();
    } else {
      console.log(res);
    }

    // confirmationHeightブロック後に監視終了
    if (
      transactionHeight !== 0 &&
      transactionHeight + confirmationHeight - 1 <= blockHeight
    ) {
      console.log(
        `${confirmationHeight} blocks confirmed. transactionHeight is ${transactionHeight} blockHeight is ${blockHeight}.`
      );

      // トランザクションの状態を確認し監視終了
      try {
        const transactionStatusRoutesApi = new TransactionStatusRoutesApi(
          configuration
        );
        const transactionStatusDTO: TransactionStatusDTO = (
          await transactionStatusRoutesApi.getTransactionStatus({
            hash: hash.toString(),
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
        `wait for ${confirmationHeight} blocks. transactionHeight is ${transactionHeight} blockHeight is ${blockHeight}.`
      );
    }

    // finalizedBlockHeightが対象ブロックを追い越した後に監視終了
    if (transactionHeight !== 0 && transactionHeight <= finalizedBlockHeight) {
      console.log(
        `${finalizedBlockHeight} block finalized. transactionHeight is ${transactionHeight} blockHeight is ${blockHeight}.`
      );

      // トランザクションの状態を確認し監視終了
      try {
        const transactionStatusRoutesApi = new TransactionStatusRoutesApi(
          configuration
        );
        const transactionStatusDTO: TransactionStatusDTO = (
          await transactionStatusRoutesApi.getTransactionStatus({
            hash: hash.toString(),
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
        `wait for finalized block. transactionHeight is ${transactionHeight} blockHeight is ${blockHeight}.`
      );
    }
  });

  // トランザクションのアナウンス実行
  try {
    const transactionRoutesApi = new TransactionRoutesApi(configuration);
    console.log(transactionPayload);
    const response = await transactionRoutesApi.announceTransaction({
      transactionPayload,
    });
    console.log(response.data);
  } catch (err) {
    console.error(err);
  }
})();

```

このようにトランザクションを送信する際は、対象アカウントのWalletを開いておくと、トランザクションが送信されて一旦未承認状態になった時に「チーン」という音が鳴り、トランザクションが1confの承認状態になった時に「ピコーン」という音が鳴るでしょう。(個人的には開発が上手く行きWalletでトランザクションが飛び交う効果音がにぎやかに鳴り響くと楽しくなってきます。)サンプルコードにはコメントを比較的多めに書いておいたので、どこで何をしているかはある程度確認できるかなと思います。もし上手く行かないところがあったら、GitHubや本記事末尾で紹介しているDiscord等でお気軽にコメントください。

:::details 実行時のログの例

```shell
~/symbol-sdk-typescript-sample-1$ npx ts-node send-transfer-transaction.ts 

40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F
https://testnet.symbol.fyi/transactions/40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F
{"payload": "B00000000000000008B788D279A618F169687225327E5721D0D6A2738F10F53A82347310E6CCD4928AA6C8705E6219BCE50E76D1BD5D499567C97E2390C6398689DEFE60EB90A506ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D0680000000001985441C044000000000000F64B73920500000098D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B0000010000000000C8B6532DDB16843A40420F0000000000"}
{ message: 'packet 9 was pushed to the network via /transactions' }
connection open
uid : PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL
{"uid": "PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL", "subscribe": "unconfirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL", "subscribe": "confirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL", "subscribe": "status/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ"}
{"uid": "PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL", "subscribe": "block"}
{"uid": "PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL", "subscribe": "finalizedBlock"}
{ uid: 'PE56L7DT7UFEV4ZUZM6JEB373PJNCLUL' }
wait for 6 blocks. transactionHeight is 0 blockHeight is 0.
wait for finalized block. transactionHeight is 0 blockHeight is 0.
transaction unconfirmed
{
  topic: 'unconfirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: '08B788D279A618F169687225327E5721D0D6A2738F10F53A82347310E6CCD4928AA6C8705E6219BCE50E76D1BD5D499567C97E2390C6398689DEFE60EB90A506',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16724,
      maxFee: '17600',
      deadline: '23931866102',
      recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
      mosaics: [Array]
    },
    meta: {
      hash: '40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F',
      merkleComponentHash: '40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F',
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
      signature: '27C609DDBD9E552425B736A1222B32C2E1ABD9431E4B59FD68705460799F4A95E5B39625D7461A7F780DF1108E85B80D37EF966D34567CE1342C8D273DF27D05',
      signerPublicKey: 'CD96C6830906530C4BCD7C45453F5D55A5FA5BAF098CFC4E512B18CC763D9573',
      version: 1,
      network: 152,
      type: 33091,
      height: '653699',
      timestamp: '23924697359',
      difficulty: '10000000000000',
      proofGamma: '14987781C5471F99EE0A168007E8F1E0AF204FC351057178071B73D9923CD4B3',
      proofVerificationHash: '5A494A1E9335148507B7C58162249217',
      proofScalar: 'A086226451E081BABC25619348E2FC5026F2688C0742D375B66F73BF580B6E02',
      previousBlockHash: 'C6DF5E1406BF16CB0FBD421AE524016E2FD16DFF1AF56D72F2AF0B871A908E21',
      transactionsHash: '40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F',
      receiptsHash: '0E12701813430073FDFA2E524C5AA182950DB5C2931FB0A015D260D6A54939C3',
      stateHash: '978B0BCAC574548EF5C28F15F6791ACAAE40A16A326C9C236D041335981E5B1D',
      beneficiaryAddress: '980F30859E6AB05706FD93188B39A42B42A44AC8DA115053',
      feeMultiplier: 100
    },
    meta: {
      hash: '5B09B4DC5B58B277FB74745FF5C9483467BE8ECB10FF61384184B5F82EE9C666',
      generationHash: 'F8084B2C740931EF6C84430E33B30248E90458AD4EE266111BE797E7AD1C5329'
    }
  }
}
wait for 6 blocks. transactionHeight is 0 blockHeight is 653699.
wait for finalized block. transactionHeight is 0 blockHeight is 653699.
transaction confirmed
{
  topic: 'confirmedAdded/TBFVGBN5XKVFWF3PKWRQRPH6SHSOTXJMXKYSTEQ',
  data: {
    transaction: {
      signature: '08B788D279A618F169687225327E5721D0D6A2738F10F53A82347310E6CCD4928AA6C8705E6219BCE50E76D1BD5D499567C97E2390C6398689DEFE60EB90A506',
      signerPublicKey: 'ECF3FF68E017A83528A0A361F1F1EE91D761B5E34008AD9474870D54F5C4D068',
      version: 1,
      network: 152,
      type: 16724,
      maxFee: '17600',
      deadline: '23931866102',
      recipientAddress: '98D985A853255F28BE66E52F6E7D1078B0D5DB4EADB2C59B',
      mosaics: [Array]
    },
    meta: {
      hash: '40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F',
      merkleComponentHash: '40EED5433899E2EB1FB33A864CCEDB9D7F5F42A59561F1B4341DE3751030B85F',
      height: '653699'
    }
  }
}
wait for 6 blocks. transactionHeight is 653699 blockHeight is 653699.
wait for finalized block. transactionHeight is 653699 blockHeight is 653699.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '300C88C9BDDBF04C4F81A80917FE058CFC9458B84E08D509ADF243693D94D27E0098DDC3FB8BA226435ACA4DE9BE95FDC692B0C683F95EF5883CA69193A44906',
      signerPublicKey: 'BF2EAFD7C2B1E84C814B797332CA10E82CC3E3C1E7BC8ACC4640E7FD33C90A2C',
      version: 1,
      network: 152,
      type: 33091,
      height: '653700',
      timestamp: '23924741192',
      difficulty: '10000000000000',
      proofGamma: '0953D5657183B9F500871FDF9BB83EDCDBF5D9B931BF32D1783FD9BCDC38F1A2',
      proofVerificationHash: 'A5192E49F1E794737D3C831D5585B401',
      proofScalar: 'DF1929D97CD80E0F0C085524840F0009FBCAF1683AE86B77E55C03DD2561FA0F',
      previousBlockHash: '5B09B4DC5B58B277FB74745FF5C9483467BE8ECB10FF61384184B5F82EE9C666',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '55917278DD7CA2D0BAEF12D5DD072CC929A2187AAC78E239E043A1F4198AC5DF',
      stateHash: '2FB6BE644C3CC1186CD737C39E14EDB24D90DBEEF87DF779136F88AA2C9003D7',
      beneficiaryAddress: '985F693ED8D58BC10F3E428E600E17F704BE042B03198B2A',
      feeMultiplier: 0
    },
    meta: {
      hash: 'D36D50647F1F41EA75FAB503F8F0795F5762C2198268EC9FB4F4013D577F7815',
      generationHash: 'D67B01773A8B6BC9E2FF7BBB0423DB4D8C843478B4556391062512687CC42DCE'
    }
  }
}
wait for 6 blocks. transactionHeight is 653699 blockHeight is 653700.
wait for finalized block. transactionHeight is 653699 blockHeight is 653700.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '64B0A8BAFA35F1D2A7F8D00475D4ACB8BA3AC546CB3445BA190A123A706CCADFC89B102396BABDA49803298B987206F49C97164DACBEDAFE63DA430954923800',
      signerPublicKey: '21CF6E21A73CB4CEBD32BA0AE2C7059D19B1A7827DF6C8D10FF33C3F1A2BF0E8',
      version: 1,
      network: 152,
      type: 33091,
      height: '653701',
      timestamp: '23924771171',
      difficulty: '10000000000000',
      proofGamma: '414F99CCF2115DC10CF761D3D4E335884ADCA8A74D9479FD67EAF38D61915D12',
      proofVerificationHash: '523DF3CEE73CA1F1684EF22599BF8B58',
      proofScalar: 'FE12167FE39A1EEA8FFD7B18D35F847B1D42BE696EA16534D9F669A0910A270C',
      previousBlockHash: 'D36D50647F1F41EA75FAB503F8F0795F5762C2198268EC9FB4F4013D577F7815',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'BFC94F3C268EA1D531DEE62CCF388CCB85746D368DAC775171A904B5897C0566',
      stateHash: 'B199DE8B7A60975EE9549075FF821F83289196564FF0A52C5354BDEC138D5DD2',
      beneficiaryAddress: '98FFA418508A3B022EF3491FC8254DEB7EC3EBA65B52DE1D',
      feeMultiplier: 0
    },
    meta: {
      hash: '0D5F10306D3F7E63158E1581C96EB1D636BD7787FACE35CADCD1D8A8E2AA7D1D',
      generationHash: 'F8210DAC0163CB037F3ABC73E59FE86C070DDC827E5922AAB4208FCE7666F824'
    }
  }
}
wait for 6 blocks. transactionHeight is 653699 blockHeight is 653701.
wait for finalized block. transactionHeight is 653699 blockHeight is 653701.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '12118E9565A88F932FE89D3595E01EE991CB4A803F375D9A1CCEF0D667C9C180F4F1DB8A6438C3626BF87E9278FA0601B88EF03DA807DE9C4220BA92A926D002',
      signerPublicKey: 'BF2EAFD7C2B1E84C814B797332CA10E82CC3E3C1E7BC8ACC4640E7FD33C90A2C',
      version: 1,
      network: 152,
      type: 33091,
      height: '653702',
      timestamp: '23924792232',
      difficulty: '10000000000000',
      proofGamma: 'B6B4EAFCAC77DFBFE42BEC7BC41E9B1D65C0A27EEE97E94F55AB08E9FF9910D9',
      proofVerificationHash: 'A8A8244A0EB6AC01E18A228DC44865E7',
      proofScalar: '67147B2A8C4B73F8CF31E98CE48DB95A76CF0DD9C24F3AF23094A86D5CCE930D',
      previousBlockHash: '0D5F10306D3F7E63158E1581C96EB1D636BD7787FACE35CADCD1D8A8E2AA7D1D',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '55917278DD7CA2D0BAEF12D5DD072CC929A2187AAC78E239E043A1F4198AC5DF',
      stateHash: 'DCE2463E6D9D10F758F1904718539329890DE030447C37511910C50DF1D34A34',
      beneficiaryAddress: '985F693ED8D58BC10F3E428E600E17F704BE042B03198B2A',
      feeMultiplier: 0
    },
    meta: {
      hash: 'CC4DF75939F1ED540890EA97688055CC686AA9B2D1FA1998A82B05FF094E35FA',
      generationHash: 'FDB798D4ACBCC9B60F4A35BCD9F5D80152909D51B5FF9A7236DD7BF0325D1723'
    }
  }
}
wait for 6 blocks. transactionHeight is 653699 blockHeight is 653702.
wait for finalized block. transactionHeight is 653699 blockHeight is 653702.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '48011E2F4DE712B135B59FE782375730882B01D8D66B5A01BAB0B89D05E66673EDEF24E25B609835E54D1D3957E942C1BF620988DF231F44AEA0810557433A08',
      signerPublicKey: 'F9A5A66EBE9AD5EAAAF07AF208BF33FB0B24C2049EFF99B65AD1BB46809852DE',
      version: 1,
      network: 152,
      type: 33091,
      height: '653703',
      timestamp: '23924824521',
      difficulty: '10000000000000',
      proofGamma: 'D0E1CDB0F05EF355CB4C3150233A6F69063F0F1641DAA48A2710136909857B01',
      proofVerificationHash: 'A5755D01A57F5CE9305FFF84DAEAFB5B',
      proofScalar: 'F4167D2CAC137C7E8B50F0C3786E012F191CB7CEFF6E85A9999F0F9B2432320C',
      previousBlockHash: 'CC4DF75939F1ED540890EA97688055CC686AA9B2D1FA1998A82B05FF094E35FA',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: 'F426FEC961E15E6F1A76117DA8363DE4064391816B024B5134F6BA19DCAF5DC0',
      stateHash: 'DF1C0EEEAACA2E966B3ECA891234482315C7057C1628E5C0CBECBD91D0ABCEF7',
      beneficiaryAddress: '98D3970759CB13CD9E6BD7FBEBD7F909F34ADA4E079B1A4D',
      feeMultiplier: 0
    },
    meta: {
      hash: '661392D9004B85260818CF03AA16AF05F987EB7ED700DB37D574059E258BB3CE',
      generationHash: 'F473AEDD8F5183A4C497ADA2C4F7798B73C06BF7DAB6D6E5C19628A267913F9A'
    }
  }
}
wait for 6 blocks. transactionHeight is 653699 blockHeight is 653703.
wait for finalized block. transactionHeight is 653699 blockHeight is 653703.
block
{
  topic: 'block',
  data: {
    block: {
      signature: '43417AF8B7CEDC9D359EF4B17745523968EF0287F835C8D5A758E919B53970B11FC2894E1953185E577FA6B442982FE051719EBAE8B9A2771FC36DACC213940E',
      signerPublicKey: '88A7993C2478345E3B86A98A2D8AC47739F7A95620238018958E3961C153E263',
      version: 1,
      network: 152,
      type: 33091,
      height: '653704',
      timestamp: '23924861796',
      difficulty: '10000000000000',
      proofGamma: '60F4CA9CD7AFBD3F5BE8ED42A40B5CE3A90F2EAD4969F0F719A12516F24D67D6',
      proofVerificationHash: '525BB53255B1A900B29D85E3564E56F3',
      proofScalar: '8214A6B61FD2BD2FE53F465DAEC6E3B2540B62EBACF083E92D6A323D78B03A03',
      previousBlockHash: '661392D9004B85260818CF03AA16AF05F987EB7ED700DB37D574059E258BB3CE',
      transactionsHash: '0000000000000000000000000000000000000000000000000000000000000000',
      receiptsHash: '987E011A37AC36B4D5524D846515B7580B33B41360083578BD721C85D07AD753',
      stateHash: 'A7E06767E5BDD277A82F472000E18038E5BAEDACB87CCE28F186BAED438BA0CE',
      beneficiaryAddress: '98258E9856A612C5346D3C0B318DCB654257BD9DD86E6481',
      feeMultiplier: 0
    },
    meta: {
      hash: '2DB314AD386B94F413F627A0370D175E846A572CD3D62DBF92175166437AC3ED',
      generationHash: 'F49E42593C78FCE7FBE9D211F3AAA4DED5E8EBCDE82F1613B54D41D93AD62A1E'
    }
  }
}
6 blocks confirmed. transactionHeight is 653699 blockHeight is 653704.
wait for finalized block. transactionHeight is 653699 blockHeight is 653704.
connection closed
```

:::

## まとめ

これで、TypeScript向けREST API clientと新公式SDKの(簡易的ではあるものの)TypeScript化されたnpmパッケージが揃ったので、新公式SDKの思想に沿ったツールを使ってTypeScript環境でのフロントエンド開発がだいぶ行いやすくなるのではないかと思います。

しかし、サンプルコードを見て頂くと、ところどころ、TypeScript的に意識のあまり高くない書き方で問題を回避しているところがあり、今の方法は簡易的なTypeScript化の限界があるかもしれません。とはいえ、JSDocでコメントを書いてあげることで改善できる部分もあるかもしれないので、そういう部分は新公式SDKの方に直接Pull Request送るような活動もできるといいなあと個人的には思っています。

Symbolのローンチ後、約1年半がたちましたが、まだまだ、Symbolブロックチェーンの開発関連のエコシステムがどのような展開を見せるのか未知数な部分が多いと感じています。今自分が作っている以下のSDK群が今後エコシステムの中でどのような立ち位置となるのかも正直未知数です。(もっと良い別のSDKが出てくるかもしれませんし、それはとても嬉しいことです。)

- [@nemtus/symbol-sdk-openapi-generator-typescript-axios](https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-axios)
- [@nemtus/symbol-sdk-openapi-generator-typescript-fetch](https://github.com/nemtus/symbol-sdk-openapi-generator-typescript-fetch)
- [@nemtus/symbol-sdk-typescript](https://github.com/nemtus/symbol)

未来のことはわかりませんが、この活動が、NEM/Symbolブロックチェーンのエコシステムの発展に少しでも寄与していたら嬉しいです。今後ともよろしくお願いします。

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

簡易的ではあるものの、既存のSDKをベースに短期間でTypeScript化に第一歩を踏み出せたのは、開発の方針で迷った際、以下Discordにて、様々な相談に乗って頂けたコミュニティの有志の開発者の方々のおかげです。この場を借りてお礼申し上げます。そして、NEM/Symbol関連の開発に興味のある方は、ぜひDiscordにご参加ください。

- [Discord of nem Japan UserGroup](https://discord.gg/H85N3C8eZW)

NEMTUSにて開発している内容について、もしご興味ある方は、以下のNEMTUSのDiscordにてGitHubでの動きを流しているチャンネルもあるので、ぜひお気軽にご参加ください。

- [Discord of NEMTUS](https://discord.gg/5qKH2HWxe8)
