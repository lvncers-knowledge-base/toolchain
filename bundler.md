# Bundler

## バンドラーとは

今のバンドラーって、単に「ファイルをまとめるツール」じゃなくて、

> “ブラウザが理解しやすい形に、アプリ全体を変換するシステム”

になってる。
ざっくり言うと、今のバンドラーはこのへん全部やってる

### 1. 依存関係を解析する

まずimportを全部辿る。

```js
import { foo } from "./foo"
import React from "react"
```

↓

```text
app.tsx
 ├ foo.ts
 ├ react
 └ button.tsx
      └ style.css
```

みたいな巨大グラフを作る。
これが「モジュールグラフ」。

### 2. ファイル変換（transpile）

ブラウザがそのまま理解できないものを変換。

例えば：

#### TypeScript

```ts
const x: number = 1
```

↓

```js
const x = 1
```

#### JSX

```jsx
<Button />
```

↓

```js
React.createElement(...)
```

あるいはReact Compiler系の別形式。

#### SCSS

```scss
$color: red;
```

↓

```css
color:red;
```

### 3. node_modules解決

これ超重要。

```js
import React from "react"
```

って、実はブラウザには意味不明。

バンドラーが：

```text
node_modules/react/index.js
```

とかを探して解決してる。
Nodeのmodule resolutionを再現してる感じ。

### 4. Tree Shaking

使ってないコードを消す。

```js
import { a, b } from "./utils"
```

で `a` しか使ってなければ、`b` 関連を削除。
これがないとbundleサイズ爆増する。

### 5. Code Splitting

巨大アプリを分割。

昔：

```text
app.js (10MB)
```

今：

```text
home.chunk.js
admin.chunk.js
settings.chunk.js
```

必要時だけロード。

```js
const Admin = lazy(() => import("./Admin"))
```

とか。

### 6. Asset管理

JSだけじゃない。

#### 画像

```js
import logo from "./logo.png"
```

↓

```text
/assets/logo.a1b2c3.png
```

に変換。

#### CSS

```js
import "./style.css"
```

↓

別CSSファイル生成。

#### Font/SVG/WebWorker

全部処理する。

### 7. 最適化

ここが本番ビルドの本体。

#### Minify

```js
function hello(name) {
  console.log(name)
}
```

↓

```js
function hello(o){console.log(o)}
```

#### Dead Code Elimination

絶対呼ばれないコード削除。

#### 定数埋め込み

```js
if (process.env.NODE_ENV === "development")
```

↓

```js
if(false)
```

↓

消える。

### 8. 開発サーバー

今はここが超重要。

#### HMR (Hot Module Reload)

保存した瞬間：

```text
変更ファイルだけ差し替え
```

React state維持したまま更新。
昔は毎回ページリロードしてた。
地獄。

### 9. ESM と CommonJS の橋渡し

現代 JS、モジュール方式が混在してる。

* CommonJS
* ESModules
* UMD
* IIFE

全部いる
バンドラーはそれを繋ぐ翻訳機。

### 10. Runtime生成

Webpack系は小さいランタイムも作る。

例えば：

```text
chunk読み込み管理
module cache
dynamic import制御
```

みたいなの。

### 現代バンドラーの実態

今のバンドラーって実質：

| 機能              | 内容       |
| --------------- | -------- |
| Compiler        | TS/JSX変換 |
| Linker          | import解決 |
| Optimizer       | 圧縮・削除    |
| Asset Pipeline  | CSS/画像処理 |
| Dev Server      | HMR      |
| Runtime Builder | chunk管理  |

全部入り。
もはや“小さなOS”みたいになってる。

だから最近は逆に、

> 「全部バンドラーがやる必要ある？」

って思想も強くなってる。

例えば：

* ブラウザネイティブESM
* import maps
* edge runtime
* server components

とかは、

「バンドラー依存を減らそう」

という流れでもある。

面白いのは、
Webpack時代は

> “ブラウザを抽象化する”

だったのに、

最近は

> “ブラウザを信頼する”

方向に戻ってることなんだよね。

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
