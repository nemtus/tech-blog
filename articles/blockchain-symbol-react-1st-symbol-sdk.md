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

Reactでは

```sh
npx create-react-app <ディレクトリ名>
```

このコマンドが打てれば誰でも作れるようになります

## 初回起動

## Angular Material導入

## Angular Flex-Layout導入

## Webアプリ全体のレイアウト実装

## ホームページの実装

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