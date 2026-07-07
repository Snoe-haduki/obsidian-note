---
aliases:
  - Nimでできること
  - Nimバックエンド
tags:
  - nim
  - backend
  - ffi
  - systems-programming
updated: 2026-07-07
---

# Nim 用途とバックエンド

## Nimで作れるもの

Nimはネイティブ実行ファイルやライブラリの作成に使える。

代表的な用途は次の通り。

- CLIツール
- サーバーサイドプログラム
- ネイティブアプリケーション
- システムプログラミング
- 組み込み・リアルタイム寄りの処理
- C/C++資産を利用するプログラム
- JavaScriptを出力するWebフロントエンド
- Node.js向けプログラム
- 開発補助ツールや自動化

## 主要バックエンド

Nim公式のBackend Integration資料では、主要バックエンドを大きく2系統に分けている。

### C系バックエンド

- C
- C++
- Objective-C

C系ターゲットでは、Nimコンパイラがソースコードを生成し、最終的にライブラリまたはネイティブ実行ファイルを作成できる。

基本コマンドは次の通り。

```bash
nim c main.nim
nim cpp main.nim
nim objc main.nim
```

通常の `nim c` はCバックエンドを使用する。

## JavaScriptバックエンド

JavaScriptターゲットでは `.js` ファイルを生成できる。

```bash
nim js main.nim
```

生成物をHTMLから参照する用途や、Node.js向けのスタンドアロンプログラムに利用できる。

ただし、JavaScriptプラットフォームが対応しないNim機能・モジュールは利用できない。Cバックエンドと完全に同じ能力を持つわけではない。

## 外部言語との連携

Nimの重要な特徴の一つがバックエンド言語との相互運用である。

C/C++コードのコンパイルやリンク、外部シンボルの取り込み、コンパイラ・リンカへのオプション受け渡しなどに利用するpragmaが用意されている。

代表例：

- `importc`
- `importcpp`
- `compile`
- `link`
- `passc`
- `passl`

つまり、既存のC/C++ライブラリをNimから利用する設計を取りやすい。

逆方向に、生成されたC/C++コードやNim側の機能を別環境へ組み込む選択肢もある。

## Nimを全部自前で完結させる必要はない

Nimの強みは「純Nimだけで世界を閉じること」だけではない。

既存C/C++資産を利用しながら、アプリケーションロジックやツール部分をNimで記述する構成も現実的である。

私の用途で考えると、ゲームエンジン、暗号ライブラリ、OS、LLM関連の研究では、この相互運用性が重要になる。

## 関連

- [[Nim 概要と特徴]]
- [[Nim メモリ管理]]
- [[Nim 開発環境とコンパイル]]

## Sources

- https://nim-lang.org/docs/backends.html
- https://nim-lang.org/docs/nimc.html
- https://nim-lang.org/docs/manual.html
