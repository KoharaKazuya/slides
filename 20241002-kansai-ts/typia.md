---
marp: true
paginate: true
---

<!-- _paginate: false -->

# Typia が熱い！

2024/10/02 [kansai.ts #8](https://kansaits.connpass.com/event/328683/)
小原 一哉

---

## 自己紹介

- 小原 一哉 (こはら かずや)
- ウェブエンジニア
- フェンリル株式会社
- X: [@KoharaKazuya](https://twitter.com/KoharaKazuya)

![bg cover right](./koharakazuya.png)

---

## Typia が熱い！！！！！ <!-- fit -->

---

## Typia とは

> **Only one line**  
> _No extra schema required_  
> _Just fine with pure TypeScript type_  
> `typia.assert<T>(input);`

<https://typia.io/> より

Typia は TypeScript で型だけ書いておけば、(Zod などと同様に) 実行時の構造チェックを追加のスキーマ宣言なしに実行できるツール。

ユーザー入力の構造が正しいかチェックするときなどに必要となる機能。

---

## サンプル

```typescript
type Pet = {
  name: string;
};

// input が Pet の構造を持っていなければエラー
typia.assert<Pet>(input);
```

---

## 他のアプローチとの比較

以前から Ajv, Zod, Valibot などがあったよね？

→ JSON Schema や独自の API を覚える必要がある。TypeScript と併用する場合、どちらにしろ型定義は必要になるので、スキーマ定義は型定義の文法できるほうが楽。

---

## 仕組み

TypeScript の型情報は実行時には消えてるでしょ。どうやってるの？

→ やはりその辺りは純粋なライブラリとしてだけでは解決できず、バンドラーの処理の拡張を必要としている。コード変換をして型情報に沿ったコードを生成しているはず。

コード変換を必要とする部分は人によっては **メタプログラミングと同様の拡張性と継続性に関する懸念** を感じることもあるはず。

各種バンドラーへの対応 → [unplugin-typia](https://github.com/ryoppippi/unplugin-typia)

---

## Typia の良さ

TypeScript の型定義をいつも通り書くだけで、実行時に「その型通りの構造してる？」というチェックができるようになる！

TypeScript のミッシングピースが埋まった！！！

---

## 実際に使ったシーン

OpenAI API に JSON を取り扱うように指示する際は JSON Schema で JSON の構造を指定する必要がある。

API から返ってきたデータの構造をバリデーションしたいし、TypeScript の型もつけたい。JSON Schema を書くのは面倒…。

---

こういう構造の JSON を取り扱ってほしい

```typescript
export type SuggestFuncArgs = {
  /**
   * 置換対象の直前の文字列。
   *
   * 置換箇所を特定するために使用します。一意に特定するため必要なだけの文字列を指定してください。
   * なるべく短い文字列を指定してください。置換対象の文字列は含まないようにしてください。
   * 完全一致する必要があります。
   */
  anchor: string;
  /**
   * 置換対象の文字列。
   *
   * 完全一致する必要があります。
   */
  target: string;
  /**
   * 置換後の文字列。
   */
  replacement: string;
};
```

---

Typia があればこう！

```typescript
// アサーション関数を生成する
const assertSuggestFuncArgs = typia.createAssert<SuggestFuncArgs>();

// JSON Schema を生成する
const jsonSchemaSuggestFuncArgs = typia.json.application<
  [SuggestFuncArgs],
  "3.1"
>().components.schemas!.SuggestFuncArgs;

// 構造のチェックをして、型がついた値が取得できる
const args = assertSuggestFuncArgs(JSON.parse(func.arguments));
```

TypeScript の型定義さえあれば一瞬でバリデーションや JSON Schema 生成ができる！

JSON Schema に含めないといけない説明文についても TypeScript のコメントから自動的に生成されている。

---

## そう、これが欲しかったんや！！！！ <!-- fit -->

Zod などでスキーマ書いてる時の「まあこうするしかないよな」が解決。

---

## まとめ

- Typia は TypeScript の型情報だけで実行時のバリデーションができるツール
- Typia を使うにはバンドラーとの連携が必要で、unplugin-typia が便利
- Typia は JSON Schema の出力もできる
