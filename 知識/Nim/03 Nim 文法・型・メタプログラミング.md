---
aliases:
  - Nim文法
  - Nimメタプログラミング
tags:
  - nim
  - syntax
  - type-system
  - metaprogramming
  - macro
updated: 2026-07-07
---

# Nim 文法・型・メタプログラミング

## 文法の基本的な印象

Nimはインデントを重視する構文を持つ。

見た目はPythonに近い印象を受けやすいが、Nimは静的型付けのコンパイル言語である。

```nim
type
  User = object
    name: string
    age: int

proc greet(u: User): string =
  "こんにちは、" & u.name & "さん"

let user = User(name: "Aoi", age: 20)
echo greet(user)
```

コードのブロック構造をインデントで表現し、型と手続きを比較的素直に記述できる。

## `var` と `let`

### `var`

再代入可能な変数。

```nim
var count = 0
count = 1
```

### `let`

単一代入のローカル変数。

```nim
let name = "Nim"
```

値を書き換える必要があるかどうかを宣言時に分けられる。

## 主な構成要素

- `proc`：プロシージャ
- `func`：副作用制約を持つ関数
- `iterator`：反復処理
- `type`：型定義
- `const`：コンパイル時定数
- `template`：構文的なコード生成・抽象化
- `macro`：ASTを操作するメタプログラミング

初心者は最初から全部覚える必要はない。

まずは `var`、`let`、`proc`、`type` を中心に学び、その後 `iterator`、ジェネリクス、template、macroへ進む方がよい。

## 型

`type` セクションでは `object` や `tuple` を定義できる。

```nim
type
  Point = object
    x: int
    y: int
```

Nimはobject inheritanceやobject variantsも扱える。

object variantsは判別子を利用して、データの種類ごとに異なるフィールドを持たせる仕組みである。

## コンパイル時実行

Nimでは、一部の処理をコンパイル時に実行できる。

コンパイル時評価はNimの重要な設計要素であり、ユーザー定義関数の評価やコード生成と組み合わせられる。

## template

`template` はコードの構造を抽象化するための機能である。

通常の `proc` とは異なり、呼び出し部分へ展開される構文的な仕組みとして利用できる。

同じコードパターンをまとめたい場合や、通常の関数だけでは表現しづらい構文的抽象化に向く。

## macro

Nimのmacroはコンパイル時に実行され、NimのASTを変換する。

概念的には次の流れになる。

```text
Nimコード
↓
AST
↓
macroがASTを受け取る
↓
ASTを変換
↓
変換後のコードをコンパイル
```

単なる文字列置換ではない。

`NimNode` とノード種別を扱い、言語の構造を理解した上でコードを生成・変換できる。

用途例：

- ボイラープレート削減
- DSL風API
- 宣言からコードを自動生成
- コンパイル時検査
- 定型コードの安全な展開

## 学習方針

Nimのマクロは強力だが、初心者が最初から使う必要はない。

まず通常のNimコードを十分に読めるようにする。その後templateを理解し、最後にASTとmacroへ進む方が安全である。

## 関連

- [[Nim 概要と特徴]]
- [[Nim マクロ]]
- [[Nim AST]]
- [[Nim 100日ドリル]]

## Sources

- https://nim-lang.org/docs/manual.html
- https://nim-lang.org/docs/tut3.html
- https://nim-lang.org/docs/macros.html
