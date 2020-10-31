---
title: "Hugo"
date: 2020-10-27T22:46:10+09:00
draft: false
tags: ["hugo"]
---
## Hexo から Hugo

サイトジェネレータを [Hugo](https://https://gohugo.io) に変えてみた。

## インストール

[Homebrew](https://brew.sh/index_ja) がインストール済みとして、

``` bash
$ brew update
$ brew install hugo
```

テーマは、conao3さんの [anatole-ext](https://github.com/conao3/anatole-ext) にした。

``` bash
$ hugo new site hoge
$ cd hoge
$ git clone https://github.com/conao3/anatole-ext themes/anatole-ext
$ hugo new posts/first-article.md
```

これで hoge ディレクトリーが作成され、その配下に雛形が作成される。

自分のサイトなので、ここは簡単に戻せることを前提に、GitPagesを使うことにする。

``` bash
$ git clone git@github.com:ac1965/hugo-blog.git ./hugo-blog
$ cd hugo-git
$ git submodule init
$ git submodule update
$ hugo
```

hugo コマンドで、publicディレクトリ下にコンテンツが生成される。

* `deploy.sh'

hexo と大きく違うのは deploy は利用者まかせかな。

私の環境では本体と公開用の gitリポジトリで管理していて、
本体は hugo-blog、公開用は ac1965.github.com で分けている。


``` bash
#! /bin/bash

hugo=~/devel/src/github.com/ac1965/hugo-blog
public=~/devel/src/github.com/ac1965/ac1965.github.io

abort ()
{
    echo -e "\033[1;30m>\033[0;31m>\033[1;31m> ERROR:\033[0m${@}\n" && exit
}

info ()
{
    echo -e "\033[1;30m>\033[0;36m>\033[1;36m> \033[0m${@}\n"
}

warn ()
{
    echo -e "\033[1;30m>\033[0;33m>\033[1;33m> \033[0m${@}\n"
}

test -d ${hugo} && cd ${hugo} || abort "${hogo} directory not found."
# clean public
rm -fr public

info "Deploying updates to GitHub..."

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
test -d ${public} && cd ${public} || exit
info "rsync.."
rsync -at --delete --exclude=".git" ${hugo}/public/. .

# Add changes to git.
git add .
git commit -avm "update:$(env LANG=C date)" && git push
```
