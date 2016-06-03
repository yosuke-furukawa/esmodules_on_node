<!--
今一番 Node.js の中で hot な discussion の一つと言えるでしょう、<span style="color: #ff5252"><b>『ES Modules が Node.js の中でどうなるか』</b></span>です。
-->

Node.js team has lots of discussion day by day. Today I will introduce one of the hottest discussion on Node.js.

The discussion is <span style="color: #ff5252"><b>"How do we handle ES Modules on Node.js"</b></span>

<!--
# ES Modules 現況
-->

## ES Modules

<!--
ES2015 が発刊されてそろそろ一年です。 ES2015 にある機能は Node.js v6でも 93% 程度カバーされています。モダンブラウザでも大体が90%を超えています。しかし、 ES Modules だけはまだどのブラウザも実装しきれていません(kangax compat table は ES Modules は省かれてます)。
-->

ECMAScript 2015 (ES2015, formerly ES6) was published almost a year ago. Node.js v6 supports 93% of the ES2015 syntax and features and most modern browsers exceed 90%. However, no JavaScript runtime currently supports ES Modules. (Note that [kangax's compatibility table](https://kangax.github.io/compat-table/es6/) does not yet have an ES Modules column.)

<img src="http://cdn-ak.f.st-hatena.com/images/fotolife/y/yosuke_furukawa/20160509/20160509010917.png" />

<!--
そもそも ECMAScript 2015 自身で定義されたのは構文だけなので、構文はともかく、どうやってモジュールを取ってくるかという Loader の部分がまだ決まりきっていません。
-->

ECMAScript 2015 defines the ES Modules syntax but ECMAScript does not define a "Loader" specification which determines how Modules are inserted into the runtime. The Loader spec is being [defined by WHATWG](https://whatwg.github.io/loader/), but is not yet finalized.

<!--
現時点はいくつも決めなきゃいけないポイントがあって
の全てを決めて一旦ロードマップ上のMilestone 0 が達成されるような状況です。
-->

The WHATWG Loader spec needs to define the following items for [Milestone 0](https://github.com/whatwg/loader/blob/master/roadmap.md) on its roadmap:

<!--
- 参照解決処理
- 取得処理
- script タグでどう書くのか
- メモ化処理(所謂caching)
-->

- Name resolution (relative and absolute URLs and paths)
- Fetch integration
- How to describe script tag: `<script type="module">`
- Memoization / caching

<!--
scriptタグでどう書くのか、参照解決処理など、ある程度決まっている処理はありますが、どの項目もまだ議論中です（[少なくとも github 上ではまだ  milestone 0 を discussion している最中に見える](https://github.com/whatwg/loader/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+milestone+0+)）。各種ブラウザでも、実装が始まっているところはありますが、仕様の方針待ちなところが多いです。
-->

The Module script tag has been [defined](https://blog.whatwg.org/js-modules), but the other items are still under discussion. You can check the status of this discussion on [GitHub](https://github.com/whatwg/loader/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+milestone+0+). Some browsers have started implementation, but most are waiting for finalization of the Loader spec.

<!--
# なんで Node.js に ES Modules が必要なのか
-->

## Why does Node.js need ES Modules?

<!--
ES Modules の仕様が定義されるよりも前に Node.js は CommonJS と呼ばれるモジュールシステムを採用しました((（正確には CommonJS の仕様からは大分外れてます。今やあれが CommonJS という事になっちゃってますが・・・）))。それと npm というパッケージマネージャの組み合わせでエコシステムを作っています。結果として npm のエコシステムは Node.js にとどまらず、Browserify や webpack を組み合わせてフロントエンドにとっても大きなエコシステムになっています。
-->

When Node.js came into existence, an ES Modules proposal didn't exist. Node.js decided to use [CommonJS](https://en.wikipedia.org/wiki/CommonJS) Modules. While CommonJS as an organization is no longer an active concern, Node.js and npm have evolved the specification to create a very large JavaScript ecosystem. [Browserify](http://browserify.org/) and more recently [webpack](https://webpack.github.io/) bring Node's version of CommonJS to the browser and solve module problems gracefully. As a result, the Node/npm JavaScript module ecosystem spans both server and client and is growing rapidly.

<!--
<span style="color: #d32f2f"><b>『CommonJS で既に育ってしまった生態系の中で ES Modules という標準仕様とどうやって相互運用性(interoperability)を取るのか』</b></span>
これが ES Modules が定義され始めた最初からずっと Node.js / npm で語られてる事でした。
-->

But how do we deal with interoperability between *standard* ES Modules and CommonJS-style modules in such a big ecosystem? This question has been debated heavily since the beginning of the ES Modules spec process.

<!--
相互運用性がないとこれまでのエコシステムと乖離(friction)ができてしまいます。せっかく Browserify や webpack で埋めたfrontend browserとNode.js との乖離がこれでまた起きることになります。
-->

Browserify and webpack currently bridge the gap between browser and server to make JavaScript development easy and somewhat unified. If we lose interoperability, we increase the friction between the existing ecosystem and new standard. If front-end developers choose ES Modules as their preferred default and server-side engineers continue to use Node's CommonJS, the gap will only widen.

<!--
# Node.js ではじゃあどうしようとしているのか
-->

## An interoperability proposal for Node.js

<!--
これではイカン、という事で [Bradley Meck](https://twitter.com/bradleymeck) 氏が interop を取ろうと Proposal を書き起こしました。最初に Proposal を書いた時は議論がいくつもあったので [ものすごくたくさんの話が巻き起こってまとまらなかった](https://github.com/nodejs/node-eps/pull/3) のですが、何度も何度も議論を重ねて今やっと `DRAFT` というステータスになっています。 
-->

[Bradley Farias (a.k.a Bradley Meck)](https://twitter.com/bradleymeck) has written a proposal for interoperability between CommonJS and ES Modules. The proposal is presented in the form of a Node.js EP (Enhancement Proposal) and the [pull request](https://github.com/nodejs/node-eps/pull/3) generated record amounts of discussion but also helped shape and tune the proposal. The EP was merged but still retains `DRAFT` status, indicating a preference rather than a clear intention to even implement ES Modules in Node.js. You can read the proposal here: <https://github.com/nodejs/node-eps/blob/master/002-es6-modules.md>.

Discussion and options explored during the development of this proposal are mostly found throughout the initial pull request comments thread but a partial summary can be found on the [Node.js wiki](https://github.com/nodejs/node/wiki/ES6-Module-Detection-in-Node).

The biggest challenge for Node.js is that it doesn't have the luxury of a `<script type="module">` tag to tell it whether any given file is in CommonJS format or an ES Module. Unfortunately you can't even be sure in all cases what type of file you have simply by parsing it as the Modules spec presents us with some ambiguities in the distinction. It's clear that we need some signal that Node.js can use to determine whether to load a file as CommonJS (a "Script") or as an ES Module.

Some constraints that were applied in the decision making process include:

* Avoiding a "boilerplate tax" (e.g. `"use module"`)
* Avoiding double-parsing if possible as Modules and Scripts parse differently
* Don't make it too difficult for non-JavaScript tools to make the determination (e.g. build toolchains such as [Sprockets](https://github.com/rails/sprockets) or Bash scripts)
* Don't impose a noticeable performance cost on users (e.g. by double-parsing large files)
* No ambiguity
* Preferably self-contained
* Preferably without vestiges in a future where ES Modules may be the most prominent type

Clearly compromise has to be made somewhere to find a path forward as some of these constraints are in conflict when considering the options available.

The route chosen for the Node.js EP, and currently accepted by the Node.js CTC for ES Modules is detection via filename extension, `.mjs` (alternatives such as `.es`, `.jsm` were ruled out for various reasons).

Detection via filename extension provides a simple route to determining the intended contents of a JavaScript file: if a file's extension is `.mjs` then the file will load as an ES Module, but `.js` files will be loaded as a Script via CommonJS.

<!--
## ES Modules on Node.js 概要
-->

### Basic interoperability algorithm

<!--
おおまかな解決アルゴリズムを記述します。
-->

The following algorithm describes how interoperability between ES Modules and CommonJS can be achieved:

<!--
```
1.  読み込もうとしているファイルが CommonJS で定義されているのか ES Modules で定義されているのかを確認する(※)
2. もし CommonJS なら
  2-1. ファイルを即時評価する（今まで通り）
  2-2. DynamicModuleRecord に `module.exports` で読み込んだものを入れる
3. もし ES Modules なら
  3-1. ファイルをパースする（import/export でファイルを取得して、bindingを作るため）
  3-2. 再帰的に全ての依存関係のあるファイルを持ってくる
  3-3. 全ての依存関係のファイルから `import`  の binding を作る
  3-4. 評価する
```
-->

```
1. Determine if file is an ES Module (ES) or CommonJS (CJS)
2. If CJS:
  2.1. Wrap CJS code to bootstrap code
  2.1. Evaluate as script
  2.2. Produce a DynamicModuleRecord from `module.exports`
3. If ES:
  3.1. Parse for `import`/`export`s and keep record, in order to create bindings
  3.2. Gather all submodules by performing recursive dependency loading
  3.3. Connect `import` bindings for all relevant submodules
  3.4. Evaluate as module
```

<!--
簡易フローチャートで書くとこうですね。
-->

![ES Modules / CommonJS interop flow](https://raw.githubusercontent.com/yosuke-furukawa/esmodules_on_node/master/images/output.png)

<!--
この後さらにケースとしてはファイルが循環参照されてたらどうするかとかの話がありますが、一旦そこは置いておきます。
-->

<!--
`読み込もうとしているファイルが CommonJS で定義されているのか ES Modules で定義されているのかを確認する` ここが今のところ最大の議論のポイントです。
-->

<!--
CommonJS なのか ES Modules なのかの確認方法ですが、読み込もうとしているファイルが<span style="color: #d32f2f"><b> `.mjs` の拡張子だったら ES Modules、それ以外の `.js` 等であれば普通に CommonJS として判断しよう</b></span>としています。
-->

<!--
つまり、 CommonJS でも ES Modules でも両方共読み込ませたいモジュールを作る場合、 `package.json` に下記のように記述し、
-->

For example, if a developer wanted to create a module that exports both module types (CommonJS and ES Modules) for backward compatibility, their `package.json` may be defined as:

```javascript
{
  "name": "test",
  "version": "0.0.1",
  "description": "",
  "main": "./index", // no file extension
}
```

<!--
`index.mjs` と `index.js` を定義します、片方には ES Modules 形式で書きます。
-->

The package will then have both an `index.mjs` and an `index.js`. The `index.mjs` is an ES Module, using the new `export` / `import` syntax:

```javascript
// index.mjs
export default class Foo {
  //..
}
```

<!--
もう片方には CommonJS 形式で書きます。
-->

And the `index.js` is a CommonJS style module, using the `module.exports` object:

```javascript
// index.js
class Foo {
  // ...
}
module.exports = Foo;
```

<!--
こうすると読み込む側が `.mjs` 形式に対応している Node.js であれば、 先に `.mjs` で解決しに行きます。見つからなければ `.js` の方を解決する、という動きになります。
-->

If the version of Node.js being used supports ES Modules via the `.mjs` file extension, it will first try to find an `index.mjs`. On the other hand, if the version of Node.js does _not_ support ES Modules (such as Node.js v4 or v6), or it can not find an `index.mjs`, it will look for an `index.js`.

<!--
## import で書く場合の path 解決方式
-->

According to the EP, you would be able to use both `require` and `import` to find packages in your node_modules:

```javascript
import mkdirp from 'mkdirp';
require('mkdirp');
```

<!--
本筋からはそれますが、 `import` の重要なポイントなので記載しておきます。 ES Modules では Node.js が暗黙的にやっているようなスマートなパスの解決をしてくれない（現時点のローダーでは）ので気をつけましょう。
例えば、 `require` で書いた場合、 `require('./foo')` のように `.js` を削除して記載することが可能でした。
-->

<!--
`import` でモジュール参照解決をする場合、ES Modules の仕様としては暗黙的に `.js` を保管してくれたりしないので気をつけましょう。
-->

For resolving modules local to your own project or package, you do not need to add a file extensions in your `require()` or `import` statements unless you want to be precise. The standard Node.js file resolution algorithm applies when you don't supply an extension, but an `.mjs` version is looked for _before_ a `.js`:

```javascript
require('./foo');
import './foo';
// these both look at
//   ./foo.mjs
//   ./foo.js
//   ./foo/index.mjs
//   ./foo/index.js

// to explicitly load a CJS module, add '.js':
import './foo.js';
// to explicitly load an ES module add '.mjs'
import './bar.mjs';
```

### Examples: Consuming CommonJS with ES Modules

<!--
これまでの説明だけでも分かりにくいと思うので例を上げて説明していきます。ここでは ES Modules から CommonJS で定義されたモジュールを読み込む場合です。ES Modules から CJS を「名前付きで」読み込んだ場合、 `default` というプロパティが入ります（これめっちゃ分かりにくい）。
-->

**Example 1: Load CommonJS from ES Modules**

```javascript
// cjs.js
module.exports = {
  default:'my-default',
  thing:'stuff'
};
```

```javascript
// es.mjs

import * as baz from './cjs.js';
// baz = {
//   get default() {return module.exports;},
//   get thing() {return this.default.thing}.bind(baz)
// }
// console.log(baz.default.default); // my-default

import foo from './cjs.js';
// foo = {default:'my-default', thing:'stuff'};

import {default as bar} from './cjs.js';
// bar = {default:'my-default', thing:'stuff'};
```

<!--
値を export して、 default の値としてアサインされる例：
-->

**Example 2: Export value and assigning "default"**

```javascript
// cjs.js
module.exports = null;
```

```javascript
// es.mjs
import foo from './cjs.js';
// foo = null;

import * as bar from './cjs.js';
// bar = {default:null};
```

<!--
関数を export する例：
-->

**Example 3: Single-function export**

```javascript
// cjs.js
module.exports = function two() {
  return 2;
};
```

```javascript
// es.mjs
import foo from './cjs.js';
foo(); // 2

import * as bar from './cjs.js';
bar.name; // 'two' ( get function name)
bar.default(); // 2 ( assigned default function )
bar(); // throws, bar is not a function
```

### Examples: Consuming ES Modules with CommonJS

<!--
反対に CommonJS から ES Modules を読み込む時は下記のようになります。こちらは `export default` で export した場合は `.default` プロパティにアサインされます。
-->

<!--
export default を利用する例:
-->

**Example 1: Using `export default`**

```javascript
// es.mjs
let foo = {bar:'my-default'};
export default foo;
foo = null; // this null value does not effect import value.
```

```javascript
// cjs.js
const es_namespace = require('./es');
// es_namespace ~= {
//   get default() {
//     return result_from_evaluating_foo;
//   }
// }
console.log(es_namespace.default);
// {bar:'my-default'}
```

<!--
export を利用する例：
-->

**Example 2: Using `export`**

```javascript
// es.mjs
export let foo = {bar:'my-default'};
export {foo as bar};
export function f() {};
export class c {};
```

```javascript
// cjs.js
const es_namespace = require('./es');
// es_namespace ~= {
//   get foo() {return foo;}
//   get bar() {return foo;}
//   get f() {return f;}
//   get c() {return c;}
// }
```

<!--
# 今のところの議論
-->

## Current state of discussion

<!--
ちょうど今盛り上がってるのには理由があって、仕様が `DRAFT` になって議論が始まった後に新しく  Counter Proposal (反対提案) が書かれました。それが [defense of dot js](https://github.com/dherman/defense-of-dot-js/blob/master/proposal.md) という Proposal です。
-->

Although built in a collaborative process, taking into account proposals for alternatives, Bradley's landed EP received a prominent counter-proposal from outside of the EP process. Going by the name "[In Defense of .js](https://github.com/dherman/defense-of-dot-js/blob/master/proposal.md)", this counter-proposal relies on the use of `package.json` rather than a new file extension. Even though this option had been previously discussed, this new proposal contains some interesting additions.

<!--
これは `.mjs` という拡張子で解決するのではなく、 `package.json` にフィールドを足すだけで解決させるようにしたいという Proposal です。
-->

<!--
## defense of dot js の内容
-->

<!--
こちらの仕様では、基本的に完全な互換性を取るのは諦め、 ES Modules で読み込むことをベースとします。 `package.json` に `main` フィールドがある時だけはすべてのファイルが CommonJS で読み込まれます。これが基本的な仕様です。
-->

_In Defense of .js_ presents the following rules for determining what format to load a file, with the same rules for both `require` and `import`:

<!--
明示的に ES Modules で読み込ませたければ `package.json` に `module` フィールドでエントリーポイントを書きます。
-->

* If `package.json` has `"main"` field but not a `"module"` field, all files in that package are loaded as CommonJS.
* If a `package.json` has a `"module"` field but not `"main"` field, all files in that package are loaded as ES Modules.
* If a `package.json` has neither `"main"` nor `"module"` fields, it will depend on on whether an `index.js` or a `module.js` exists in the package as to whether to load files in the package as CommonJS or ES Modules respectively.
* If a `package.json` has both `"main"` and `"module"` fields, files in the package will be loaded as CommonJS unless they are enumerated in the `"module"` field in which case they will be loaded as ES Modules, this may also include directories.
* If there is no `package.json` in place (e.g. `require('c:/foo')`), it will default to being loaded as CommonJS.
* A special `"modules.root"` field in `package.json`, files under the directory specified will be loaded as ES Modules. Additionally, files loaded relative to the package itself (e.g. `require('lodash/array')`) will load from within this directory.

### _In Defense of .js_ Examples

```javascript
// package.json
// all files loaded as CommonJS
{
  "main": "index.js" // default module for package
}
```

```javascript
// package.json
// default to CommonJS, conditional loading of ES Modules
{
  "main": "index.js", // used by older versions of Node.js as default module, CommonJS
  "module": "module.js" // used by newer versions of Node.js as default module, ES Module
}
```

<!--
こうすると、 `module` フィールドがあれば Node.js は ES Module としてエントリポイントを見に行きます。古いバージョンの Node.js は `main` フィールドのエントリポイントを見るだけなので古いバージョンにも対応されます。
-->

<!--
しかし、このやり方には問題が１つあります。 `module` 以外のエントリポイントを `require` から指定できません。例えば、 `lodash` とかでよく見る `require('lodash/array')` みたいな読み込み方ができません。そこでこれを解決するために `modules.root` というフィールドを利用します。
-->

```javascript
// package.json
// CommonJS with directory exceptions
{
  "main": "index.js",
  "module": "module.js",
  "modules.root": "lib" // all files loaded within this directory will be ES Modules
}
```

<!--
上記のように`"modules.root": "lib"` フィールドがあると `lib/*` 以下を `require` から読み込めるようになります。つまり、 ES Modules で書いてあっても `require('lodash/array')` みたいな書き方ができるようになり、ある程度互換性を保てるようになります。
-->

The above example is used to show how to maintain backward compatibility for packages. For older versions of Node.js, `require('foo/bar')` will look for a CommonJS `bar.js` in the root of the package. However, for newer versions of Node.js, the `"modules.root": "lib"` directory will dictate that loading `'foo/bar'` will look for an ES Module at `lib/bar.js`.

<!--
とはいえ、新しいバージョンで `main` も `module` も無い package.json では暗黙的には必ず ES Modules になってしまうので、かなり breaking changes です、その代わり拡張子での解決は要らないので、 `.mjs` などの拡張子を検討する必要はありません。なので、 "Defense of dot js" なわけです。
-->

<!--
## CommonJS と ES Modules 両方対応するなら？
-->

### Supporting both CommonJS and ES Modules

<!--
基本的には ES Modules に傾けるのがこの仕様のポイントですが、人気のあるモジュールはそうはいきません。 ES Modules と CommonJS の両対応させる必要のあるモジュールは存在するでしょう。この時は諦めて transpile して、 ES Modules => CommonJS のファイルも用意しておきます。 transpile した JavaScript も一緒に package に入れておきます。 `main` フィールドと `module` フィールドを書いておけば古い Node.js と新しい Node.js で読み込み先を変えてくれます。
-->

Under most proposals, including the Node.js EP and _In Defense of .js_, it is assumed that packages wishing to provide support for old and newer versions of Node.js will use a transpilation mechanism. Under the `.mjs` solution, the ES Modules would be transpiled to `.js` files next to their originals and the different versions of Node.js would resolve to the right file. Under _In Defense of .js_, the ES Modules would exist under a subdirectory specified by `"modules.root"` and be transpiled to CommonJS forms in the parent directory; additionally, `package.json` would have both `"main"` and `"module"` entry-points.

<!--
## Bradley Meck仕様との違い
-->

## Hard choices

<!--
<b>Defense of dot js 側は将来的に ES Modules に全て揃えよう</b> という姿勢です。互換性をある程度損なっているし、それが起こす混乱はある程度受け入れる考えです。それに対して Bradley Meck 氏の仕様は互換性重視です。あくまで今のCommonJSと相互運用性を取るというのを主軸に添えて語られています。拡張子で ES ModulesなのかCommonJSなのかが変わるというのは言い換えれば一つ一つのファイル単位で ModuleなのかCommonJSなのかを切り替えられる柔軟な仕様です。過渡期は `.mjs` と　`.js` が混じるかもしれませんが、将来的にユーザーの判断でどっちがデファクトになるかによっては `.mjs` だけ残る可能性はあります。
-->

_In Defense of .js_ presents a view that we need to switch to ES Modules from CommonJS and prioritizes such a future. On the other hand, the Node.js EP prioritizes compatibility and interoperability.

<!--
これに対してさらに Counter で Bradley Meck 氏はブログを書いています。
-->

<!--
なんで `.mjs` を選んだのか、という理由が書いてあります。
ブログの基本的な論調としてはみんな `.js` という拡張子に頼りすぎているという話です。
-->

<!--
JavaScript には様々な"方言" があります。CommonJS, UMD, AMD、これらの全ての JavaScript は全て `.js` になってます。
-->
<!--
ES Modules というのはもうそれ単体で評価可能な Script ではないし、パースして他のファイルをロードしてチェックするという処理が必要な以上、もういっそ拡張子ごと変えるというアイデアも分からなくはないです。しかも Module だと暗黙的に strict mode で動くので、普通のScript とは別物だと思ったほうがいいです。
-->

Bradley recently wrote a [post](https://medium.com/@bradleymeck/understanding-the-hard-choice-1ea3008fc9d0#.3gryiqkmv
) attempting to further explain the difficult choice and why a file extension was an appropriate way forward. In it, he goes into further details about why it is not possible to parse a file to determine whether it is an ES Module or not. He also further explores the difficulties of having an out-of-band descriptor (e.g. `package.json`) determine what type of content is in a `.js` file.

<!--
他の言語の例を挙げるなら、 `.pl` （Perl スクリプト） と `.pm` （Perl モジュール）のように拡張子を変える事を是とする文化もあります。また、ファイルを開く前に拡張子だけ見れば Module  なのか Script なのかが分かるので、人間にとっても意味はあります。
-->

Although it may be sad to consider the loss of a universal `.js` file extension, it's worth noting that other languages have already paved this path. Perl for instance uses `.pl` for Perl Script, and `.pm` for Perl Module.

<!--
この仕様はまだまだ議論中です。また今週の TSC ミーティングで議論があるでしょう。先週もありましたが、話はまとまりきらずに終わりました。今のうちに何かこうしたいという意志があれば [issue](https://github.com/nodejs/node-eps/issues/13) か [PR](https://github.com/nodejs/node-eps/pulls) に書くことをおすすめします。
-->

## Getting involved

Even though the Node.js CTC has accepted the EP in its current form and stated its preference on how ES Modules would be implemented in Node.js (if they are implemented in Node.js at all), discussion continues and there is still room for change. You can engage with the Node.js community on this topic in the [Node.js EP repository issues list](https://github.com/nodejs/node-eps/issues). Be sure to first review existing comments to see if your concerns have already been addressed.

Bradley and the Node.js CTC are very concerned about getting this decision right, in the interests of Node.js users everywhere. The choices that Node.js is having to make to accommodate ES Modules are difficult and are not being approached lightly.

<!--
# まとめ
-->

<!--
- ES Modules の現時点の状況
- ES Modules を Node.js はどうするのか
- ES Modules on Node.js Proposalの詳細
- Counter Proposal である Defense of dot js の話
-->
