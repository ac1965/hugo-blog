---
title: "Hexo でハロー"
date: 2015-04-28T08:49:41+09:00
tags: ["hexo"]
draft: false
---
Welcome to [Hexo](http://hexo.io)!

## 環境

* OS X Yosemite 10.10.3
* Node.js

``` bash
$ git clone git://github.com/creationix/nvm.git ~/.nvm
$ test -f ~/.nvm/nvm.sh && source ~/.nvm/nvm.sh
$ nvm ls-remote
$ nvm install 0.12.2
$ node -v
v0.12.2
```


## 前準備
github でリポジトリほっておく。これは空っぽでよい。

例： Macのbrewでhubをインスコしてた場合
``` bash
cd ~/Documents/github
mkdir ac1965.github.io
cd ac1965.github.io
hub init
hub create ac1965/ac1965.github.io
```

## Hexo インストール

``` bash
$ npm install -g hexo
```

### 準備（ブログ用のディレクトリを作る）

hexo init フォルダ

フォルダ指定がないと、カレントに作るのでフォルダ指定しておこう。

``` bash
cd ~/Documents
$ hexo init site
$ cd site
$ npm install
```

### 動作確認

``` bash
$ hexo server
```

この状態でウェブ・ブラウザから[http://localhost:4000](http://localhost:4000/)で確認してみる。

### Github にデプロイしてみる

Hexo の 3.0 以降はどうやらデプロイがぐぐってみたのと違う。

``` bash
$ hexo deploy
ERROR Deployer not found: github
```

ようはデプロイヤが github から git に変わったのと、モジュールをインストールする必要があった。

``` bash
$ npm install hexo-deployer-git --save
```

詳しくは：[https://github.com/hexojs/hexo/issues/1040](https://github.com/hexojs/hexo/issues/1040)

参考：_config.ymlの deploy部分

```
deploy:
  type: git
  repo: git@github.com:ac1965/ac1965.github.io.git
  branch: master
```

### テーマを変えてみよう

[テーマ](http://hexo.io/themes/) で気に入ったものを探してみるとか。

``` bash
$ git clone https://github.com/joyceim/hexo-theme-apollo.git themes/apollo
```

参考：_config.ymlの languageとthemes 部分

```
language: defualt
...
theme: apollo
```


### ポスト

``` bash
$ hexo new "My New Post"
INFO  Created: ~/Documents/site/source/_posts/My-New-Post.md
```

source/_posts/My-New-Post.md をエディタで編集（[GFM](https://help.github.com/articles/github-flavored-markdown/):Github Flavored Markdown）ですね。

詳しくは： [Writing](http://hexo.io/docs/writing.html)

### 静的ファイルの作成

``` bash
$ hexo generate
$ hexo generate --deploy
$ hexo deploy --generate
```
詳しくは： [Generating](http://hexo.io/docs/generating.html)
