---
marp: true
paginate: true
---

<!-- _paginate: false -->

# 数値順で並び替えしたい！ 罠と対策 <!-- fit -->

2023/10/06 [Kyoto.js 20](https://kyotojs.connpass.com/event/296322/)
小原 一哉

---

## 自己紹介

- 小原 一哉 (こはら かずや)
- ウェブエンジニア
- フェンリル株式会社
- X (Twitter): [@KoharaKazuya](https://twitter.com/KoharaKazuya)

![bg cover right](./koharakazuya.png)

---

## 数値順で並び替えをしたい

ファイル名などの部分的に数値を含む文字列を扱っているとき、数値部分を数値として認識しつつ昇順などに並び替えたい、ということがある

```
data0.json       data0.json
data1.json       data1.json
data10.json  ->  data2.json
data2.json       data9.json
data9.json       data10.json
test.json        test.json
```

※ 単純に文字列の辞書順だと左にようになってしまう

---

## `Array.prototype.sort`

JavaScript で並び替えをするには `Array.prototype.sort` という関数がある。

**⚠ sort の有名な罠**

```javascript
[0, 1, 2, 9, 10].sort(); // → [0, 1, 10, 2, 9]
```

引数なしで呼び出すと中身を文字列にしてから UTF-16 コード単位の値で昇順になる。

---

## 並び替えのカスタマイズ

`Array.prototype.sort` には引数として比較関数を与えられるので、カスタマイズ。

```javascript
[0, 1, 2, 9, 10].sort((a, b) => a - b); // → [0, 1, 2, 9, 10]
```

---

## 文字列はどうするか？

`data9.json` みたいに部分的に数値を含む場合はどうする？

→ `String.prototype.localeCompare` が使える！

---

## `String.prototype.localeCompare` とは

文字列と文字列を比較して前か後か、または同じかを示す数値を返す (= sort の比較関数の返り値として求められる挙動)

Intl.Collator に対応している環境の場合、細かなオプションが追加で使える

→ `numeric` オプション！

```javascript
["a1x", "a10x", "a2x", "a1y"].sort((a, b) =>
  a.localeCompare(b, undefined, { numeric: true })
);
// ['a1x', 'a1y', 'a2x', 'a10x']
```

---

## `Intl.Collator` が使えない環境では？

出現するすべての数値を十分長い固定の桁数になるように 0 埋めしてから比較する

```javascript
["a1x", "a10x", "a2x", "a1y"].sort((a, b) => {
  const norm = (s) => s.replace(/\d+/g, (m) => m.padStart(10, "0"));
  return norm(a).localeCompare(norm(b));
});
// ['a1x', 'a1y', 'a2x', 'a10x']
```

---

## まとめ

- `Array.prototype.sort` の罠に注意
- `String.prototype.localeCompare` という関数があるよ
- `Intl.Collator` に `numeric` というオプションがあるよ
- 固定桁数になるように 0 埋めするという手もあるよ
