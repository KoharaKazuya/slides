---
marp: true
paginate: true
---

<!-- _paginate: false -->

# SWC の制約から見る、TypeScript のオプション

2024/09/20 [TypeScript Meet Up #3 - Osaka.ts](https://kobets.connpass.com/event/324480/)
小原 一哉

---

## 自己紹介

- 小原 一哉 (こはら かずや)
- ウェブエンジニア
- フェンリル株式会社
- X: [@KoharaKazuya](https://twitter.com/KoharaKazuya)

![bg cover right](./koharakazuya.png)

---

## 今回のイベントのテーマ

- メインテーマ: **TypeScript** とその周辺知識
- サブテーマ: **Vercel** についてなんでも

_(イベントページより)_

---

## 今回話すこと

SWC の TypeScript に関する制約から見る、TypeScript の様々なオプション

---

## SWC とは

SWC は乱暴に言ってしまうと Rust 製の速い Babel 代替品。

**TypeScript** をトランスパイルして JavaScript を出力する機能も持つ。

---

## SWC は Vercel のプロダクト？

SWC はコミュニティードリブンのプロジェクト ([README](https://github.com/swc-project/swc/blob/85cc2bd79c3193cb0a8b54e4fce0efc1aa15b271/README.md) より)。

しかし、作者の [kdy1](https://github.com/kdy1) は **Vercel** 所属の人。また、公式サイトのフッターには `Powered by Vercel` とある。

Next.js の次期コンパイラ Turbopack は SWC を内部で使っている。

---

## SWC の TypeScript に関する制約

SWC は TypeScript を JavaScript にトランスパイルできるものの型チェックはしないなど、本家 TypeScript Compiler (tsc) の一部機能しか扱わない。

フルスペックの TypeScript は扱えず、いくつか制約が必要なので以後詳しく見ていく。
また、合わせて TypeScript コンパイラオプションについても見ていく。

制約については [Migrating from tsc – SWC](https://swc.rs/docs/migrating-from-tsc) より。

---

## isolatedModules: true

SWC はファイルごとに独立してコンパイラを動かす = 別ファイルに依存した機能は動かない。これはおそらく高速化のため。

別ファイルに依存した機能を使っていないことを保証するため、TypeScript コンパイラオプション `isolatedModules: true` が必要。

### isolatedModules オプション

単一ファイルのトランスパイルプロセスでは正しく解釈できない特定のコードを記述した場合に警告させるオプション。

<https://www.typescriptlang.org/tsconfig/#isolatedModules>

---

## importsNotUsedAsValues: "error"

SWC は前述の理由により、import されたものが型か値を区別できない。

```typescript
import { Example } from "./example"; // ← Example は型？ クラスなどの値？
```

TypeScript コンパイラオプション `importsNotUsedAsValues: "error"` を使用すると、すべての型インポートに `type` をつけることを強制できる。

### importsNotUsedAsValues オプション

非推奨。後述の verbatimModuleSyntax を使おう。

---

## esModuleInterop: true

TypeScript は歴史的経緯からインポートの相互運用性に関して問題がある。SWC は Babel と同じアプローチを取るので、それを伝えるため TypeScript コンパイラオプション `esModuleInterop: true` が必要。

### esModuleInterop オプション

乱暴に言うと CommonJS で書かれたライブラリを

```typescript
import _ from "lodash"; // 定義側で default は定義していない
_.chunk(["a", "b", "c", "d"], 2);
```

のように使うために必要。

---

## verbatimModuleSyntax: true

TypeScript コンパイラオプション `isolatedModules`, `preserveValueImports`, `importsNotUsedAsValues` を置き換え、シンプルにするために導入されたもの。

SWC は実行時に不要な型のみの import をしないようにするため、型のみの import 文などはなるべく削除したいが、どれが型かわからない。

### verbatimModuleSyntax オプション

`verbatim`: 逐語的に

型のみの import をしているときは type をつけることを強制し、トランスパイル時に type がついているものは削除する。

---

## useDefineForClassFields

TypeScript はコンパイラオプション `useDefineForClassFields` の設定により、`class A { field = 0 }` のようなコードのトランスパイル結果が異なる。

SWC は TypeScript と同じ出力をするため `useDefineForClassFields` の値を知る = 指定する必要がある。クラスを使用しないか継承を使用しなければ気にしなくてよい。

### useDefineForClassFields オプション

TypeScript は以前からパブリッククラスフィールドの構文を許容しておりナイーブな意味で捉えていたが、その後の ECMAScript の標準化過程で想定外のほうに転がったっぽい。ECMAScript に合わせるにはトランスパイル結果を変える必要があるためオプションにした。

---

```typescript
// source (TypeScript)
class A {
  field = 0;
}
// output (useDefineForClassFields: false)
class A {
  constructor() {
    this.field = 0;
  }
}
// output (useDefineForClassFields: true)
class A {
  constructor() {
    Object.defineProperty(this, "field", {
      enumerable: true,
      configurable: true,
      writable: true,
      value: 0,
    });
  }
}
```

---

## まとめ

- SWC で TypeScript を使う際の制約を紹介した
- TypeScript コンパイラオプションをいくつか紹介した
