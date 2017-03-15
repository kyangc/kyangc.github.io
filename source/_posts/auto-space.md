title: 自动在博客的中英文单词中间加上空格
date: 2016-12-28 10:00:00
categories: 技术
tags: 折腾 
description: 强迫症患者的福音-_,-
---

偶然间在逛[Kaedea](http://kaedea.com/2016/06/26/front-auto-space/)博客的时候发现了这个玩意：[pangu.js](https://github.com/vinta/pangu.js)，很好很强势，终于可以解放老夫的空格键了，强迫症伤不起。

嘛，Github上pangu.js介绍的这段话也很有意思：

> 漢學家稱這個空白字元為「盤古之白」，因為它劈開了全形字和半形字之間的混沌。另有研究顯示，打字的時候不喜歡在中文和英文之間加空格的人，感情路都走得很辛苦，有七成的比例會在 34 歲的時候跟自己不愛的人結婚，而其餘三成的人最後只能把遺產留給自己的貓。畢竟愛情跟書寫都需要適時地留白。

嗯，记录下在[Next](https://github.com/iissnan/hexo-theme-next)主题下是怎么实现的吧：

* 在这里找到模板文件：

```bash
vim path_to_hexo/themes/next/layout/_layout.swig
```

* 在head块中声明JS：

```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/pangu/3.3.0/pangu.min.js"></script>
```

* 在body块的末尾调用一下就好辣

```js
<script>pangu.spacingPage();</script>
```
