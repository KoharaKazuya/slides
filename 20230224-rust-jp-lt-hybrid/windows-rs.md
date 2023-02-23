---
marp: true
paginate: true
---

<!-- _paginate: false -->

# windows-rs 使ってみた <!-- fit -->

2023/02/24 [Rust LT ハイブリッド #1](https://rust.connpass.com/event/272526/)
小原 一哉

---

## 自己紹介

- 小原 一哉 (こはら かずや)
- ウェブエンジニア
- フェンリル株式会社
- Twitter: [@KoharaKazuya](https://twitter.com/KoharaKazuya)

![bg cover right](./koharakazuya.png)

---

## 動機

「Windows で動く自作の動画プレイヤーを作ってみたい」

---

## 選択肢 (本筋ではないので略)

- Web サイト (PWA): パフォーマンスが悪い、同期的な処理が難しい
- Tauri: (動画を扱う範囲は) 同上
- ネイティブ
  - UWP: GUI を前提としたノウハウが出てくることに苦手意識がある
  - Win32: C++ の API が辛い

※ 最近ちょっと触ったことがあるだけの人の感想です

---

## 選択肢 (本筋ではないので略)

- Media Foundation: コーデックが充実していない。OGG とか。
- DirectShow: 途切れず連続再生みたいなよくあるユースケースをカバーしてない
- GStreamer: DirectShow like なので無用なラッパーな気がしてくる。マルチプラットフォームはとくに求めていない

※ 最近ちょっと触ったことがあるだけの人の感想です

---

Win32 と Media Foundation を使って動画プレイヤーを作るぞ！

↓

C++ の API を使うわけだけど、C++ 覚えるんか？ 辛そう

**Rust で書けます！**

---

## windows-rs

[windows-rs](https://github.com/microsoft/windows-rs) は Windows API のメタデータから生成された Rust のクレート。かなり大部分の Windows API を呼び出すことができ、Rust っぽく書ける。

コード例:

```rust
CreateWindowExA(
    WINDOW_EX_STYLE::default(),
    window_class,
    s!("This is a sample window"),
    WS_OVERLAPPEDWINDOW | WS_VISIBLE,
    CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
    None, None, instance, None,
);
```

---

## どうやって使うのか

- Windows 上で
- windows クレートをインストールし、
- 使いたい API が必要とする feature を有効化したら

あとは普通のクレートの API のように使えば OK

---

## 良いところ

- Rust で書ける
- Rust らしさがある
  - 出力を受け取る変数のポインターを渡す → 返り値
  - null ポインターを与えられる箇所に `Option`
  - COM で `AddRef`, `Release` などを手動でしなくてよい
- 環境構築が楽 (謎の make と戦わなくてよい)
- VS Code でデバッガーのステップ実行もできる

---

## 良くないところ

- (当然だが) ほぼすべての API が unsafe
- ビルドがやたら重い
- features の管理が面倒
- わずかだが含まれていない API や定数がある
- API の検索が重い
- 使っている人が非常に少なく、参考にできる情報がない

---

## その他

- 動画プレイヤーの実装は芳しくない (OGG の MFT がないので実装中)
- Windows の歴史が重い (~~迷走~~ 試行錯誤の結果が見て取れる)
- MSDN のドキュメンテーションが丁寧ですごい……が、必要な知識が多すぎて何を言っているかわからないことが多い

---

## まとめ

- 動画プレイヤーの実装するための選択を紹介した
- windows-rs を紹介した
- windows-rs を触った感想を述べた
