---
marp: true
paginate: true
---

<!-- _paginate: false -->

# 画像をサスペンドするというアイデア

2025/01/15 [クラスメソッドの React 事情大公開スペシャル#6](https://classmethod.connpass.com/event/339885/)
小原 一哉

---

## 自己紹介

- 小原 一哉 (こはら かずや)
- ウェブエンジニア
- フェンリル株式会社
- X: [@KoharaKazuya](https://twitter.com/KoharaKazuya)

![bg cover right](./koharakazuya.png)

---

## 疑問: Web アプリとスマホアプリって見分けられる？

アクセスまでの色々な差があるが、一旦そこは置いておくと
**アプリの中を回遊している間、ユーザーにとって違いはあるのか？**

→ ある。データ読み込み中に対する表示方針の違い。

Web は読み込み中のデータをなるべくユーザーに届けようとして **中途半端な状態でも表示する** 。

---

## ロード中でもなるべく表示しようとする方針の功罪

- メリット: 届けたい情報のコア（ほとんどがテキストデータで先に読み込まれる）が、早くユーザーに届けられる。
- デメリット: 画面がガチャガチャする。**ユーザーに「不安定」な印象を与える** 。

---

![bg contain](./image-loading-1.png)

---

![bg contain](./image-loading-2.png)

---

## つまり CLS の話？

CLS の話ではありません。

（先程のスクリーンショットの例だと、CLS は発生していないが画像の読み込み中状態は表示されている）

---

## 重いアセットの読み込みも待ちたいケースもある

Web は基本的に重いアセット読み込みを待たせない方針を優先してて、これは素晴らしい方針。ほとんどのサイトにおいて先程の例のような挙動が理想的だが、**画像の読み込み完了を待って完成した画面をパッと表示したいシーンがある** 。

---

## 画像の読み込みの完了を待つには？

→ まずは全画面にカバー（スプラッシュスクリーンなど）。その後、画面内の全画像の読み込み状態を管理し、すべて完了すればカバーを外すなど。

---

_画面内の全画像の状態管理？ うん、面倒くさいね_

---

## 救世主 React Suspense

[React ドキュメント](https://ja.react.dev/reference/react/Suspense) より

> `<Suspense>` を使うことで、子要素が読み込みを完了するまでフォールバックを表示させることができます。

これで画像の読み込みを管理すれば手軽に実現できる！

---

## やってみた例

<https://www.youtube.com/watch?v=T8OJiJsuh20&t=3s>

---

## 仕組み

- 「読み込み完了を待ちたい画像」のコンポーネントを作る
- キャッシュ管理しつつ、まだ未読み込みの画像を表示するときは読み込みを開始する
- 読み込みを完了するまでの `Promise` を作る
- 画像の読み込みは `HTMLImageElement` の `onload`, `onerror` を使う
- 画像の読み込みは `Promise.race` を用いてタイムアウトをつけてタイムアウトしたら諦めて通常通り空白の画像状態で表示させる
- 読み込み中は `Promise` と `use` などでサスペンドする
- Tanstack Query などを使うとキャッシュ管理、Promise、サスペンドなどの管理が楽になる ([実例](https://github.com/KoharaKazuya/internet-news-agent/blob/284062b0528e3cec21543487d5594e15bb1dce4d/web/app/queries/image.ts))

---

## 注意

もちろん、すべての画像に対してサスペンドすべきではなく対象を慎重に検討する必要がある。`<img loading=...>` と同様の話。

---

## まとめ

- Web アプリは中途半端な状態を表示する
- 中途半端な状態をなるべく避けたいケースがある
- 画像の読み込み完了を待ってから画面を表示するには React Suspense が便利
