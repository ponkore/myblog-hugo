+++
shortname = "webpack-for-non-SPA"
title = "SPAでない Web Application のための webpack"
description = "SPAでない Web Application のための webpack"
date = "2018-07-17T09:13:54+09:00"
categories = ["Programming"]
tags = ["JavaScript"]
archives = ["2018-07"]
url = "post/2018-07/17/webpack-for-non-spa"
thumbnail = "/img/2018-07/17/webpack-for-non-spa.png"
+++

SPAでない Web Application で webpack を使おうしたときに少しハマったの
でメモしておきます。

<!--more-->

## 何が問題だったか

TypeScript を使いたかったので、素直に webpack を使おうと思ったのですが、
React とか SPA での使い方(要するに一つの js ファイルに bundle する感じ)は
例が多いのですが、1 html に 1 js (ソースは 1 tsファイル) というパター
ンはあまり見かけなかったので、少し試行錯誤しました。

## ディレクトリ構成

だいたい以下のようなディレクトリ構成を想定しておきます。

* src/ts/pageNN.ts に各ページに書く TypeScript のソースを置きます。
* src/ts/common.ts (仮称)は、共通モジュールとなります。
* コンパイル結果は public/js/pageNN.js に書き出すイメージです。
* ただしOSSのモジュール等は、public/js/vendor.js に書き出し、pageNN.js
  には含みません。
* デバッグ用に .map を出力しています。
* CSS のソースは、src/css/base.css 他に置き、コンパイルしたら
  public/js/style.css.js に出力するイメージです。

```
.
├── package-lock.json
├── package.json
├── public
│     ├── page01.html
│     ├── page02.html
│     ├──    :
│     └── js
│         ├── page01.js
│         ├── page01.js.map
│         ├── page02.js
│         ├── page02.js.map
│         ├──    :
│         ├──    :
│         ├── style.css
│         ├── style.css.js
│         ├── style.css.js.map
│         ├── style.css.map
│         ├── vendor.js
│         └── vendor.js.map
├── src
│     ├── css
│     │     ├── base.css
│     │     ├──   :
│     │     └──
│     ├── js
│     │     └── jquery.numeric.js
│     └── ts
│         ├── page01.ts
│         ├── page02.ts
│         ├──     :
│         ├──     :
│         └── common.ts
├── tsconfig.json
├── tslint.json
└── webpack.config.js
```

## webpack.config.js の設定

最終型に至るまでの過程を順を追って説明します。

### 複数ソース複数出力の設定

ページ毎にソースを分ける、ということを実現するには、`entry:` を分けて
やればよいです。

※本当は、ページが増えたら自動的に entry も増える、といった構成がよい
と思うのですが、webpack.config.js をあまり複雑にしたくなかったのでこれ
で妥協しました。

``` webpack.config.js
module.exports = {
  entry: {
    'entry01': './src/ts/page01',
    'entry02': './src/ts/page02',
  },
  output: {
    path: path.resolve(__dirname, 'public/js'),
  },
  // :
};
```

このように記述することで、`public/js/page01.js` 等にコンパイル結果が出
力されます。

### vendor.js への外部依存ソースの分離

このままだと、各ソースが依存(TypeScript で言うところの import) してい
るソースは、上記 page01.js 等に各々書き出されてしまいます。

これはこれで悪くはないのですが、各 js ファイルが大きくなってしまうのが
気になるとか、どうせならキャッシュを効かせたい、等と考えるようになりま
す。そこで、.ts が依存するソースは、`vendor.js` にひとまとめにしてしま
います。

``` webpack.config.js
module.exports = {
  // :
  optimization: {
    splitChunks: {
      name: 'vendor',
      chunks: 'initial',
    }
  },
  // :
};
```

これにより `public/js/vendor.js` というファイルが生成されるようになる
ので、HTML 側から忘れずに参照するようにしておきます。

### TypeScript まわり

TypeScript 関連を以下に抜き出してみました。ここでの記載内容は、SPA の
場合とあまりかわらないはずです。

```webpack.config.js
module.exports = {
  // :
  module: {
    rules: [
      {
        test: /\.ts$/,
        // exclude: /node_modules/,
        use: {
          loader: 'ts-loader',
        }
      },
      // :
    ]
  },
  resolve: {
    extensions: ['.ts', '.js']
  },
  // :
};
```

### CSS まわり(Sass の導入)

CSS については、Sass を使うことにします。また css ソースは、複数ソース
に分かれていることを想定し、複数ソース→１ファイルに変換する、というイ
メージで考えてみます。まずは entry への追加です。

```webpack.config.js
const glob = require('glob');
const cssfiles = glob.sync('src/**/*.css').map(f => `./${f}`);

module.exports = {
  entry: {
    'entry01': './src/ts/page01',
    'entry02': './src/ts/page02',
    // :
    'style.css': cssfiles
  },
  // :
};
```

ここでは対象となる css を glob でかき集めていますが、`./` を先頭につけ
る必要があったので、上記のような書き方になっています。

``` webpack.config.js
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractCSS = new ExtractTextPlugin({ filename: '[name]', allChunks: false });

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: extractCSS.extract({ use: 'css-loader' })
      },
      {
        test: /\.scss$/,
        use: extractCSS.extract({ use: [{ loader: 'css-loader' }, 'sass-loader'] })
      },
      {
        test: /\.(png|gif|svg|woff)$/,
        use: 'file-loader'
      }
    ]
  },
  plugins: [
    extractCSS
  ],
  resolve: {
    extensions: ['.css', '.png', '.gif', '.svg']
  },
  // :
};
```

### デバッグ関連の設定

実はここでもハマりました。ざっくりデバッグ関連の設定を抜き出しておきま
す。

``` webpack.config.js
const DEBUG = !process.argv.includes('production');
module.exports = {
  // :
  devtool: DEBUG ? 'source-map' : false,
  devServer: {
    contentBase: path.resolve(__dirname, 'public'),
    publicPath: '/js/',
    port: 3000,
    watchContentBase: true
  }
```

`devtool: source-map` でソースマップを出力します。あと上記 `devServer`
の設定が結構デリケートでした（contentBase, publicPath,
watchContentBase, のどれかひとつでも間違えると、livereload してくれま
せん。しかもなんのエラーもなく手がかりもない...）。

## まとめ

ここまでの結果をまとめておきます。

* webpack.config.js

``` webpack.config.js
const path = require('path');
const glob = require('glob');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractCSS = new ExtractTextPlugin({ filename: '[name]', allChunks: false });
const cssfiles = glob.sync('src/**/*.css').map(f => `./${f}`);
const DEBUG = !process.argv.includes('production');

module.exports = {
  entry: {
    'entry01': './src/ts/page01',
    'entry02': './src/ts/page02',
    'style.css': cssfiles
  },
  output: {
    path: path.resolve(__dirname, 'public/js'),
  },
  optimization: {
    splitChunks: {
      name: 'vendor',
      chunks: 'initial',
    }
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        // exclude: /node_modules/,
        use: {
          loader: 'ts-loader',
        }
      },
      {
        test: /\.css$/,
        use: extractCSS.extract({ use: 'css-loader' })
      },
      {
        test: /\.scss$/,
        use: extractCSS.extract({ use: [{ loader: 'css-loader' }, 'sass-loader'] })
      },
      {
        test: /\.(png|gif|svg|woff)$/,
        use: 'file-loader'
      }
    ]
  },
  plugins: [
    extractCSS
  ],
  resolve: {
    extensions: ['.ts', '.js', '.css', '.png', '.gif', '.svg']
  },
  devtool: DEBUG ? 'source-map' : false,
  devServer: {
    contentBase: path.resolve(__dirname, 'public'),
    publicPath: '/js/',
    port: 3000,
    watchContentBase: true
  }
};
```

* html

```html
<html>
  <head>
    <!--
      head here
    -->
    <link href="js/style.css" rel="stylesheet" type="text/css" />
    <script type="text/javascript" src="js/vendor.js"></script>
  </head>
  <body>
    <!--
      body here
    -->
    <script type="text/javascript" src="js/page01.js"></script>
  </body>
</html>
```

今後は SPA でアプリを作ることが多くなりそうですが、もしかしたら使うこ
ともあるかもしれないので、まとめておきました。
