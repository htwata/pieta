---
layout: post
title:  "Electronの開発環境構築"
date:   2017-06-25 20:01:14 +0900
tags: electron nodejs
permalink: /electron/install
---

Electronの開発に必要なnode.jsとElectronの導入を行う

## node.jsのインストール

### lubuntu 16.10

apt getでinstallしようとすると、古いNodeJSがインストールされるため、
パッケージマネジャーを用いてインストールする方法が一般的

Electronの実行環境であるNode.jsは、
バージョン管理するパッケージマネジャーが何種類か存在する
ex. nvm, n package, nodebrew

http://bam6o0.hatenablog.com/entry/2017/05/05/050927

#### nodebrewのインストール

https://github.com/hokaccha/nodebrew

・URLを参考に、nodebrewのインストールとPATHの設定を行う
{% highlight bash %}
$ wget git.io/nodebrew
$ perl nodebrew setup
{% endhighlight %}

・PATHの設定のため.bashrcに下記を追加
~/.bashrc
{% highlight bash %}
export PATH=$HOME/.nodebrew/current/bin:$PATH
{% endhighlight %}

・反映
{% highlight bash %}
$ source ~/.bashrc
{% endhighlight %}

#### node.jsのインストール

{% highlight bash %}
# nodebrew install-binary latestで最新版をインストール
nodebrew install-binary v8.1.2
{% endhighlight %}

・PATHの設定
{% highlight bash %}
$ vim ~/.profile
#追加
export PATH=$HOME/.nodebrew/current/bin:$PATH
$ source ~/.profile
{% endhighlight %}

#### electornのインストール


{% highlight bash %}
# -g はグローバル設定
$ npm -g install electron
{% endhighlight %}

下記を参考にelectronのプロジェクトを作成・動作確認
[参考サイト]: http://qiita.com/Quramy/items/a4be32769366cfe55778
