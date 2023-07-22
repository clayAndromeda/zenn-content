---
title: "Slackで綺麗にリンク（URL）を投稿するための方法と時短テクニック"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: true
---

SlackでURLを共有する時、長いURL（リンク）をそのまま貼り付けると非常に見た目が悪い。

![](/images/20230720/01.png)

これを改善する1つの手段として、テキストをハイパーリンク化するという方法がある。

![](/images/20230720/02.png)

ハイパーリンク化は至って簡単。

リンクを設定したいテキストを選択して、リンクボタンを選択。その後表示されるポップアップにURLを入力すると実現できる。

![](/images/20230720/03.png)

![](/images/20230720/04.png)

ただし、この方法は非常に面倒くさい。そこで、もっと楽にハイパーリンクテキストを作る方法を紹介する。

## 1. Ctrl + Vで選択中テキストのリンクを貼り付ける

実は先ほどのリンクを設定したいテキストを選択した状態のときに、`Ctrl + V`を押すことで、テキストのハイパーリンク化が実現できる。多くの場合、この方法さえ覚えておけば事足りるだろう。

![](/images/20230720/05.png)

## 2. リッチテキストを生成するブックマークレットを使用する

プログラマは怠惰なので、もっと楽をしたい。

つまり、リンクを貼り付けるテキストそのものも自動生成したい。

[こちらのサイトで公開されているブックマークレットを利用すれば](https://media-massage.net/blog/linkbookmarklet/)、今開いているタイトルにページのURLのリンクを付与した文字列をコピーできる。実際のブックマークレットは以下。

@[card](https://media-massage.net/blog/linkbookmarklet/)

```js
javascript:void(function(){let w=window.open(null,null,"height=60,width=500"),d=w.document;d.open();d.write('<body style="padding:10px 15px;margin:0;display:flex;flex-flow:row nowrap;align-items:center"><a id="a" style="flex-grow:1" target="_blank"></a><button id="copy" style="width:100px;height:30px;margin-left:10px;cursor:pointer">Copy</button></body>');d.title="Copy as rich-text";let u=window.location.toString(),c=d.getElementById("copy"),a=d.getElementById("a");a.innerHTML=window.document.title;a.href=u;function copyToClip(doc,html,text){function listener(e){e.clipboardData.setData("text/html",html);e.clipboardData.setData("text/plain",text||html);e.preventDefault()}doc.addEventListener("copy",listener);doc.execCommand("copy");doc.removeEventListener("copy",listener)}c.onclick=function(){copyToClip(d,a.outerHTML,u);w.close()};d.close();c.focus()}())
```

ブックマークレットを実行すると、画像のようなポップアップウィンドウが表示される。Copyボタンを押すと、ハイパーリンク化したテキストがコピーされる。後はそれをSlackに貼り付けるだけだ。 このような、装飾付きの文字のことを、リッチテキストと言うらしい。

![](/images/20230720/06.png)

※ブックマークレットとは何なのか、については[こちらの記事](https://qiita.com/aqril_1132/items/b5f9040ccb8cbc705d04)が詳しい。

@[card](https://qiita.com/aqril_1132/items/b5f9040ccb8cbc705d04)

## 3. markdown形式のリンクテキストをショートカット一発でハイパーリンクに変換する

ブックマークレットを使うと、`[ページタイトル](https://ページURL)`といったmarkdown形式のリンクテキストを作って、クリップボードにコピーできる。ブックマークレットは以下の通り。

```js
javascript:(function(){var f=document.title;var c=location.href;var e="["+f+"]("+c+")";var b=document.createElement("div");b.appendChild(document.createElement("pre")).textContent=e;var d=b.style;d.position="fixed";d.left="-100%";document.body.appendChild(b);document.getSelection().selectAllChildren(b);var a=document.execCommand("copy");document.body.removeChild(b)})();
```

![](/images/20230720/07.png)

このブックマークレットはmarkdownを書いている時は便利だが、Slackにコピペすると非常に醜い見た目になってしまう。

実はこのmarkdown形式リンクテキストを、ショートカット一発でハイパーリンクテキストに変換する方法がある。対象のテキストを `Ctrl + A`で全選択したあと、`Ctrl + Shift + F`を押すと、一発でハイパーリンクテキストに変換できる。

![](/images/20230720/08.png)

![](/images/20230720/09.png)

## 4. Slackの入力欄のWYSIWYG化をやめる

markdown形式のリンクテキストをハイパーリンクに変換する方法を紹介したが、こんな一手間をかけずともハイパーリンクになってほしいのが本音だ。

それは可能だ。

Slackには、入力欄をmarkdownで書式設定できるようにするモードがある。環境設定 > 詳細設定から、「マークアップでメッセージを書式設定する」にチェックを入れると、入力欄では文字が装飾されなくなる。

![](/images/20230720/10.png)

この状態で先ほどのmarkdown形式リンクテキストを入力し送信すると、markdownで指定した書式通りに、ハイパーリンクテキストを作ってくれるのだ。

![](/images/20230720/11.png)

![](/images/20230720/12.png)

元々の入力欄で文字が装飾されるモードのことを一般に「WYSIWYG」と呼ぶらしい。どのモードを選択するかは、使い比べてみるのがよいと思う。

参考: [学生に向けて喋った/Slackの入力欄のWYSIWYG化を許すな/ガーリィレコード『フラフープデブ』 – chao情報](https://chao.tokyo/archives/2319)

@[card](https://chao.tokyo/archives/2319)


