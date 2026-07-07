---
aliases:
  - Nim標準ライブラリ
  - Nimパッケージ管理
tags:
  - nim
  - stdlib
  - nimble
  - package-manager
updated: 2026-07-07
---

# Nim 標準ライブラリとNimble

## 標準ライブラリの分類

Nim 2.2.10の公式Standard Library資料では、ライブラリを次の3種類に分けている。

- pure libraries
- impure libraries
- wrappers

## pure libraries

外部の `.dll` や `lib*.so` バイナリに依存しないライブラリ。

Nim内で完結しやすい標準機能と考えると分かりやすい。

## impure libraries

外部バイナリへの依存を持つライブラリ。

## wrappers

Cライブラリに対する非常に低レベルなインターフェースを提供するimpure library。

Nimから外部Cライブラリを扱うための橋渡しに近い。

## `system`

`system` モジュールはコンパイラによって暗黙にimportされる。

```nim
echo "Hello"
```

このコードで `import system` を書く必要はない。

`system` は基本的なprocやoperator、入出力機能など、Nimプログラムの土台を提供する。

公式資料は `system` を直接importしないよう説明している。

## 標準ライブラリの立ち位置

Nim公式Standard Library資料は、標準ライブラリが基本機能を担当し、第三者パッケージについてはNimble Directoryを確認するよう案内している。

つまり基本的な考え方は次の通り。

```text
標準ライブラリ
↓
足りない機能が明確になる
↓
Nimbleで外部パッケージを追加
```

最初から外部パッケージを大量導入するより、まず標準ライブラリを確認する方が依存関係を抑えやすい。

## Nimble

NimbleはNimのデフォルトのパッケージマネージャである。

外部パッケージの導入だけでなく、Nimbleパッケージの作成やdevelop workflowにも使われる。

典型的には次のようなコマンドを利用する。

```bash
nimble install パッケージ名
```

プロジェクトやパッケージでは `.nimble` ファイルが重要になる。

## パッケージ一覧

Nimの外部パッケージを探す入口としてNimble Directoryがある。

- https://nimble.directory/

また、`nim-lang/packages` はNimble用パッケージの中央一覧として機能する。

- https://github.com/nim-lang/packages

## 外部パッケージ利用時の注意

`nim-lang/packages` のREADMEは、掲載パッケージがpeer reviewやscreeningを受けているわけではなく、品質や成熟度を保証できないと明記している。

したがって、パッケージを選ぶときは最低でも次を確認する。

- 最終更新
- 対応Nimバージョン
- issueの状態
- release/tag
- READMEと使用例
- テストの有無
- 依存パッケージ
- ライセンス

暗号、ネットワーク、認証、ファイル形式など安全性に関わる用途では、単に「Nimbleにある」ことを採用理由にしてはならない。

## 私の整理

Nim初心者はまず `system`、`strutils`、`sequtils`、`os` など主要標準モジュールに慣れる。

その後、実際のプロジェクトで必要になった機能だけNimbleから追加する。

この順序がNim Package Wikiや100日ドリルの教材構成とも相性がよい。

## 関連

- [[Nimble]]
- [[Nim Package Wiki]]
- [[system]]
- [[strutils]]
- [[sequtils]]
- [[os]]

## Sources

- https://nim-lang.org/docs/lib.html
- https://nim-lang.org/docs/system.html
- https://github.com/nim-lang/nimble
- https://github.com/nim-lang/packages
- https://nimble.directory/
