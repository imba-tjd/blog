--- layout: post title: Markdown笔记 date: 2018-02-22 21:48:48.000000000
-06:00 type: post parent\_id: '0' published: true password: '' status:
publish categories: - 未分类 tags: [] meta:
\_oembed\_6f580eb81eda0f7655fe687942e45f01: "{{unknown}}"
\_wpcom\_is\_markdown: '1' \_oembed\_54b0338fad42ad8465492efa05b8225f:
"{{unknown}}" timeline\_notification: '1519307332'
\_rest\_api\_published: '1' \_rest\_api\_client\_id: "-1"
\_publicize\_job\_id: '15019656696'
\_oembed\_cc0b5f2dd0f2e74f6729b4a7d339e80b: "{{unknown}}"
\_oembed\_4fb53ca1242a015f9916264416e5bae5: "{{unknown}}" author: login:
imbalancedweb email: imba.tjd@gmail.com display\_name: imba-tjd
first\_name: '' last\_name: '' permalink:
"/2018/02/22/markdown%e7%ac%94%e8%ae%b0/" ---

> https://www.zhihu.com/question/21652901\
> https://www.appinn.com/markdown

Links
-----

``` {.wp-block-code}
隐式链接标记：在链接文字后面加上一个空的方括号，再在后面把链接定义出来：
[Google][]
[Google]: https://www.google.com/ncr
有的（GFM）不加空方括号也可以
如果链接本身含有不匹配的单括号，可以用这一种，也可以加<>

如果链接里含有空格，可以换成%20或者转义，或者加<>。什么都不做，有的只会认为空格前的才是连接，GFM不会识别为链接

在链接的小括号内加空格和双引号可以设定链接的title属性

中括号内可以直接嵌套中括号，引号如果要嵌套就要用\转义了

图片链接：[![imgurl]][linkurl] --or-- [![](imgurl)](linkurl)
```

Emphasis {#em}
--------

``` {.wp-block-code}
在强调符号内部边界处（包括前面和后面）的字符如果是标点符号（包括中文和英文），并且之后没有空格地跟着非标点字符（包括数字字母中文，但可以换行），则强调符号不会生效。如：“**例如：**这个、1**"2"**3”，就会被渲染成源文本。解决方法可以在星号外围两边加空格，或者把标点移出去

解决冒号，使用：\*\*(?!([!-/:-@[-`{-~\s])|$)或者用中文的Unicode进行匹配，但这样会导致错误匹配到“正常：**加黑**”。所以匹配到的必须是和前面成对的，需要用到平衡组

**abc\***可以正常显示为abc*，甚至不用转义也可以；或者用实体也行；要不就避免这种表示方法，改用反引号包裹文本
```

backtick
--------

``` {.wp-block-code}
如果`和`之间有`，则需要在两端使用两个`，无法使用斜杠来转义。写作：``1`2``，效果：1`2，如果内容只有`，在两端加空格。如果需要在中间写更多`，就在两边继续加`，代码块同理。也可以使用code包裹\`，注意转义，否则会出现双重样式

`转换成html后就是code标签，但同时还会把特殊字符转换成html实体；html实体直接写进两个`之间，渲染后仍保持&#....;的样子

markdown语法在code标签中仍有效，在pre和`中无效

`123``123`会渲染成123``123
```

表格中的pipe符号
----------------

``` {.wp-block-code}
一般情况下（非表格中），`|`、<code>|</code>、<code>\|</code>、 <code>|</code>都可以正常显示成|
而`\|`会显示为\|，<code>\|</code>会显示为实体

在GFM的表格中，竖线被认为是表格分隔符的优先级高于`，所以`|`和<code>|</code>是已知不可以的
这是GitHub的bug：https://github.com/lunet-io/markdig/issues/236

在表格中要显示竖线时，写成`\|`在Github上可以正常显示，但在MS Docs里的效果不一样，前者可以正常显示表格但会把反斜杠也显示出来
<code>\|</code>应该是同理

使用<code>|</code>是已知两者都可以的，但是可读性太不好
```

未分类
------

-   在 HTML 块标签间的 Markdown
    格式语法将不会被处理，而在段标签间是有效的
-   如果在code标签范围内写保留字符，写实体或直接写都可以，优先于上一条
-   如果你确实想要依赖 Markdown 来插入\
    标签的话，在插入处先按入两个以上的空格然后回车。（某些实现不需要，但GitHub需要）
-   如果列表项目间用空行分开，在输出 HTML 时 Markdown
    就会将项目内容用标签包起来
-   **如果你的 `*` 和 `_` 两边都有空白的话，它们就只会被当成普通的符号**

-   https://spec.commonmark.org/

