---
title: React XSS问题之Server Rendering
date: 2020-04-01 00:33:00
tags:
- 前端
- React
- Web安全

categories:
- 技术
---
最近在看到Redux官网上关于Server Rendering的文章，其中有这么一段代码：
![前端菜鸡表示看半天没看出问题](https://miro.medium.com/max/3132/1*30HMtDWQ9Erij96c0oOKJg.png)
我们将Redux Store中的状态传递给网页的过程中。由上面的那张截图中，我们在script tag里，仅以JSON.stringify将状态存至一个全局下的变量，以便让网页取得状态。
这个官网上说的潜在的安全隐患。

<!--more-->

道理是这样的当浏览器解析这页面的html时，会先遇到<script>然后继续往下解析，直到浏览器读到</script>。这表示如果你的Redux Store中有一个状态值如底下这段程式码，你就会收到一个跳出来的XSS告警。

浏览器并不会如我们所想的的，一路读到上面那张截图中的</script>。取而代之的是，浏览器只会读到状态中bio: "as后头的</script>。

### 如何解决
我们网页中所有的输入介面都必须具备HTML escape的功能（多数情况下React已经帮你做了这个工作）。
在Server Rendering中，当状态由server送至client时，由于你不会再让React核心处理那些字串，意即没人替你实现这个功能。故你需要主动将它们序列化以达到HTML escape的目的。

Github上有一个现成的[轮子](https://github.com/yahoo/serialize-javascript)来做这个事。