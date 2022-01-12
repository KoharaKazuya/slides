---
marp: true
paginate: true
---

<!-- _paginate: false -->

# なぜ WebAssembly (+ Rust) に注目するのか、なぜ実用できていないのか <!-- fit -->

2022/01/13 TSC 定例
小原 一哉

---

## 前書きとアウトライン

この資料では私がなぜ WebAssembly (と Rust) に注目しているのかと注目しているのになぜ実用できていないのかについて説明します。

前半で WebAssembly と Rust の軽い紹介と注目したいと思える特徴について、後半で実用に躊躇しているポイントについて考えを述べます。

私が WebAssembly について一番頑張って調べたのが [2017 年に Qiita に記事を投稿した](https://qiita.com/KoharaKazuya/items/593aa8efbf743b20ec28) ときなので古い情報が含まれているかもしれません。

---

## WebAssembly と Rust の紹介 <!-- fit -->

---

## WebAssembly とは

<https://webassembly.org/> より

> WebAssembly（略して Wasm）は、スタックベースの仮想マシン用のバイナリ命令フォーマットです。Wasm は、プログラミング言語のポータブルコンパイルターゲットとして設計されており、クライアントおよびサーバーアプリケーションの Web での展開を可能にします。(訳注: 機械翻訳)

一言で言うなら「新しい VM」。初学者にわかりやすい表現は「JavaScript の代わりに配るバイナリ」「Java アプレットのリブート」。

Java バイトコード、LLVM IR と同様、直接書くようなものではない (はず)

---

## WebAssembly の特徴 (1/2)

<https://webassembly.org/> より

1. 効率的で高速
2. 安全
3. ソースを開いてデバッグ可能
4. オープンウェブプラットフォームの一部

---

## WebAssembly の特徴 (2/2)

### 効率的で高速？

そもそものニーズ。テキストのソースコード (JavaScript) を配るよりバイナリ形式の方がサイズとロード時間が小さくなるはず。一般的なハードウェア機能を利用してネイティブ速度で実行できるように目指している。

### 安全？

ブラウザのネイティブ拡張を捨てざるを得なかった歴史を踏まえて設計されているため、デフォルトで隔離されている。

参考: [出来ることは計算だけ？「WebAssembly」は一体なにが新しいのか〜エンジニアが語る技術愛 ＃03〜｜ミクシル](https://mixil.mixi.co.jp/people/12242)

---

## WebAssembly の利用例

- Google Meet (参考: [なぜ Google Meet の背景ぼかしが最強なのか（一般公開版）](https://zenn.dev/kounoike/articles/google-meet-bg-features))
- Google Earth (参考: [(WebAssembly のプロダクション利用をまとめたサイト)](https://worldofwasm.bubbleapps.io/))
- Figma (参考: [(WebAssembly のプロダクション利用をまとめたサイト)](https://worldofwasm.bubbleapps.io/))
- Photoshop (参考: [Photoshop's journey to the web](https://web.dev/ps-on-the-web/))
- [sql.js](https://sql.js.org/)
- [FFMPEG.WASM](https://ffmpegwasm.netlify.app/)
- こえのブログ (参考: [アメブロ 2019: こえのブログでの PWA | CyberAgent Developers Blog](https://developers.cyberagent.co.jp/blog/archives/20506/))
- Amazon IVS (最も使われている Wasm ライブラリ) (参考: [WebAssembly | 2021 | The Web Almanac by HTTP Archive](https://almanac.httparchive.org/en/2021/webassembly))
- Unity WebGL (参考: [WebAssembly が WebGL ビルドのスタンダードに！ | Unity Blog](https://blog.unity.com/ja/technology/webassembly-is-here))

(全部リンク先からの伝聞。実際に調べてないので現在はどうなっているか不明)

---

## Rust とは

<https://www.rust-lang.org/ja> より

> 効率的で信頼できるソフトウェアを誰もがつくれる言語

新しめのプログラミング言語。パフォーマンスが意識された設計で、ランタイムやガーベッジコレクターがなく高速。型システムと所有権モデルによりメモリ安全性とスレッド安全性が保証される。エコシステム、周辺ツール群も意識されてサポートされているため色々揃ってる。

**Wasm を出力できる言語の中でも相性がよい** (後述)。

C, C++ の後継というわけではないがたぶんユースケースが最も近いのはそっちあたり。JavaScript, Go が中心の私にとっては新しい概念が多く習得が非常に難しく感じる。

---

## なぜ Wasm と Rust に注目するのか <!-- fit -->

---

## なぜ Wasm に注目するのか？ (1/2)

- 増加する JS サイズ: 2018 → 2021 で 19 % 近くの増加 ([Web Almanac 2021](https://almanac.httparchive.org/ja/2021/javascript) より)
- もっと高機能に、もっと速く、というニーズ
  - Google のオフィス向けツール, Slack, Miro, GitHub Codespaces など、以前ならデスクトップアプリしか考えられなかったものが Web で提供されるのが当たり前に
  - 実用に耐える速さの壁を越えられるようになると、今までできなかったことができるように (動画編集 on Web など)
  - UX がビジネスインパクトを持つという当たり前の事実 & [Google の検索ランキングスコアに速さが含まれるように](https://developers.google.com/search/blog/2020/05/evaluating-page-experience?hl=ja)

(「検索する」という行動様式が変わらない限りおそらく) Web の高機能化の流れは止まらない & UX は常に重要なので高機能化に伴う高速化の需要は高まっていくはず。

---

## なぜ Wasm に注目するのか？ (2/2)

- Web 以外の資産の利用: FFMPEG.WASM など
- Web 以外の利用シーン、新しい VM としての可能性
  - 「安全に任意コードを実行できる環境」としてのポテンシャルの高さ、実績 (ブラウザ上)、使いやすさ (中間表現)
  - Fastly Compute@Edge, Proxy-Wasm
  - [WASI](https://hacks.mozilla.org/2019/03/standardizing-wasi-a-webassembly-system-interface/)
- 言語の壁の突破、大統一言語: Java が見た夢をもう一度

もちろん Web での利用が中心になるはずだけど、Web 以外の利用も考えられる。私はとくに CDN エッジロケーションで動かすスタイルが流行ると思っていた (過去形)

---

## (Wasm を考えるとき) なぜ Rust に注目するのか？

WebAssembly を出力できる手段は多くあるが、Rust はとくに相性が良いように思える。

- Rust 公式でのビルドサポート ([Tier 2](https://doc.rust-lang.org/rustc/platform-support.html))、整備されたツール群
- ランタイム (ガーベッジコレクター含む) を持つ言語は、ランタイムそのもののサイズが Wasm の「小さいサイズ」というメリットが薄れさせてしまう
- 抽象度が高く書きやすい文法やライブラリ

---

## なぜ Wasm と Rust を実用できていないのか <!-- fit -->

---

## なぜ Wasm を実用できていないのか？ (1/2)

- (私がやってきた仕事の中では) 「速さ」の需要がゼロ
- (仮に「速さ」が要求されたとして) Wasm なしのチューニングの余地があり余っている
- Wasm にしただけでは速くならない、というのが通説
  - JavaScript がそもそも異常に速い
  - みんなが使えば Wasm も最適化されていき速くなる期待はできる
- 実績がないから速くできるかわからない、わからないからコストをかけられない、コストをかけられないから実績が作れない、というジレンマ

---

## なぜ Wasm を実用できていないのか？ (2/2)

- Wasm が効果的なユースケースが特殊
  - Wasm だけだと純粋な計算機: DOM は操作できない、通信できない、デバイスは見えないなど
  - Wasm が外部とやりとりするために JavaScript を経由する必要があるが、オーバーヘッドがある: 回数が多いと逆に遅くなる
  - アプリケーションのうち、計算量が多く画面描画や通信などの頻度が入出力が少ない一部を処理するのに向いている: そういった部分がなければ不要
    - [Rust and WebAssembly の以前のチュートリアルはライフゲームだった](https://rustwasm.github.io/book/game-of-life/introduction.html)
  - 重い処理もデータが小さいならサーバーに送ってそっちでやれば済む

私が作ってきたアプリケーションで (たぶん) 重さの原因のほとんどが DOM 操作、UI ライブラリ周り = Wasm で解決しない領域。

AI 関連は向いてそう。ただ、パッケージングされたものを利用する側に周りそう。

---

## なぜ Rust を実用できていないのか？

- **難しい**: 組織的に習得するビジョンが見えない、挑戦する勇気が持てない
  - 元から触れる人は (狙って採用しない限りは) まずいない & 触れるようになるまで時間がかかる & 人の入れ替わりがある
  - 私はこれまでに「なぜかビルドできない」で完全に詰まることが何度かあった
  - 所有権、ジェネリクス、トレイトあたりが最低限わかればアプリケーションは書けるかも。これらだけでも私は苦労した
  - 私自身はまだアプリケーションを書ける気がしていない

Wasm との組み合わせにおいては気になるところはとくにない

楽観的に見ればなんだかんだで「なぜか動く」で使えるかも……

---

## どのように Wasm と Rust を活用できるか <!-- fit -->

---

## 活用のアイデア

- デスクトップアプリの Web 化
  - 高機能業務アプリのポート
  - Web 以外で書かれた資産のポート (低レイヤーに強い人が必要)
- ロジックをサーバー、クライアント (iOS, Android, Web) で共通化する
  - とくにバリデーションロジックはサバクラ共通で必要になりやすい
  - [Wasmer](https://wasmer.io/) ([iOS サポート](https://wasmer.io/posts/wasmer-2.1)、[Java](https://github.com/wasmerio/wasmer-java))
  - Rust ([[1]](https://dev.to/h_ajsf/building-wasm-android-and-ios-app-with-singlecommon-rust-core-code-3ja4), [[2]](https://medium.com/swlh/rust-cross-platform-mobile-development-9117a67ac9b7))
