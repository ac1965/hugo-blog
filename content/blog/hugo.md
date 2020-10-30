---
title: "Hugo"
date: 2020-10-27T22:46:10+09:00
draft: false
tags: ["hugo"]
---
## Hexo から Hugo

サイトジェネレータを [Hugo](https://https://gohugo.io) に変えてみた。

## インストール

Homebrew がインストール済みとして、

``` bash
$ brew update
$ brew install hugo
```

``` bash
$ hugo new site hoge
```

これで hoge ディレクトリーが作成され、その配下に雛形が作成される。

自分のサイトなので、ここは簡単に戻せることを前提に、GitPagesを使うことにする。

``` bash
$ git clone https://github.com/ac1965/hugo-git.git
$ cd hugo-git
$ git submodule init
$ git submodule update
```

hugo コマンドで、publicディレクトリ下にコンテンツが生成される。
