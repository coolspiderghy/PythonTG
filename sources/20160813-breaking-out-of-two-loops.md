# 高手分享：如何立即跳出两重嵌套循环？

title: 高手分享：如何立即跳出两层嵌套循环？
author: Ned Batchelder
translator: 赵喧典
reviewer: EarlGrey
date: 20160813
permalink: breaking-out-of-two-loops
keywords: coverage.py开发者, ned batchelder, 嵌套循环, 跳出循环, 生成器, 两重性, 相同字符查找

***

本文作者为 Coverage.py 的开发者 Ned Batchelder，是一位经验非常丰富的 Python 高手，而且也积极组织参加 Python 社区的活动。

本译文为 PythonTG 翻译组最新出品，译者为赵喧典，由编程派作者 EarlGrey 校对。

译者简介：赵喧典，浙江工业大学学生，专业是: 计算机科学与技术 + 自动化。爱玩，应用控，技术控，致力于成为高玩/技术宅，终极目标是 hacker/geek。

以下是正文：

如何立即跳出嵌套的两重循环？这是大家经常会碰到的问题。例如，要检验字符串中是否存在相同的字符，如何在找到一对相同的字符之后就停止循环？经典的做法是，写一个两重嵌套循环，对字符串的索引进行迭代：

```python
s = "a string to examine"
for i in range(len(s)):
    for j in range(i+1, len(s)):
        if s[i] == s[j]:
            answer = (i, j)
            break   # 如何 break 两次呢???
```

此处，我们用两重循环来生成用于检验的两个索引。当条件满足时，我们希望能同时结束这两重循环。

对此，有一些常见的方法。但我个人很不喜欢它们：

- 将循环放在一个函数中进行，利用函数返回来跳出循环。但这样的做法不尽如人意，因为循环可能不适合重构为一个新的函数，也许你需要在循环的过程中访问其他局部变量。
- 抛出异常，并在两重循环之外捕获它。这是把异常当作 goto 语句来用了。但是这里并没有异常的条件，只是变向地利用了异常机制。
- 使用布尔变量来标记循环的结束，并在外循环中检查变量值以执行第二次 break 操作。该方法毫无技术成分，某些情况下可能是有效的，但大多数情况下只造成计算资源浪费，程序效率不高 (noise and bookkeeping)。

我更偏向于使用的方法，也是我在 PyCon 2013 上提到的一种方法，[更自然地循环](http://nedbatchelder.com/text/iter.html)，就是将两重循环写成一重循环，然后简单地跳出循环。

这需要花更多的功夫在循环上，但对于抽象化迭代过程是一次很好的练习。这正是 Python 非常擅长的，但人们也很容易忽视 Python 的这一特点，而把它当作一般的语言来使用，不能发挥循环抽象的优势。

让我们重新考虑这个问题。真的需要两重循环吗？写代码之前，不妨再仔细看一遍问题描述：

> 要检验字符串中是否存在相同的字符，如何在找到一对相同的字符之后就停止循环？

在这描述中，我并没有看出两重循环的意味。事实上，只需要对索引对进行一重循环就可以了。写法如下：

```python
def unique_pairs(n):
    """在 range(n) 范围内生成索引对"""
    for i in range(n):
        for j in range(i+1, n):
            yield i, j

s = "a string to examine"
for i, j in unique_pairs(len(s)):
    if s[i] == s[j]:
        answer = (i, j)
        break
```

此处，我们写了一个生成器用于生成需要的索引对。现在，我们的循环就成了对索引对的一重循环，而不是对索引的两重循环。两重循环依然存在，只是被抽象了出去，移到了 unique\_pairs 生成器内部。

这使我们的代码更贴近自然语言的描述。还应注意到，我们不用再写两次 len(s) 了，这其实是原代码需要重构的另一个标志。而且，如果在其他地方也需要这样迭代的话，还能复用 unique\_pairs 生成器。但是要记住，复用并不是写一个函数的必要条件。

我知道这个方法有点奇异。**但它真的是最佳的解决方案了。**如果你仍旧停留在两重循环阶段，多想想该如何组织程序。事实上，当你尝试着一次性跳出两重循环，从某种意义上来说，这两重循环就是一回事了，而不是两回事。**将二重性隐藏到一个生成器内部，就可以如你所想的那样组织代码了。**

Python 有着强大的抽象化工具，包括生成器和其他能抽象迭代过程的技术。如果你想了解更多，我在[更自然地循环](http://nedbatchelder.com/text/iter.html)这场演讲中分享有更多的细节。

***

[点此查看原文链接](http://nedbatchelder.com//blog/201608/breaking_out_of_two_loops.html)。

[Python 翻译组](https://github.com/PythonTG)是EarlGrey@编程派发起成立的一个专注于 Python 技术内容翻译的小组，目前已有近 30 名 Python 技术爱好者加入。

翻译组出品的内容（包括教程、文档、书籍、视频）将在编程派微信公众号首发，欢迎各位 Python 爱好者推荐相关线索。

推荐线索，可直接在编程派微信公众号推文下留言即可。
