---
title: "Symbol x Angular その1 Angularでsymbol-sdkを使うための環境構築"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blockchain", "symbol", "angular", "typescript", "rxjs"]
published: false
---

# Angularでsymbol-sdkを使うための環境構築

## 概要

この記事では、Angularで作成されたWebアプリ上でsymbol-sdkというnpmパッケージを用いてSymbolというブロックチェーンを利用するための第一歩となる環境構築について説明します。Angularのデフォルトの設定のままsymbol-sdkをインストールするといくつかのエラーが発生します。それを修正するための方法について説明し、同時にsymbol-sdkをAngularプロジェクトで使う際のサンプルコードを紹介します。

以下URLでサンプルコードを実際に動かしているので、必要に応じてご参照ください。

- サンプルページ(ホームページ) [https://symbol-sample-angular.nemtus.com](https://symbol-sample-angular.nemtus.com)
- サンプルページ(アカウント情報表示ページ) [https://symbol-sample-angular.nemtus.com/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](https://symbol-sample-angular.nemtus.com/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA) ... URL末尾のアドレスを変更頂くことで、任意のアドレスの情報を表示できます。
- サンプルページのGitHubレポジトリ [https://github.com/nemtus/symbol-sample-angular](https://github.com/nemtus/symbol-sample-angular)

## 要約

- Angularプロジェクト(Angular CLI バージョン12系)では、symbol-sdkをインストールしただけでは以下のようなエラーが発生します。

1. コンパイル時に`Error: Module not found: Error: Can't resolve 'crypto'`のエラーが発生
2. コンパイル時に`Error: Module not found: Error: Can't resolve 'stream'`のエラーが発生
3. 実行時に`Uncaught ReferenceError: global is not defined`のエラーが発生

- それぞれ以下の方法で解決できます。

1. [https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-crypto](https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-crypto)
2. [https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-stream](https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-stream)
3. [https://dev.classmethod.jp/articles/angular6-referenceerror/](https://dev.classmethod.jp/articles/angular6-referenceerror/)

Angular, Symbolに既に慣れている方はここまでの情報で十分で、以下を読む必要は薄いかもしれませんが、Symbolブロックチェーンの説明や、htmlやJavaScriptやVueやReact等の一定レベルの経験がある方向けのAngularチュートリアル的な内容も含めたsymbol-sdkの使用方法のサンプル紹介等を以下に記載しています。
(書いている内に恐ろしい程の長文になってしまったのですが...)もしよろしければご参照の上、この機会に、Angularやrxjsの雰囲気や、Angularでsymbol-sdkをどのように使用するかのイメージをぜひ膨らませてみてください。

## 前提となる環境

- OS
  - Ubuntu 20.04 (on WSL2)
- Node.js
  - v14.17.4
- npm
  - 7.20.5
- Angular CLI
  - 12.2.2

## Symbolとは

Symbolとは、NEMブロックチェーンの次期バージョンとして、長い開発期間を経て、2021年3月17日にローンチされた、パブリックブロックチェーンです。

Symbolの前バージョンのNEMには以下のような特徴がありました。

- 利用しやすいAPI
- 開発者が独自にコントラクトを実装せずとも利用可能なようにブロックチェーン自体に実装済の豊富な機能

Symbolでは、前バージョンのNEMの良さを継承しつつ、以下のような大幅な機能追加と性能改善と設計の見直しが行われました。

- 分散された多数のノードが自律的に維持されるようなインセンティブ設計
- スケーラビリティ等の性能改善
- 複数のトランザクションをまとめて実行する機能の追加によるトークンのトラストレスな交換等のサポート
- ブロックチェーン上に、より柔軟にデータを刻むことが可能な機能の追加

詳細は以下の公式なドキュメントをご参照ください。

- Symbol公式ドキュメント [https://docs.symbolplatform.com/ja/index.html](https://docs.symbolplatform.com/ja/index.html)
- NEM公式ドキュメント [https://nem.io/](https://nem.io/)

## symbol-sdkとは

Symbolブロックチェーンでは、ノードが公開しているAPIを利用した開発を行うことになります。

API呼び出し等の処理を自前で実装しても良いのですが、以下のようにTypeScriptがサポートされたSDKがsymbol-sdkというnpmパッケージとして公式にリリースされているので、そちらを使用すると便利でしょう。

- symbol-sdk GitHubレポジトリ [https://github.com/symbol/symbol-sdk-typescript-javascript](https://github.com/symbol/symbol-sdk-typescript-javascript)
- symbol-sdk npmパッケージ [https://www.npmjs.com/package/symbol-sdk](https://www.npmjs.com/package/symbol-sdk)
- 公式ドキュメント symbol-sdk 使用方法ガイド アカウント情報取得 [https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html#method-01-using-the-sdk](https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html#method-01-using-the-sdk)

なお、symbol-sdkでは、後述するrxjsが積極的に使用されています。rxjsは使いこなすことができると非常に便利なのですが、学習コストは高めで、積極的な使用には賛否が分かれるところではあるかと思います。好みによってはPromiseに置き換えて使用するという戦略もあるかもしれませんので、案件に応じてご検討頂くとよろしいかと思います。

## rxjsとは

前述の通り、symbol-sdkの中では、rxjsが積極的に利用されています。

rxjsでは、非同期なデータを、川の流れのようなものをObservableという概念に見立てて、流れを生み出す機能や、流れをモニタリングして上流から流れを受け取って加工した上で下流に流す機能や、流れをモニタリングして何かが流れてきたら何らかの処理を行う機能等が提供されています。

Promise, Thenの仕組みが1回限りの非同期処理の完了を待って次の作業を行うのと似ていますが、rxjsでは1回限りではなく連続した流れを継続的に扱うための仕組みが提供されているとも言えるでしょう。

rxjsは多くの方に取って、それなりに学習コストが大きいものと思いますが、データの流れや、各種オペレーターの動作をイメージできるようになると、REST APIやWeb Socket等の非同期な通信処理をきわめて柔軟に効率よく記述することが可能となります。

しかし、rxjsのオフィシャルなドキュメントを初見で読むと、処理の種類があまりにも豊富にあることと、言葉で説明された処理の概念や内容がとても意味不明に感じられてつらいと思います。

rxjs習得における個人的なおすすめは、基礎的な概念(≒Observableという概念)やメジャーなオペレーター(map, mergeMap, combineLatest)に絞って、以下のマーブル図や簡易的な説明でイメージを固めつつ、イメージが固まったら必要に応じて公式サイト等のドキュメントも参考にしつつsymbol-sdkやAngularのようなrxjsを使うコードをちょっとだけ書いてみて動作を試すのが良いと思います。

- rxjs マーブル図 [https://rxmarbles.com/](https://rxmarbles.com/)
- rxjs 公式サイト [https://rxjs.dev/](https://rxjs.dev/)

## Angularとは

AngularはReact, Vueと同様に、JavaScript(、より厳密に言うとAngularの場合、TypeScript)を用いたSPA(Single Page Application)フレームワークです。

以下のように、Angularそれ自体が多くの機能を網羅していること、TypeScript, rxjsによるメリットが得られやすいといった点が特徴と言えるでしょう。

- Angularそれ自身で必要な機能がほとんどそろっているところ... 他のフレームワークで必要になりがちな、追加のツールとして何を選定するか？といった検討を行う必要がかなり少ないと思います。(最近は少しずつAngular公式やCLIデフォルト以外のツールの利用が増えてきていると思いますが...)
- TypeScript前提のSPAでありデフォルトでTypeScriptのメリットを享受できるところ ... 一度TypeScriptでの開発者体験の良さに慣れるとTypeScriptではないJavaScriptには戻れないと思います...
- rxjsの使用を前提としたAngularの機能があり、非同期な処理によって任意のタイミングに取得される情報の表示を柔軟にシンプルに実装できるところ

しかし、上記恩恵を受けるためには、Angular特有の機能, TypeScript, rxjsの概念や知識の習得が必要で、習得のための学習コストは高めかもしれません。

個人的に感じているAngular習得のコツとしては以下を感じていて、

- 早い段階から実際に非同期な通信をAngularのサービスやrxjsで扱ってみてrxjsの書き方に慣れること
- できるだけシンプルで小規模なアプリを作ってみること
- ルーティングに応じたコンポーネントのディレクトリ配置や、コンポーネント分割の粒度について、自分なりに納得できる形でルール化すること
- UIを整えるのにできるだけリソースを割かずに済むUIライブラリを最初から使っておくこと

それらを意識した形で、この記事のサンプルアプリ、サンプルレポジトリを公開しているので、Angularを用いたWeb開発のチュートリアル的なネタとしてもご活用くださいますと幸いです。

- Angular公式ドキュメント [https://angular.jp/](https://angular.jp/)

前置きがだいぶ長くなってしまいましたが、それでは本題の説明をはじめます。

## Angular CLIインストール

以下コマンドでグローバルにAngular CLIをインストールします。

```sh
npm install -g @angular/cli
```

インストールが終わったら`ng version`を実行し、バージョンを表示させて正常にインストールできているか確認しておきましょう。以下のようにバージョンが表示されていればAngular CLIのインストールはOKです。

```sh
$ ng version

     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/

Angular CLI: 12.2.2
Node: 14.17.4
Package Manager: npm 7.20.5
OS: linux x64

Angular: 
... 

Package                      Version
------------------------------------------------------
@angular-devkit/architect    0.1202.2 (cli-only)
@angular-devkit/core         12.2.2 (cli-only)
@angular-devkit/schematics   12.2.2 (cli-only)
@schematics/angular          12.2.2 (cli-only)
```

## Angularの新規プロジェクト作成

`ng new symbol-sample-angular`コマンドを実行すると、angularの雛型プロジェクトが、`symbol-sample-angular`ディレクトリ内に生成されます。routingとcssの設定を尋ねられるので、以下のような選択で進みましょう。

```sh
$ ng new symbol-sample-angular
? Would you like to add Angular routing? Yes
? Which stylesheet format would you like to use? CSS
CREATE symbol-sample-angular/README.md (1065 bytes)
CREATE symbol-sample-angular/.editorconfig (274 bytes)

~
中略
~

CREATE symbol-sample-angular/src/app/app.component.ts (225 bytes)
✔ Packages installed successfully.
    Successfully initialized git.
```

## 初回起動

生成された`symbol-sample-angular`ディレクトリに移動し、`ng serve`コマンドでローカルで開発用の環境でWebアプリを実行してみましょう。以下のように、`Compiled successfully.`と表示されたら、ブラウザで[http://localhost:4200](http://localhost:4200)にアクセスしてみて、AngularのWelcomeページが表示されていたら成功です。

```shell
$ cd symbol-sample-angular/
symbol-sample-angular$ ng serve
⠋ Generating browser application bundles (phase: setup)...Compiling @angular/core : es2015 as esm2015
Compiling @angular/common : es2015 as esm2015
Compiling @angular/platform-browser : es2015 as esm2015
Compiling @angular/router : es2015 as esm2015
Compiling @angular/platform-browser-dynamic : es2015 as esm2015
✔ Browser application bundle generation complete.

Initial Chunk Files | Names         |      Size
vendor.js           | vendor        |   2.39 MB
polyfills.js        | polyfills     | 128.52 kB
main.js             | main          |  56.83 kB
runtime.js          | runtime       |   6.64 kB
styles.css          | styles        | 736 bytes

                    | Initial Total |   2.58 MB

Build at: 2021-08-28T08:43:03.329Z - Hash: 1917621e6f02bb07c54e - Time: 7999ms

** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **

✔ Compiled successfully.
✔ Browser application bundle generation complete.

5 unchanged chunks

Build at: 2021-08-28T08:43:04.024Z - Hash: 001b0c4dd7c8c521bf7d - Time: 350ms

✔ Compiled successfully.
```

## Angular Material導入

できるだけ少ない記述量で無理なくリッチなUIに仕上げたいため、Angular Materialを導入します。
導入手順は以下URLを参考にします。

[https://material.angular.io/guide/getting-started](https://material.angular.io/guide/getting-started)

一旦`Ctrl` + `C`で`ng serve`を止めて、`ng add @angular/material`コマンドを実行します。いくつか確認を求められますが、テーマはカスタムを選択して自分好みに修正しやすいようにしておくと便利だと思います。それ以外はデフォルトで良いでしょう。

```sh
$ ng add @angular/material
ℹ Using package manager: npm
✔ Found compatible package version: @angular/material@12.2.3.
✔ Package information loaded.
 
The package @angular/material@12.2.3 will be installed and executed.
Would you like to proceed? Yes
✔ Package successfully installed.
? Choose a prebuilt theme name, or "custom" for a custom theme: Custom
? Set up global Angular Material typography styles? Yes
? Set up browser animations for Angular Material? Yes
UPDATE package.json (1149 bytes)
✔ Packages installed successfully.
CREATE src/custom-theme.scss (1544 bytes)
UPDATE src/app/app.module.ts (502 bytes)
UPDATE angular.json (3168 bytes)
UPDATE src/index.html (587 bytes)
UPDATE src/styles.css (181 bytes)
```

Angular Materialを利用した実装を行う際には、以下の公式サイトのコンポーネントカタログページが参考になるでしょう。

- Angular Material公式サイト [https://material.angular.io/components/categories](https://material.angular.io/components/categories)

## Angular Flex-Layout導入

できるだけ少ない記述で、PC～モバイルまで幅広い画面での表示に対応できるようにしたいため、Angular Flex-Layoutを導入します。
導入手順は以下URLを参考にします。

[https://github.com/angular/flex-layout#getting-started](https://github.com/angular/flex-layout#getting-started)

`npm i -s @angular/flex-layout @angular/cdk`コマンドを実行してパッケージをインストールした後、`src/app/app.module.ts`ファイルに以下のような内容を追記します。

```typescript:app.module.ts
import { FlexLayoutModule } from '@angular/flex-layout';
...

@NgModule({
    ...
    imports: [ FlexLayoutModule ],
    ...
});
```

Angular Flex-Layoutを利用した実装時には、以下のデモサイトを見ながら実装するとわかりやすいでしょう。

- Angular Flex-Layoutデモサイト [https://tburleson-layouts-demos.firebaseapp.com/#/docs](https://tburleson-layouts-demos.firebaseapp.com/#/docs)
- Angular Flex-Layout GitHubレポジトリ [https://github.com/angular/flex-layout](https://github.com/angular/flex-layout)

## Webアプリ全体のレイアウト実装

Angularとしての環境構築がある程度一旦終わったので、具体的な実装に進みます。
まずはアプリ全体のレイアウト関連の実装を行います。
最初にデフォルトで表示されている内容をすべて消してしまいましょう。
`src/app/app.component.html`ファイル内の記述を、`<router-outlet></router-outlet>`を除き、全て削除して以下のようにします。
この`<router-outlet></router-outlet>`の箇所にURLに応じた内容が表示されるようにこの後実装を加えていくことになります。

```html:app.component.html
<router-outlet></router-outlet>
```

`ng serve`して[http://localhost:4200](http://localhost:4200)にアクセスし、空白のページが表示されることを確認しておきましょう。

全体のレイアウトは、メニューバーやサイドナビをAngular Materialの以下のコンポーネントやMaterial Iconsを使用して実装しようと思います。

- Toolbar [https://material.angular.io/components/toolbar/overview](https://material.angular.io/components/toolbar/overview)
- Sidenav [https://material.angular.io/components/sidenav/overview](https://material.angular.io/components/sidenav/overview)
- Icon [https://material.angular.io/components/icon/overview](https://material.angular.io/components/icon/overview)
- Material icons [https://fonts.google.com/icons?selected=Material+Icons](https://fonts.google.com/icons?selected=Material+Icons)
- List [https://material.angular.io/components/list/overview](https://material.angular.io/components/list/overview)

実装の際、表示に必要なデータについては、`app.component.ts`内で定義して、`app.component.html`内で参照して表示するといったことも可能です。
また`<mat-toolbar>`等の通常のhtml要素と異なるAngular Materialのコンポーネントについては、`app.module.ts`内でimportして、importsの配列内に記述することで、使用可能になります。
何をimportすべきかを覚えるのは大変だと思うので、必要に応じてAngular Materialの公式サイトの対象コンポーネントのAPIタブの情報を参考にすると良いでしょう。

結果的に`src/app/app.component.html`, `src/app/app.component.ts`, `src/app/app.module.ts`を以下のように実装し、`ng serve`すると、メニューバーと、メニューボタンを押す度にサイドナビが開閉し、サイドナビにホームアイコンとともにHomeというホームページへのリンクが表示された状態のレイアウトに整えましょう。

`src/app/app.component.html`

```html:app.component.html
<mat-toolbar color="primary">
  <button mat-icon-button (click)="sidenav.toggle()">
    <mat-icon>menu</mat-icon>
  </button>
  <h1>{{ title }}</h1>
</mat-toolbar>
<mat-sidenav-container fxFlexFill>
  <mat-sidenav mode="side" opend #sidenav>
    <mat-nav-list>
      <ng-container *ngFor="let sideNavLink of sideNavLinks">
        <mat-list-item>
          <a routerLink="{{ sideNavLink.linkPath }}">
            <div fxLayout="row" fxLayoutAlign="start center">
              <mat-icon>{{ sideNavLink.icon }}</mat-icon>
              <span>{{ sideNavLink.name }}</span>
            </div>
          </a>
        </mat-list-item>
      </ng-container>
    </mat-nav-list>
  </mat-sidenav>
  <mat-sidenav-content>
    <router-outlet></router-outlet>
  </mat-sidenav-content>
</mat-sidenav-container>
```

`src/app/app.component.ts`

```typescript:app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  title = 'Symbol x Angular';
  sideNavLinks = [
    {
      name: 'Home',
      icon: 'home',
      linkPath: '/',
    },
  ];
}
```

`src/app/app.module.ts`

```typescript:app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { FlexLayoutModule } from '@angular/flex-layout';
import { MatToolbarModule } from '@angular/material/toolbar'; // 追加
import { MatIconModule } from '@angular/material/icon'; // 追加
import { MatSidenavModule } from '@angular/material/sidenav'; // 追加
import { MatListModule } from '@angular/material/list'; // 追加

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    FlexLayoutModule,
    MatToolbarModule, // 追加
    MatIconModule, // 追加
    MatSidenavModule, // 追加
    MatListModule, // 追加
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## ホームページの実装

全体的なレイアウトができたので、個別のページの実装に進みます。
まずは`/`のページ(= [http://localhost:4200/](http://localhost:4200/) )の実装を行います。
このページでは、このWebアプリの大まかな目的を説明する文言を入れてみましょう。

ここでは以下のようなコンポーネント分割ルールで作業を進めます。
`/`のページでは、`src/app/page/home`ディレクトリ内のコンポーネントを表示し、
`src/app/page/home`コンポーネント内に、`src/app/view/home`コンポーネントを表示するようなルールで作業を進めます。

このように、pageディレクトリ配下と、viewディレクトリ配下に、コンポーネントを分けた理由は、pageディレクトリ配下のコンポーネントではデータの取得や加工を行い、そのデータをviewディレクトリ配下の子コンポーネントに渡してviewディレクトリ側は表示に関わる処理を行うといった役割分担とすることで、各コンポーネント毎の役割が明確となり、シンプルで可読性の高い構造にすることを意図しています。

それでは実際にコンポーネントを作成していきましょう。

まずは、以下コマンドを実行し、ディレクトリ`src/app/page`, `src/app/view`を作成します。

```sh
$ cd src/app/
src/app$ mkdir page
src/app$ mkdir view
```

作成したディレクトリ`src/app/page`に移動し、`ng g component home`コマンドを実行すると、コンポーネントのファイルとして、`src/app/page/home/home.component.css`, `src/app/page/home/home.component.html`, `src/app/page/home/home.component.spec.ts`, `src/app/page/home/home.component.ts`の4ファイルが生成され、`src/app/app.module.ts`の中に、生成されたコンポーネントのクラスが定義されている`src/app/page/home/home.component.ts`がimportされ、declarationsの配列の中にコンポーネントが追加されていることがわかると思います。このようにコンポーネントのクラスを`src/app/app.module.ts`のdeclarationsの配列の中に追加することで、対象のコンポーネントを利用できるようになります。

```sh
src/app$ cd page/
src/app/page$ ng g component home

CREATE src/app/page/home/home.component.css (0 bytes)
CREATE src/app/page/home/home.component.html (19 bytes)
CREATE src/app/page/home/home.component.spec.ts (612 bytes)
CREATE src/app/page/home/home.component.ts (267 bytes)
UPDATE src/app/app.module.ts (979 bytes)
```

`src/app/app.module.ts`

```typescript:app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { FlexLayoutModule } from '@angular/flex-layout';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatIconModule } from '@angular/material/icon';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';
import { HomeComponent } from './page/home/home.component'; // この行追加されている

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent // この行追加されている
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    FlexLayoutModule,
    MatToolbarModule,
    MatIconModule,
    MatSidenavModule,
    MatListModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

次に、`src/app/view/home`ディレクトリに、先ほど作成したコンポーネントの子コンポーネントとして使うコンポーネントを作成していきます。
`src/app/view`ディレクトリに移動し、`ng g component home`コンポーネントを実行し、子コンポーネントを作成します。

```sh
src/app/page$ cd ../view/
src/app/view$ ng g component home

CREATE src/app/view/home/home.component.css (0 bytes)
CREATE src/app/view/home/home.component.html (19 bytes)
CREATE src/app/view/home/home.component.spec.ts (612 bytes)
CREATE src/app/view/home/home.component.ts (267 bytes)
```

コンポーネントの名前が親コンポーネントと重複しないように、作成されたコンポーネントの中身を修正して、`src/app/app.module.ts`に登録していきます。

`src/app/view/home/home.component.ts`

```typescript:home.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-view-home', // app-home -> app-view-homeに修正
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class ViewHomeComponent implements OnInit { // HomeComponent -> ViewHomeComponent に修正

  constructor() { }

  ngOnInit(): void {
  }

}
```

`src/app/view/home/home.component.spec.ts`

```typescript:home.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { ViewHomeComponent } from './home.component'; // HomeComponent -> ViewHomeComponent

describe('ViewHomeComponent', () => { // HomeComponent -> ViewHomeComponent
  let component: ViewHomeComponent; // HomeComponent -> ViewHomeComponent
  let fixture: ComponentFixture<ViewHomeComponent>; // HomeComponent -> ViewHomeComponent

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ ViewHomeComponent ] // HomeComponent -> ViewHomeComponent
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(ViewHomeComponent); // HomeComponent -> ViewHomeComponent
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

`src/app/view/home/home.component.html`

```html:
<p>view-home works!</p> <!-- home -> view-home -->
```

`src/app/app.module.ts`

```typescript:app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { FlexLayoutModule } from '@angular/flex-layout';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatIconModule } from '@angular/material/icon';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';
import { HomeComponent } from './page/home/home.component';
import { ViewHomeComponent } from './view/home/home.component'; // 追加

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    ViewHomeComponent // 追加
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    FlexLayoutModule,
    MatToolbarModule,
    MatIconModule,
    MatSidenavModule,
    MatListModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

次に親コンポーネント`src/app/page/home/home.component.html`内に子コンポーネント`src/app/view/home/home.component.html`を表示できるよう追加します。

```html:home.component.html
<p>home works!</p>
<app-view-home></app-view-home>
```

これでコンポーネントの親子関係が実装できたので、URLの`/`のページ(=ホームページ)にアクセスした時に、親コンポーネントの`src/app/page/home/home.component.html`が表示されるよう、`src/app/app-routing.module.ts`にルーティングの実装を追記します。

`src/app/app-routing.module.ts`

```typescript:app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

ここまで実装し、`ng serve`を実行し、[http://localhost:4200/](http://localhost:4200/)にアクセスすると、ブラウザ上に、

home works!

view-home works!

と表示されており、親コンポーネントの中に子コンポーネントが埋め込まれて表示できていることが確認できると思います。

コンポーネントの相互関係、ルーティングの実装ができたので、実際に表示する内容を実装していきます。

親コンポーネントの`src/app/page/home/home.component.html`ファイル, 子コンポーネントの`src/app/view/home/home.component.html`を以下のように実装します。

`src/app/page/home/home.component.html`

```html:home.component.html
<app-view-home></app-view-home>
```

Angular Materialの以下コンポーネントを利用して実装します。

- Card [https://material.angular.io/components/card/examples](https://material.angular.io/components/card/examples)

`src/app/view/home/home.component.html`

```html:home.component.html
<mat-card>
  <mat-card-header>
    <h2>Home</h2>
  </mat-card-header>
  <mat-card-content>
    <p>Angularでのsymbol-sdkの使い方を説明するためのアプリです。</p>
  </mat-card-content>
</mat-card>
```

この記事ではホーム画面の実装はここまでとします。

## アカウント情報表示ページの実装

ここまででだいぶAngular, Angular Material, Angular Flex-Layout等を用いた開発の雰囲気がつかめてきているのではないかと思うので、次に実際にsymbol-sdkを用いて、メインネットのアカウント情報を取得して表示するページの実装にチャレンジしてみましょう！

`/explorer/accounts/{address}`のURLにアクセスすると、`{address}`の箇所に指定したアドレスのメインネットのアカウント情報を表示するようなページの実装を行います。

まず最初にコンポーネントを作成しましょう。詳細な手順は省きますが、HomeComponentとViewComponentを作成した時と同様に、URLのパスと同様の構造のディレクトリを作成し、そこにAccountComponentとViewAccountComponentを作成してコンポーネントの親子関係を構成します。

AccountComponentの作成

1. `src/app/page/explorer/accounts`ディレクトリの作成
1. `src/app/page/explorer/accounts`ディレクトリへ移動
1. `ng g component accounts`コマンドを実行するとAccountComponent関連4ファイルが作成され、`src/app/app.module.ts`へのコンポーネントの登録が実行される

ViewAccountComponentの作成

1. `src/app/view/explorer/accounts`ディレクトリの作成
1. `src/app/view/explorer/accounts`ディレクトリへ移動
1. `ng g component accounts`コマンドを実行するとViewAccountComponent関連4ファイルが作成されるが、ファイル内のコンポーネント名を`AccountComponent` -> `ViewAccountComponent`に変更し、selectorの箇所を`app-account` -> `app-view-account`に変更し、htmlファイル内の表示を`account works!` -> `view-account works!`に変更し、`src/app/app.module.ts`のdeclarationsの配列内に`ViewAccountComponent`を追加してコンポーネントを登録する。

親コンポーネントAccountComponentに、ViewAccountComponentを子コンポーネントとして表示

1. `src/app/page/explorer/accounts/account/account.component.html`にViewAccountComponentのselectorのタグを追記

ここまででやるとそれぞれのファイルが以下のような実装になったと思います。

`src/app/page/explorer/accounts/account/account.component.ts`

```typescript:account.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-account',
  templateUrl: './account.component.html',
  styleUrls: ['./account.component.css']
})
export class AccountComponent implements OnInit {

  constructor() { }

  ngOnInit(): void {
  }

}
```

`src/app/page/explorer/accounts/account/account.component.html`

```html:account.component.html
<p>account works!</p>
<app-view-account></app-view-account> <!-- 追加 -->
```

`src/app/page/explorer/accounts/account/account.component.spec.ts`

```typescript:account.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { AccountComponent } from './account.component';

describe('AccountComponent', () => {
  let component: AccountComponent;
  let fixture: ComponentFixture<AccountComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ AccountComponent ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(AccountComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});

```

`src/app/view/explorer/accounts/account/account.component.ts`

```typescript:account.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-view-account', // view-account -> app-view-account
  templateUrl: './account.component.html',
  styleUrls: ['./account.component.css']
})
export class ViewAccountComponent implements OnInit { // AccountComponent -> ViewAccountComponent

  constructor() { }

  ngOnInit(): void {
  }

}
```

`src/app/view/explorer/accounts/account/account.component.html`

```html:account.component.html
<p>view-account works!</p> <!-- account -> view-account -->
```

`src/app/view/explorer/accounts/account/account.component.spec.ts`

```typescript:account.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { ViewAccountComponent } from './account.component';

describe('ViewAccountComponent', () => { // AccountComponent -> ViewAccountComponent
  let component: ViewAccountComponent; // AccountComponent -> ViewAccountComponent
  let fixture: ComponentFixture<ViewAccountComponent>; // AccountComponent -> ViewAccountComponent

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ ViewAccountComponent ] // AccountComponent -> ViewAccountComponent
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(ViewAccountComponent); // AccountComponent -> ViewAccountComponent
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

`src/app/app.module.ts`

```typescript:app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { FlexLayoutModule } from '@angular/flex-layout';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatIconModule } from '@angular/material/icon';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';
import { MatCardModule } from '@angular/material/card';

import { HomeComponent } from './page/home/home.component';
import { ViewHomeComponent } from './view/home/home.component';
import { AccountComponent } from './page/explorer/accounts/account/account.component'; // 自動的に追加
import { ViewAccountComponent } from './view/explorer/accounts/account/account.component'; // 追加

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    ViewHomeComponent,
    AccountComponent,  // 自動的に追加
    ViewAccountComponent // 追加
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    FlexLayoutModule,
    MatToolbarModule,
    MatIconModule,
    MatSidenavModule,
    MatListModule,
    MatCardModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

ここまでできたら、親コンポーネントのAccountComponentをルーティングの設定ファイルに追記します。

`src/app/app-routing.module.ts`

```typescript:app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AccountComponent } from './page/explorer/accounts/account/account.component'; // 追加
import { HomeComponent } from './page/home/home.component';

const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  },
  { // 追加
    path: 'explorer/accounts/:address', // 追加
    component: AccountComponent, // 追加
  }, // 追加
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

これで、`ng serve`して、[http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA)にアクセスしてみると、ブラウザに以下のように表示される状態になりました。

account works!

view-account works!

いよいよこのページにアカウント情報を表示していく実装をしていきましょう。

まずURL内で動的に指定されるアドレスの情報をコンポーネントで利用する実装を加えましょう。

AngularではURL内の動的なパラメーターを、ルーティングの設定ファイル内であらかじめ`:address`のように設定しておくことで、実際に表示されたURL内のパラメーターをObservableな値としてコンポーネントから利用する仕組みが標準で提供されています。

`src/app/page/explorer/accounts/account/account.component.ts`ファイルを以下のように実装しましょう。

```typescript:
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Component({
  selector: 'app-account',
  templateUrl: './account.component.html',
  styleUrls: ['./account.component.css']
})
export class AccountComponent implements OnInit {
  address$?: Observable<string>;
  address?: string;

  constructor(private route: ActivatedRoute) { }

  ngOnInit(): void {
    this.address$ = this.route.params.pipe(map((params) => params.address));
    this.address$.subscribe(
      (address) => {
        console.log(address);
        this.address = address;
      }
    );
  }

}
```

`ng serve`して[http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA)をブラウザで開き、開発者ツールのコンソールを確認すると、URLで指定したアドレスが表示されていると思います。

表示されているページのURLの情報を、Angular標準のActivatedRouteというサービスからObservableなデータとして取得し、URL内のパスパラメーターの中のaddressの値が取得でき次第、ブラウザの開発者ツールのコンソールに表示するという内容になっています。

このファイルの実装は、Angularやrxjsでの特徴がかなり入っています。
具体的には以下のような内容です。

- サービス(≒状態と処理を持つもの)をインポートして、constructorの引数内に注入することで、以降の処理では、あたかもコンポーネント内で定義されたものであるかのように、別のサービスをコンポーネント内でthisで呼び出して利用することができていることがわかると思います。
- 非同期に取得されるデータをObservableでラップされた型の変数として定義しておき、サービスが返してくれるObservableでラップされた型をそのまま代入しておくことで、いつデータが流れてくるかわからないような非同期でリアクティブな通信も柔軟に扱えるようにできます。
- Observableな値に何らかの変換を加えて別のObservableな値にしたい場合、pipeで受け取って、pipeの中でmap等のrxjsオペレーターを順に作用させて別のObservableな値に加工することができます。
- Observableな変数には末尾に$をつけておき、その変数がObservableであることをわかりやすくしておくことが慣例となっています。
- Observableなデータが流れてきたら、subscribeでデータの流れをキャッチしてその都度Observableではない変数に値そのものを代入しておくことで、最新の値自体を常に取得しておくこともできます。Observableな値をそのまま使うか、subscribeして値にしておくかは好みもあると思いますし、ケースバイケースでどちらの方が便利というのは一概に言いにくいですが、AngularではObservableなデータをそのままhtml上に表示する方法(asyncパイプ ... 後述)が標準で用意されているので、Observableな値をそのままアプリ内で引き回すのも便利です。

このあたりがAngularの便利なところでもあり、同時に難しいと感じるところでもあると思います。ここでは一旦そういうもの？くらいのイメージを持ってもらえるとありがたいです。

アドレスの情報をURLから取得できるようになったので、アドレスの情報を使って実際にノードのAPIと通信を行い、アカウント情報を取得する処理を実装してみましょう。

以下では記事作者が個人的に運用しているノードを使用した実装を説明します。

- http [http://symbol-sakura-16.next-web-technology.com:3000](http://symbol-sakura-16.next-web-technology.com:3000)
- https [https://symbol-sakura-16.next-web-technology.com:3001](http://symbol-sakura-16.next-web-technology.com:3000)

:::message
使用するノードはこの記事では1ノード固定で説明を進めますが、実装次第では、Symbolブロックチェーン上で維持されている1000台以上のノードを自由に使えることも覚えておくと良いでしょう。
その場合、以下の公式ノードリストから、httpsでも使用可能なノードを選んで使うと良いでしょう。

[https://symbolnodes.org/nodes/](https://symbolnodes.org/nodes/)
:::

実装をはじめる前にノード関連で注意すべき点があります。
ローカルでの開発では、httpsなノードアドレスは使えないためhttpなノードアドレスを使う必要がありますが、実際にデプロイした場合、デプロイ先のWebアプリがhttpsの場合、ノードアドレスもhttpsを使う必要があるということです。
そのため、ローカルでの開発時にはhttpなノードアドレスを、デプロイ先の本番環境ではhttpsなノードアドレスを使う必要があります。

この違いはAngularの環境変数に設定して、コンポーネントやサービスでそれを参照する形で解決しておくことにします。(本格的な開発では、環境に応じて環境変数でフラグだけたてて、フラグに応じたノードの一覧を取得して、それを適宜利用するようなサービスを作ってそれを使用できるとより良いと思います。)

`src/environments/environment.ts`

```typescript:environment.ts
export const environment = {
  production: true,
  nodeUrl: 'http://symbol-sakura-16.next-web-technology.com:3000',
};
```

`src/environments/environment.prod.ts`

```typescript:environment.prod.ts
export const environment = {
  production: true,
  nodeUrl: 'https://symbol-sakura-16.next-web-technology.com:3001',
};
```

:::message alert
なお、Angularのようなフロントエンドで設定した環境変数は、実際にWebアプリとして公開されると誰もがアクセスできてしまうため、秘密鍵やアカウント情報のような公開してはならない情報をフロントエンドの環境変数に設定するのは絶対にNGであることは覚えておいてください。
:::

:::message
SymbolブロックチェーンのノードはAPIが直接公開されること前提で実装されているため、アプリでどのノードを使っているかという情報がフロントエンドに露出していても問題ないでしょう。
:::

symbol-sdkを使ってノードからアカウント情報を取得するサービスを実装していきます。

まずsymbol-sdkとrxjsをインストールします。Angularの場合、rxjsは既にインストールされていますが、以下手順通り`npm install symbol-sdk rxjs -S`を実行すると良いでしょう。

symbol-sdk GitHubレポジトリ インストール手順 [https://github.com/symbol/symbol-sdk-typescript-javascript#installation]

次にサービスを作成します。

サービス作成時に意識したほうが良い点として、自分が作成しているWebアプリがほしいデータ構造と、symbol-sdkのデータ構造の間のギャップを埋める処理をできるだけ一か所にまとめておき、symbol-sdkの今後の変更にアプリ側で余分な対応をする必要がないような構造にすることが大事です。

Symbolのローンチまでの(長～い)開発期間中、symbol-sdkは度重なる破壊的変更が加えられてきた経緯や、今後別のより軽量化されたsdkの開発等も検討されているらしいことから、今後、破壊的変更が加えられても、できるだけシンプルに変更に追従できるようにしておくことが重要だと思われるからです。

具体的には、アプリのメインロジック(≒Angularのサービス)はsymbol-sdkに依存しない形で実装し、アプリが必要とするデータを、アプリ側で型やinterfaceとして定めて、それを満たすような形でsymbol-sdkを用いてデータをアプリのメインロジックに渡すような実装にしておくことが望ましいと思います。

今実装しようとしている内容は、アカウント情報として、以下のような情報を表示したいと考えているので、

- アドレス
- 公開鍵
- モザイク保有量
- インポータンス

サービスに関わるファイルを配置するディレクトリとして`src/app/model`ディレクトリを作成し、その配下に、アカウント情報に関連したサービスのファイルを配置するディレクトリとして`src/app/model/account`ディレクトリを作成し、`src/app/model/account/account.model.ts`ファイルを以下のような内容で作成します。

```typescript:account.model.ts
export type Account = {
  address: string;
  publicKey: string;
  mosaics: {
    id: string;
    amount: bigint;
  }[];
  importanceMicroXym: bigint;
}
```

この型に沿った情報をsymbol-sdkで取得するサービス(≒インフラサービス)、インフラサービスを利用してアプリに情報を渡すために使用するサービスの計2個のサービスを作っていきます。

`src/app/model/account`ディレクトリに移動し、`ng g service account-infrastructure`コマンドでAccountInfrastructureServiceを、`ng g service account`コマンドでAccountServiceを作成し、それぞれ以下のアカウント情報取得の公式ドキュメントを参考にに実装し、`src/app/page/explorer/accounts/account/account.component.ts`で開発者ツールに情報を表示できるよう実装します。

- 公式ドキュメント symbol-sdk 使用方法ガイド アカウント情報取得 [https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html#method-01-using-the-sdk](https://docs.symbolplatform.com/ja/guides/account/getting-account-information.html#method-01-using-the-sdk)

`src/app/model/account/account-infrastructure.service.ts`

```typescript:account-infrastructure.service.ts
import { Injectable } from '@angular/core';
import * as symbolSdk from 'symbol-sdk';
import { environment } from 'src/environments/environment';
import { Account } from './account.model';
import { InterfaceAccountInfrastructureService } from './account.service';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AccountInfrastructureService implements InterfaceAccountInfrastructureService {
  private nodeUrl = environment.nodeUrl;
  private repositoryFactoryHttp = new symbolSdk.RepositoryFactoryHttp(this.nodeUrl);
  private accountHttp = this.repositoryFactoryHttp.createAccountRepository();
  private accountInfo$?: Observable<symbolSdk.AccountInfo>;
  private account$?: Observable<Account>;

  constructor() { }

  getAccount$(address: string): Observable<Account> {
    const symbolSdkAddress = symbolSdk.Address.createFromRawAddress(address);
    this.accountInfo$ = this.accountHttp.getAccountInfo(symbolSdkAddress);
    this.account$ = this.accountInfo$.pipe(
      map((accountInfo) => {
        const account: Account = {
          address: accountInfo.address.plain(),
          publicKey: accountInfo.publicKey.toString(),
          mosaics: accountInfo.mosaics.map((mosaic) => {
            return {
              id: mosaic.id.toHex(),
              amount: BigInt(mosaic.amount.toString())
            }
          }),
          importance: BigInt(accountInfo.importance.toString())
        }
        return account;
      })
    );
    return this.account$;
  }
}
```

`src/app/model/account/account.service.ts`

```typescript:account.service.ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { AccountInfrastructureService } from './account-infrastructure.service';
import { Account } from './account.model';

export interface InterfaceAccountInfrastructureService {
  getAccount$: (address: string) => Observable<Account>;
}

@Injectable({
  providedIn: 'root'
})
export class AccountService {
  private account$?: Observable<Account>;

  constructor(private accountInfrastructureService: AccountInfrastructureService) { }

  getAccount$(address: string): Observable<Account> {
    this.account$ = this.accountInfrastructureService.getAccount$(address);
    return this.account$;
  }
}
```

`src/app/page/explorer/accounts/account/account.component.ts`

```typescript:account.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';
import { map, mergeMap } from 'rxjs/operators';
import { Account } from 'src/app/model/account/account.model';
import { AccountService } from 'src/app/model/account/account.service';

@Component({
  selector: 'app-account',
  templateUrl: './account.component.html',
  styleUrls: ['./account.component.css']
})
export class AccountComponent implements OnInit {
  address$?: Observable<string>;
  address?: string;
  account$?: Observable<Account>;
  account?: Account;

  constructor(private route: ActivatedRoute, private accountService: AccountService) { }

  ngOnInit(): void {
    this.address$ = this.route.params.pipe(map((params) => params.address));
    this.address$.subscribe(
      (address) => {
        console.log("address", address);
        this.address = address;
      }
    );
    this.account$ = this.address$.pipe(mergeMap((address) => this.accountService.getAccount$(address)))
    this.account$.subscribe(
      (account) => {
        console.log("account", account);
        this.account = account;
      }
    );
  }

}
```

ここの実装でのポイントは、AccountService内で定義したinterfaceの規格を満たすようにAccountInfrastructureServiceを実装することで、AccountServiceがAccountInfrastructureServiceに依存するのではなく、AccountInfrastructureServiceがAccountServiceに(あたかも仮想的に)依存しているかのような状態にし、symbol-sdkに何らかの変更があった場合に、AccountInfrastructureService -> AccountService -> AccountComponentのような流れで連鎖的に全ての箇所を修正しなければならないような状況を防ぎ、AccountInfrastructureServiceのみを修正すればOKな状態を自然と維持することを意図していることです。

これは、オブジェクト試行設計において、依存性逆転の原則(Dependency Inversion Principle)と呼ばれる考え方を実現することを意図しています。

一見、無駄な層があるだけのようにも見えるのですが、InfrastructureServiceとServiceの2層に分けておき、Serviceの方をアプリ側で必要なデータ構造に沿ってシンプルに利用できる形に保ち、InfrastructureServiceの方でServiceが必要とする型のデータを常にインターフェースを満たす形で実装することが強制されることで、依存性逆転の原則が実現できるという風に捉えて頂ければと思います。

さて、ここまで実装して、ようやくアカウント情報が表示できるかと思いきや、`ng serve`を実行すると、以下のように、コンパイルでエラーが出ます。

```sh
./node_modules/futoin-hkdf/hkdf.js:22:35-54 - Error: Module not found: Error: Can't resolve 'crypto' in '/home/yasunori_matsuoka/sandbox/symbol-sample-angular/node_modules/futoin-hkdf'

BREAKING CHANGE: webpack < 5 used to include polyfills for node.js core modules by default.
This is no longer the case. Verify if you need this module and configure a polyfill for it.

If you want to include a polyfill, you need to:
        - add a fallback 'resolve.fallback: { "crypto": require.resolve("crypto-browserify") }'
        - install 'crypto-browserify'
If you don't want to include a polyfill, you can use an empty module like this:
        resolve.fallback: { "crypto": false }

./node_modules/symbol-sdk/dist/src/core/crypto/Crypto.js:19:15-32 - Error: Module not found: Error: Can't resolve 'crypto' in '/home/yasunori_matsuoka/sandbox/symbol-sample-angular/node_modules/symbol-sdk/dist/src/core/crypto'

BREAKING CHANGE: webpack < 5 used to include polyfills for node.js core modules by default.
This is no longer the case. Verify if you need this module and configure a polyfill for it.

If you want to include a polyfill, you need to:
        - add a fallback 'resolve.fallback: { "crypto": require.resolve("crypto-browserify") }'
        - install 'crypto-browserify'
If you don't want to include a polyfill, you can use an empty module like this:
        resolve.fallback: { "crypto": false }

Error: account-infrastructure.service.ts:32:23 - error TS2583: Cannot find name 'BigInt'. Do you need to change your target library? Try changing the 'lib' compiler option to 'es2020' or later.

32               amount: BigInt(mosaic.amount.toString())
                         ~~~~~~


Error: account-infrastructure.service.ts:35:23 - error TS2583: Cannot find name 'BigInt'. Do you need to change your target library? Try changing the 'lib' compiler option to 'es2020' or later.

35           importance: BigInt(accountInfo.importance.toString())
```

## コンパイル時に`Error: Module not found: Error: Can't resolve 'crypto'`のエラーが発生

Angularのビルドで利用されているwebpackがNode.jsのcryptoモジュールをビルド対象に含められないことが原因です。

[https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-crypto](https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-crypto)

解決するためには、上記URLの参考資料の通り、`npm i crypto-browserify -S`コマンドを実行してインストールした後、`tsconfig.json`ファイルの`compilerOptions`配下に以下のように対象モジュールのパスを追記することで解決できます。

```json
    ....

    "paths": {
      "crypto": ["./node_modules/crypto-browserify"]
    }

    ...
```

## コンパイル時に`error TS2583: Cannot find name 'BigInt'. Do you need to change your target library? Try changing the 'lib' compiler option to 'es2020' or later.`のエラーが発生

こちらはBigIntが比較的新しめの機能なので、利用するために設定が必要なため発生しているエラーです。後述の通り、`tsconfig.json`ファイルの`"lib"`の配列内に`"es2020"`を追加します。

ここまで修正して再度`ng serve`を実行すると、今度は`stream-browserify`について同様のエラーが発生します。

## コンパイル時に`Error: Module not found: Error: Can't resolve 'stream'`のエラーが発生

[https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-stream](https://qiita.com/sengoku/items/21dc21e0095dc3d9c0de#warning-module-not-found-error-cant-resolve-stream)

crypto-browserifyのエラーと同様に、上記URLの通り、`npm i stream-browserify -S`コマンドを実行してインストールした後、`tsconfig.json`ファイルの`compilerOptions`配下に以下のように対象モジュールのパスを追記することで解決できます。

```json
    ....

    "paths": {
      "crypto": ["./node_modules/crypto-browserify"],
      "stream": ["./node_modules/stream-browserify"] // 追加
    }

    ...
```

再度`ng serve`を実行すると、コンパイルは成功していますが、ブラウザで[http://localhost:4200/](http://localhost:4200/)を表示すると、画面が真っ白になっており、開発者ツールのコンソールにエラーが表示されています。

## 実行時に`Uncaught ReferenceError: global is not defined`のエラーが発生

[https://dev.classmethod.jp/articles/angular6-referenceerror/](https://dev.classmethod.jp/articles/angular6-referenceerror/)

上記URLを参考に`src/polyfills.ts`に`(window as any).global = window;`を追記することで解決できます。

## symbol-sdkをAngularで利用する際のエラー解消まとめ

エラー解決方法と、解決した後の各ファイルの状態をこちらにまとめておきます。

- webpackのNode.jsのcrypto, streamモジュールの依存性解決の問題の解決 ... パッケージインストール＆`tsconfig.json`にパスを追記
- global is not undefinedの問題の解決 ... `polyfills.ts`に`(window as any).global = window;`を追記
- (bigintを使うためにES2020に対応できるよう設定を`tsconfig.json`に追記) ... symbol-sdkの導入に伴う問題ではなく、symbol-sdkのUInt64の型をTypeScriptのbiging型に変換してアプリ側で持つようにしたために発生した問題を解決するための手順であることには注意が必要です。bigint使わなければ不要です。

パッケージインストール

```sh
npm install crypto-browserify stream-browserify -S
```

`tsconfig.json`ファイルに追記

```json:tsconfig.json
/* To learn more about this file see: https://angular.io/config/tsconfig. */
{
  "compileOnSave": false,
  "compilerOptions": {
    "baseUrl": "./",
    "outDir": "./dist/out-tsc",
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "sourceMap": true,
    "declaration": false,
    "downlevelIteration": true,
    "experimentalDecorators": true,
    "moduleResolution": "node",
    "importHelpers": true,
    "target": "es2017",
    "module": "es2020",
    "lib": [
      "es2020", // bigintの問題解決のため追記
      "es2018",
      "dom"
    ],
    // crypto, streamの問題解決のためここから
    "paths": {
      "crypto": ["./node_modules/crypto-browserify"],
      "stream": ["./node_modules/stream-browserify"]
    }
    // ここまで追記
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

`src/polyfills.ts`に追記

```typescript:src/polyfills.ts
import 'zone.js';
(window as any).global = window; // global is not definedの問題解決のため追加
```

ここまでできると、`ng serve`して、[http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA)にアクセスしてみると、ブラウザの開発者ツールにaccountのデータが表示される状態になりました。

これで親コンポーネント上でアカウントの情報が取得できたので、後はこれを子コンポーネントに表示すればアカウント情報の表示が完成です。

親コンポーネントの`src/app/page/explorer/accounts/account/account.component.html`ファイル内で子コンポーネントを表示している箇所で、以下のようにアカウント情報のObservableなデータを渡してあげ、

```html:account.component.html
<app-view-account [account$]="account$"></app-view-account>
```

子コンポーネントの`src/app/view/explorer/accounts/account/account.component.ts`ファイル内で以下のように親コンポーネントからのObservableなデータを受け取り、

```typescript:account.component.ts
import { Component, Input, OnInit } from '@angular/core';
import { Observable } from 'rxjs';
import { Account } from 'src/app/model/account/account.model';

@Component({
  selector: 'app-view-account',
  templateUrl: './account.component.html',
  styleUrls: ['./account.component.css']
})
export class ViewAccountComponent implements OnInit {
  @Input() account$?: Observable<Account>;

  constructor() { }

  ngOnInit(): void {
  }

}
```

子コンポーネントの`src/app/view/explorer/accounts/account/account.component.html`ファイル内で以下のように表示することで、このhtmlファイル上のasyncパイプを通じてObservableなデータが流れてきたら、その値をasyncパイプを通じて受け取って、表示することができます。
`ngIf`や`ngFor`や`ng-container`等の利用方法や、Angular Flex-Layoutの利用方法等も盛り込んで、Angularらしさを意識した表示にしてみました。

```html:app.component.html
<ng-container *ngIf="account$ | async as account">
  <mat-card>
    <mat-card-header>
      Address
    </mat-card-header>
    <mat-card-content>
      {{account.address}}
    </mat-card-content>
  </mat-card>
  <mat-card>
    <mat-card-header>
      Public Key
    </mat-card-header>
    <mat-card-content>
      {{account.publicKey}}
    </mat-card-content>
  </mat-card>
  <mat-card>
    <mat-card-header>
      Mosaics
    </mat-card-header>
    <mat-card-content>
      <div fxLayout="row" fxLayoutGap="12px">
        <div fxLayout="column">
          <div>Id</div>
          <ng-container *ngFor="let mosaic of account.mosaics">
            <div>{{mosaic.id}}</div>
          </ng-container>
        </div>
        <div fxLayout="column">
          <div>Amount</div>
          <ng-container *ngFor="let mosaic of account.mosaics">
            <div>{{mosaic.amount}}</div>
          </ng-container>
        </div>
      </div>
    </mat-card-content>
  </mat-card>
  <mat-card>
    <mat-card-header>
      Importance
    </mat-card-header>
    <mat-card-content>
      {{account.importance}}
    </mat-card-content>
  </mat-card>
</ng-container>
```

これで、`ng serve`して、[http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA)のページにアクセスすると、アドレス、公開鍵、保有モザイク(≒トークン ... 保有量はdivisibilityが考慮されていない整数表記)、インポータンス(μXYM表記)が表示されたと思います。

最後に、サイドナビにアカウントページへのリンクを追加しておきましょう。

`src/app/app.component.ts`ファイルの`sideNavLinks`にアカウントページへのリンクやアイコンや名前を以下のように設定しておくとアクセスしやすくて良いでしょう。リンク先のアドレスは例えば自分のアドレスに変える等してみても良いと思います。

```typescript:app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  title = 'Symbol x Angular';
  sideNavLinks = [
    {
      name: 'Home',
      icon: 'home',
      linkPath: '/',
    },
    // ここから
    {
      name: 'Account',
      icon: 'account_circle',
      linkPath: '/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA'
    },
    // ここまで追加
  ];
}
```

これで、`ng serve`して、ホームページ[http://localhost:4200/](http://localhost:4200/)にアクセスし、サイドナビのAccountをクリックして[http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA](http://localhost:4200/explorer/accounts/NDLXI3OMXJCHO2A2ZD54TO4UZJQQV36DQYK33SA)に遷移し、symbol-sdkを用いてノードからアカウント情報を取得する簡易的なサンプルアプリを作成することができました！🎉

## まとめ

Angular, rxjsについての入門的な内容を一通り網羅しつつ、symbol-sdkを用いてSymbolブロックチェーンのノードと通信し、URLに指定されたアドレスのアカウント情報を表示するところまで解説しましたが、いかがだったでしょうか？

自分自身で書いていて思ったのが、ほとんどの部分がAngularとrxjsの解説になっているなあということです。

symbol-sdkを使うところは、一度サンプルコードを見て雰囲気つかんだ後は、TypeScriptの補完を頼りにさくさく書いていけそうな雰囲気を感じて頂けるのではないかと思います。

このように既存のWeb開発やアプリ開発のほんのわずかな延長線上で、ブロックチェーンの恩恵を手軽に受ける開発をできることがNEMやSymbolの強みだと思います。

ただ、Angularデフォルトの設定の状態に、symbol-sdkをインストールして使おうとすると、最初に環境面の問題に遭遇すると思うので、その点は本記事の[要約](##要約)や[symbol-sdkをAngularで利用する際のエラー解消まとめ](##symbol-sdkをAngularで利用する際のエラー解消まとめ)を参考にしてくださいますと幸いです。

この記事が、Symbolを使ってWebサービスを開発しようと思っている方や、Angularにこれから入門しようと思っている方に取って、何らかの助けになれば幸いです。

長文最後までお付き合い頂きありがとうございました。

## 最後に

もしNEMTUSに対しNEMやSymbol関連記事の寄稿や、サンプルとして公開したアプリについて何かありましたら、以下GitHubにて記事やサンプルアプリを公開しておりますので、お気軽にIssueやPull Request等、連携くださいますと幸いです。

- [https://github.com/nemtus/](https://github.com/nemtus/)

NEMTUSとして、NEM, Symbolに関する様々な技術情報を継続的に発信していきたいと考えていますので、今後ともどうぞよろしくお願いします。

## 記事作成者

- 名前
  - 松岡靖典
- 所属
  - 株式会社CauchyE(旧:株式会社LCNEM)
    - [https://cauchye.com/](https://cauchye.com/)
  - NPO法人NEMTUS
    - [https://nemtus.com/](https://nemtus.com/)
- 略歴
  - 株式会社CauchyEでWeb開発やブロックチェーンに関する開発を行いつつ、NPO法人NEMTUSにてブロックチェーン技術の普及推進活動にも従事
- SNS
  - twitter: [https://twitter.com/salaryman_tousi](https://twitter.com/salaryman_tousi)
  - GitHub: [https://github.com/YasunoriMATSUOKA](https://github.com/YasunoriMATSUOKA)
