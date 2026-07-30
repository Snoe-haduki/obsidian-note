---
title: Vulkan Tutorial (Introduction) 日本語訳
source: https://qiita.com/shibainuudon/items/6159d3745336f984bd03
author:
  - "[[shibainuudon]]"
published: 2022-06-16
created: 2026-07-09
description: はじめに Vulkanについての資料で、最も参考になると私が思っているのがVulkan Tutorialです。Vulkanについて日本語で書かれた資料は少なく、Vulkan Tutorialについても英語のみとなっていますので、日本語に訳していこうかと思います。 私は翻...
tags:
  - clippings
  - vulkan
---
この記事は最終更新日から3年以上が経過しています。

- [Vulkan](https://qiita.com/tags/vulkan)

14

最終更新日 投稿日 2022年06月16日

## はじめに

Vulkanについての資料で、最も参考になると私が思っているのが [Vulkan Tutorial](https://vulkan-tutorial.com/) です。Vulkanについて日本語で書かれた資料は少なく、Vulkan Tutorialについても英語のみとなっていますので、日本語に訳していこうかと思います。

- 私は翻訳の専門家ではないので、訳に間違いが含まれると思います。その場合はコメント等で指摘していただけるとありがたいです。
- 記事中のリンクは原文のままとしています。つまり英語版のWikipediaへのリンクを日本語版に置き換えたりはしていません。ただし、Vulkan Tutorial内へのリンクは、最初は英語版ですが日本語版ができれば日本語版に変更します。

---

元記事：

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__2f5535dc6d3098c3c41abb18414d2d49" frameborder="0" height="80"></iframe>

## 導入(Introduction)

## 概要(About)

このチュートリアルは [Vulkan](https://www.khronos.org/vulkan/) グラフィックスとコンピュートAPIの基本的な使い方を教えます。Vulkanは [Khronosグループ](https://www.khronos.org/) (OpenGLで知られています)による新しいAPIで、現代のグラフィックスカードのより良い抽象化を提供します。この新しいインターフェースはあなたのアプリケーションが何をしようとしているかをより良く記述できるようにし、より良いパフォーマンスに導き、そして [OpenGL](https://en.wikipedia.org/wiki/OpenGL) や [Direct3D](https://en.wikipedia.org/wiki/Direct3D) といったAPIと比べてドライバーの振る舞いに対して驚くことを少なくします。Vulkanの背後にあるアイデアは [Direct3D 12](https://en.wikipedia.org/wiki/Direct3D#Direct3D_12) や [Metal](https://en.wikipedia.org/wiki/Metal_\(API\)) と似ていますが、Vulkanはクロスプラットフォームであり、Windows,Linux,Androidで同時に開発できるというアドバンテージを持っています。

しかし、この利益の対価として、あなたはかなり冗長なAPIと一緒に働かなければいけません。すべてのグラフィックスAPIに関する詳細は、あなたのアプリケーションによって最初から設定される必要があります。それには、フレームバッファの作成やバッファやテクスチャ画像といったオブジェクトのメモリ管理を含みます。グラフィックスドライバーがやる仕事は少なくなりますが、それは正しい振る舞いをするためにはあなたのアプリケーションがやらなければいけない仕事が多くなるということを意味します。

ここで読み取って欲しいメッセージは、Vulkanは万人のためのものではないということです。高パフォーマンスコンピュータグラフィックスに熱中し、それについてなにかしたいと願っているプログラマーをターゲットにしています。もしあなたがコンピュータグラフィックスよりゲーム開発に興味があるなら、OpenGLやDirect3Dを支持したいかもしれません。そして、それがすぐにVulkanに賛成して避難されるというわけではありません [^1] 。もうひとつの代案は、 [Unreal Engine](https://en.wikipedia.org/wiki/Unreal_Engine#Unreal_Engine_4) や [Unity](https://en.wikipedia.org/wiki/Unity_\(game_engine\)) といったエンジンを使うことで、高レベルAPIを公開している場合はVulkanを使うことができます。

それはともかく、このチュートリアルでは以下の項目が要求されます

- Vulkan互換のグラフィックスカードとドライバ ([NVIDIA](https://developer.nvidia.com/vulkan-driver), [AMD](http://www.amd.com/en-us/innovations/software-technologies/technologies-gaming/vulkan), [Intel](https://software.intel.com/en-us/blogs/2016/03/14/new-intel-vulkan-beta-1540204404-graphics-driver-for-windows-78110-1540), [Apple Silicon (Or the Apple M1)](https://www.phoronix.com/scan.php?page=news_item&px=Apple-Silicon-Vulkan-MoltenVK))
- C++の経験(RAIIとinitializer listsに慣れ親しんでいる)
- C++17をサポートしたコンパイラ (Visual Studio 2017+, GCC 7+, または Clang 5+)
- 3Dコンピュータグラフィックスの経験

このチュートリアルはOpenGLやDirect3Dに対する知識は仮定しませんが、基本的な3Dコンピュータグラフィックスについて知っていることは要求します。例えば透視投影の背後にある数学に関して説明しません。コンピュータグラフィックスの導入には [このオンラインブック](https://paroj.github.io/gltut/) を見てください。その他のコンピュータグラフィックスの情報源は：

- [Ray tracing in one weekend](https://github.com/RayTracing/raytracing.github.io)
- [Physically Based Rendering book](http://www.pbr-book.org/)
- Vulkanが使われているオープンソースのエンジン([Quake](https://github.com/Novum/vkQuake) and [DOOM 3](https://github.com/DustinHLand/vkDOOM3))

あなたはC++の代わりにCを使うことができますが、その場合は別の線形代数ライブラリを使う必要があり、コードの構造からあなた自身で作る必要があります。我々はC++のクラスやRAIIといった機能をロジックやリソースのライフタイムを管理するために使います。Rust開発者のための [別のバージョン](https://github.com/bwasty/vulkan-tutorial-rs) もあります。

別のプログラム言語での開発者がついてきやすくするため、そして基本のAPIの経験を積むため、我々はオリジナルの C API を使います。しかし、あなたは新しい [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp) バインディングのほうを好むかもしれません。これはダーティーワークを抽象化し、エラーを予防することを助けてくれます。

## E-book

もしこのチュートリアルをe-bookで読みたいなら、EPUBまたはPDFをダウンロードすることができます。

- [EPUB](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial%20en.epub)
- [PDF](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial%20en.pdf)

## チュートリアルの構成(Tutorial structure)

我々はVulkanがどのように動くかの概要と、画面に三角形を描画するためにしなければいけないことから始めます。全ての小さいステップの目的は、全体像の中のそれらの役割を理解したあとではっきりします。次に、 [Vulkan SDK](https://lunarg.com/vulkan-sdk/) 、線形代数のために [GLM library](http://glm.g-truc.net/) 、ウィンドウ作成のために [GLFW](http://www.glfw.org/) といった開発環境をセットアップします。このチュートリアルでは Windows/Visual Studio と Ubuntu Linux/GCC でのセットアップ方法をカバーします。

その後、三角形を描画するために必要なVulkanプログラムでの部品を実装していきます。それぞれの章は大まかに以下のような構成になっています：

- 新しいコンセプトとその目的の紹介
- 関連するAPIを使ってあなたのプログラムに統合する
- それのパーツをヘルパー関数に抽象化する

各章は前章の続きとして書かれていますが、それぞれが独立したVulkan機能の紹介記事として読むことも可能です。これによってリファレンスとして使うこともできるようになっています。全てのVulkan関数と型は仕様書にリンクをしているので、それをクリックするとさらに詳しく学習することができます。Vulkanはとても新しいAPIで、仕様書自体に欠点があるかもしれません。そんなときは [Khronosのリポジトリ](https://github.com/KhronosGroup/Vulkan-Docs) にフィードバックを送るといいでしょう。

先程述べたように、Vulkan API はグラフィックスハードウェアを最大限コントロールするために、たくさんのパラメータを持った冗長なAPIとなっています。これが原因で、基本的な、例えばテクスチャを作るといった作業でもたくさんのステップを毎回繰り返す必要があります。そのため、我々はチュートリアルを通してヘルパー関数のコレクションを作っていきます。

全ての章の締めくくりには、説明した部分の完全なソースコードをリンクしておきます。あなたがコードの構造に迷っていたり、バグに悩んでいて比較したいときに、それを参照することができます。全てのコードは複数のベンダーのグラフィックスカード上で正しさを検証しています。それぞれの章の最後にコメントセクションを配置してあるので、特定の議題について質問をすることができます。あなたを助けるために、あなたのプラットフォーム、ドライバーバージョン、ソースコード、期待する振る舞いと実際の振る舞いを明記してください。

このチュートリアルはコミュニティによるものになるように意図されています。Vulkanはまだ新しいAPIで、ベストプラクティスはまだ確立していません。もしチュートリアルやこのサイトに何かフィードバックがあれば、どうか躊躇せずに [GitHub repository](https://github.com/Overv/VulkanTutorial) へIssueやプルリクエストを送ってください。あなたはチュートリアルに更新があったかどうかの通知のためにリポジトリを見ることもできます。

Vulkanでスクリーンに三角形を描画するという通過儀礼を終えたあとは、線形変換、テクスチャや3Dモデルを含むようにプログラムを拡張していきます。

もしあなたが以前にグラフィックスAPIで遊んだことがあるなら、あなたは最初のジオメトリがスクリーンに現れるまでにたくさんのステップがあることを知っているでしょう。Vulkanには最初のステップがたくさんありますが、あなたはそれを個別のステップで見ることになるでしょう。理解しやすいようにそして冗長に感じないように。これも重要であることを心に留めておいてほしいのですが、ひとたび退屈に見える三角形を描画できれば、テクスチャありの3Dモデルを描画するまでにそれほど多くの追加の仕事は必要ありません。そしてその先のそれぞれのステップはもっと価値があるものです。

もしあなたがチュートリアルを追うなかで問題に遭遇したら、最初にFAQをチェックしてあなたの問題がすでに解決したリストにないか見てください。もしそれでも解決しないときは、気軽にコメントで質問してください。

ハイパフォーマンスグラフィックスAPIの未来にダイブする準備はできましたか？ [それでは行きましょう。](https://qiita.com/shibainuudon/items/368567bc11ca30fa0074)

[次の記事](https://qiita.com/shibainuudon/items/368567bc11ca30fa0074)

[0](#comments)

コメント一覧へ移動

X（Twitter）でシェアする

Facebookでシェアする

はてなブックマークに追加する

[^1]: 約注：ここよくわからない