---
marp: true
paginate: true
---

<!-- _paginate: false -->

# dedent とテンプレートリテラルを理解しよう

2025/01/24 [kansai.ts #9](https://kansaits.connpass.com/event/338795/)
小原 一哉

---

## 自己紹介

- 小原 一哉 (こはら かずや)
- ウェブエンジニア
- フェンリル株式会社
- X: [@KoharaKazuya](https://twitter.com/KoharaKazuya)

![bg cover right](./koharakazuya.png)

---

## 状況設定

みなさん、JavaScript でこんなことありませんか？

```javascript
function hello() {
  const message = `* インデント付きのメッセージ
  1. リストA
  2. リストB
     * ネスト
* アイテムその2
`;
  console.log(message);
}
```

**改行含む文字列を埋め込むとき、ソースコードの見た目が汚い！**

---

## dedent という概念

_indent を解除する_ という意味合いの **dedent** というツールがある（おそらく造語）

Python の組み込み関数らしい…。

---

```javascript
function hello() {
  const message = dedent`
    * インデント付きのメッセージ
      1. リストA
      2. リストB
         * ネスト
    * アイテムその2
  `;
  console.log(message);
}
```

↓

```txt
* インデント付きのメッセージ
  1. リストA
  2. リストB
     * ネスト
* アイテムその2
```

---

## dedent 系の JavaScript ライブラリ

- [dedent](https://www.npmjs.com/package/dedent)
- [ts-dedent](https://www.npmjs.com/package/ts-dedent)
- [multiline-ts](https://www.npmjs.com/package/multiline-ts)
- [@qnighy/dedent](https://www.npmjs.com/package/@qnighy/dedent)

[他にもいろいろある](https://www.npmjs.com/search?q=keywords:dedent)

---

## テンプレートリテラルとは

そもそも ` `` ` で囲むのって？ → JavaScript の [テンプレートリテラル](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals)

改行文字と埋め込み式が扱えるようになった新しい文字列リテラル
<small>_（というような説明はおそらく正しくないがそう言う認識で大丈夫なはず）_</small>

```javascript
const n = 1;
const message = `1行目
2行目
${n * 10}
`;
```

---

## タグ付きテンプレートリテラルとは

` `` ` の前にくっついてるのって？ → [タグ付きテンプレート](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals#%E3%82%BF%E3%82%B0%E4%BB%98%E3%81%8D%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88)

タグ関数 = テンプレートリテラルの前にくっついている関数 = テンプレートリテラルと埋め込み式の値を受け取って新しい値を返す関数

```javascript
const doc = html`<!DOCTYPE html>
  <html>
    ${body}
  </html>`;
```

タグ付きテンプレートによって独自の構文を JavaScript 中に埋め込んだりできる

---

## タグ付きテンプレートリテラルの特殊なところ (1/2)

- タグ関数は文字列以外を返してもよい
- タグ関数の第 1 引数 = テンプレートリテラルを含んだオブジェクトは凍結されて与えられる
  - リテラル = ソースコードに書かれているものが与えられるので変更する意味がない → 変に破壊されないように
  - 埋め込み式を処理する前の処理などをキャッシュできる

---

## タグ付きテンプレートリテラルの特殊なところ (2/2)

- タグ関数はエスケープシーケンスを処理済み = 通常の文字列と、処理前 = ソースコードに書かれたそのままの文字列を受け取れる
  - `` tag`\u{0041}` `` → `A` & `\u{0041}`
  - 処理前の文字列をそのまま取得したいとき → `String.raw`
- テンプレートリテラルと違い、エスケープシーケンスで不正があってもエラーにならない
  - 任意の構文を扱えるようにするため
  - 構文エラーを検知したければタグ関数中に実装すればよい

---

## タグ関数のシグネチャ

```typescript
// lib.es5.d.ts
interface TemplateStringsArray extends ReadonlyArray<string> {
  readonly raw: readonly string[];
}

// 多分こんな感じ
type TagFunction = (
  // 埋め込み式で区切られたテンプレートリテラルが順に入ってる配列
  strings: TemplateStringsArray,
  // 残りの引数に埋め込み式の値が順に入っている
  ...args: unknown[]
) => unknown;
```

---

## 色々な実装 (1/4): `dedent`

- 利用者が多い
- 独自のエスケープシーケンス処理（変な感じ）
  - <small>一部の文字をエスケープシーケンスを処理し、一部はしない不思議な特殊処理が埋め込まれている。おそらく作者がかつてエスケープシーケンス処理周りの理解が不十分だった名残</small>
- メンテが続いてそう
- 2 年前まで TypeScript 型定義が含まれていなかったので代わりに `ts-dedent` が使われることがあった

npm: [dedent](https://www.npmjs.com/package/dedent)

---

## 色々な実装 (2/4): `ts-dedent`

- 利用者が多い
- シンプルな実装
- メンテが止まってそう
  - <small>依存もないし完成してるのであまり気にならない</small>

npm: [ts-dedent](https://www.npmjs.com/package/ts-dedent)

---

## 色々な実装 (3/4): `multiline-ts`

- シンプルな実装
- メンテが続いてそう

npm: [multiline-ts](https://www.npmjs.com/package/multiline-ts)

---

## 色々な実装 (4/4): `@qnighy/dedent`

- 独自のエスケープシーケンス処理
  - <small>標準の処理を再現しているだけ & 標準で変わることない部分だろうから気にならない</small>
- 不正なエスケープシーケンスのレポートあり
- エレガントな機能 & API
  - 他のタグ関数との組み合わせが可能
  - 先頭改行、末尾改行などの扱いが素朴
- 最適化がくっついてくる
- メンテが止まってそう
  - <small>依存もないし完成してるのであまり気にならない</small>

npm: [@qnighy/dedent](https://www.npmjs.com/package/@qnighy/dedent), Zenn: [テンプレートリテラルよもやま話 & dedent ライブラリ](https://zenn.dev/qnighy/articles/14f4ecdbcb18c3)

---

## 個人的おすすめ

ここまでに紹介したものを含むいくつかの実装を読んだ感想より、

- 実装のシンプルさと利用者数の多さから生まれる安定性を求める → `ts-dedent`
- 応用力の高さと原理主義的スマートさを求める → `@qnighy/dedent`

---

## まとめ

- dedent という概念
- テンプレートリテラルとタグ付きテンプレートリテラル
- さまざまな dedent の実装と個人的おすすめ
