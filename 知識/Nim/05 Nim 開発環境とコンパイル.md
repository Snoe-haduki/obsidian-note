---
aliases:
  - Nim開発環境
  - Nimコンパイル
tags:
  - nim
  - compiler
  - development-environment
  - choosenim
updated: 2026-07-07
---

# Nim 開発環境とコンパイル

## 現行安定版

2026-07-07確認時点で、Nim公式サイトが案内する安定版は **Nim 2.2.10**。

2.2.10は2026-04-24に公開されたNim 2.2系列の第5パッチリリースで、公式リリース記事では76 commits分のbugfixとimprovementを含むと説明されている。

## インストール

公式サイトではWindows、Linux、macOS向けの導入方法が案内されている。

Nimは通常、Cバックエンドを利用して最終的なネイティブコードを生成するため、対応するCコンパイラ環境も重要になる。

## choosenim

choosenimを使うとNimのバージョン管理を行いやすい。

Nim 2.2.10公式リリース記事では、既存のchoosenim環境を更新する方法として次のコマンドが案内されている。

```bash
choosenim update self
choosenim update stable
```

同記事ではchoosenim **v0.8.16** の利用が推奨されている。

これは2026-04-24時点の公式リリース記事による情報なので、将来は再確認する。

## 最初のコンパイル

例として `main.nim` を作る。

```nim
echo "Hello, Nim!"
```

コンパイルする。

```bash
nim c main.nim
```

コンパイルして実行する場合は `r` コマンドを使える。

```bash
nim r main.nim
```

リリース向け定義を有効にしてコンパイルする例：

```bash
nim c -d:release main.nim
```

## コンパイラの基本形

Nim Compiler User Guideでは、基本構文を次の形で説明している。

```text
nim command [options] [projectfile] [arguments]
```

主なコマンド：

- `c` / `compile`：デフォルトコードジェネレータでコンパイル
- `r`：コンパイルして実行
- `doc`：ドキュメント生成

## バックエンド選択

```bash
nim c main.nim
nim cpp main.nim
nim objc main.nim
nim js main.nim
```

→ [[Nim 用途とバックエンド]]

## 公式ドキュメントの読み分け

### Language Manual

言語仕様、構文、型、意味論を確認する。

### Compiler User Guide

コンパイラコマンド、スイッチ、クロスコンパイル、最適化、組み込み・リアルタイム関連を確認する。

### Standard Library

標準モジュールとAPIを調べる。

### Official Tutorials

基本構文、OOP、例外、ジェネリクス、template、macroの学習に使う。

### Playground

ブラウザ上でNimコードを試す。

## 初心者向け方針

最初からIDE、巨大なフレームワーク、複雑なビルド設定を揃える必要はない。

まずは次の構成で十分。

```text
エディタ
+ ターミナル
+ Nim
+ Cコンパイラ環境
+ 公式ドキュメント
```

小さな `.nim` ファイルで、出力、変数、条件分岐、ループ、proc、ファイル操作を試す。

その後、Nimbleプロジェクトや外部パッケージへ進む。

## 関連

- [[Nim 概要と特徴]]
- [[Nim 標準ライブラリとNimble]]
- [[Nim 100日ドリル]]

## Sources

- https://nim-lang.org/
- https://nim-lang.org/install.html
- https://nim-lang.org/blog/2026/04/24/nim-2210.html
- https://nim-lang.org/docs/nimc.html
- https://nim-lang.org/documentation.html
