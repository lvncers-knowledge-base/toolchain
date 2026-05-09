# Bundler

## 歴史

最初のころのWebって、JavaScriptファイルをそのまま `<script>` で読み込むだけだったんだよね。
だからコードが大きくなると、

* 読み込み順で壊れる
* グローバル変数が衝突する
* ファイル数が増えると遅い
* ライブラリ管理が地獄

みたいな問題が出てきた。

そこから「複数ファイルをまとめて1つにしよう」って流れで“バンドラー”文化が始まった感じ。
ざっくり時代ごとに見るとこんな感じ

### 1. Scriptタグ時代（〜2010前後）

```html
<script src="jquery.js"></script>
<script src="utils.js"></script>
<script src="app.js"></script>
```

全部手動。
依存順ミスると即死。
この時代はまだ「モジュール」という概念が弱かった。

流行ってたのは：

* jQuery
* Prototype.js
* MooTools

とか。

### 2. CommonJS & Browserify時代（2011〜2014）

Node.js が流行り始めて、

```js
const foo = require("./foo")
```

が便利すぎた。
でもこれは本来サーバー用。

そこで、

> 「requireをブラウザでも使いたい！」

って生まれたのが
[Browserify](https://browserify.org/?utm_source=chatgpt.com)

#### Browserifyの革命

Node風コード：

```js
const _ = require("lodash")
```

↓

ブラウザで動く1ファイルに変換。
これがかなり革命だった。
「npmの世界」がブラウザに来た。

### 3. Webpack時代（2014〜2020）

ここで王者登場。

https://webpack.js.org

Webpackは単なる「JS結合ツール」じゃなくて、

> “全部をモジュールとして扱う”

思想だった。

#### 何がすごかった？

JSだけじゃなく：

* CSS
* SCSS
* 画像
* フォント
* SVG

まで全部importできた。

```js
import "./style.css"
import logo from "./logo.png"
```

当時はかなり衝撃。

#### Loader文化

Webpackの特徴は「loader」。

```js
{
  test: /\.tsx$/,
  loader: "ts-loader"
}
```

変換パイプラインを自由に作れる。
でも…

#### 地獄の設定ファイル時代

```js
module.exports = {
  entry: ...
  output: ...
  module: ...
  plugins: ...
}
```

長い。難しい。壊れる。
「webpack.config.js職人」が生まれた。
当時のフロント界隈、

> “Webpack設定できる人 = 強い”

みたいな空気あった。
ちょっと黒魔術感ある。

### 4. Rollup時代（2016〜）

Webpackが巨大化しすぎて、

> 「ライブラリ作るならもっとシンプルでよくない？」

で出てきたのが

https://rollupjs.org

#### ES Modules特化

```js
import { foo } from "./foo"
```

を前提に設計。
Tree Shaking（未使用コード削除）が強かった。
ライブラリ作者に人気。
React系ライブラリもかなりRollup使ってた。

### 5. Parcel時代（2017〜）

[Parcel](https://parceljs.org/?utm_source=chatgpt.com)

思想は：

> “設定とかもう嫌だ”

ゼロコンフィグ。

```bash
parcel index.html
```

で動く。
かなり未来感あった。
ただ巨大アプリではWebpack優勢だった。

### 6. Vite時代（2020〜現在）

ここで流れが大きく変わる。

https://vite.dev

#### なぜ革命だった？

それまで：

```text
起動
↓
全部bundle
↓
開発サーバー開始
```

だった。
大規模になると数十秒待つ。

Viteは：

> 「開発時はbundleしなくてよくない？」

って考えた。

#### ES Modules をブラウザが理解する時代

ブラウザが直接：

```js
import ...
```

できるようになった。

だから開発時は：

* 必要ファイルだけ変換
* オンデマンド配信

になった。
超速い。

#### 本番だけRollup

面白いのが、

* 開発 → ESModules配信
* build → Rollup

ってハイブリッド構成。

### 7. esbuild / SWC / Turbopack時代

JavaScript製バンドラーは遅かった。

だから：

> 「RustとかGoで書こう」

になった。

#### esbuild

https://esbuild.github.io

Go製。
めちゃ速い。
Vite内部でも使われた。

#### SWC

https://swc.rs

Rust製。
Next.jsが採用。
Babel置き換え。

#### Turbopack

https://turbo.build/pack

Webpack作者がRustで再設計。
Next.js向け。
まだ発展中。

## 今の流れ

今は「全部を巨大bundleにする」思想が少し崩れてる。

理由：

* HTTP/2
* ESM
* Edge配信
* ブラウザ進化

で、昔ほど「1ファイル化」が絶対正義じゃなくなった。

だから最近は：

* 部分bundle
* lazy loading
* island architecture
* server components

みたいな方向に進んでる。
面白いのは、

Webpack時代

「ブラウザが弱いから全部隠蔽する」

↓

Vite以降

「ブラウザが強くなったから素直に使う」

になってること。

Web技術って、
「複雑化してからシンプルに戻る」
を周期的に繰り返すんだよね。
