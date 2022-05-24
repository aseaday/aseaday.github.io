---
title: "宏观经济放弃手册（二）"
classes: wide
date: 2022-05-21
categories:
  - blog
tags:
  - economy
---
宏观经济的研究是一个不确定性极高的东西。在目前的自媒体和碎片化时代，不难看到很多大V或者媒体会根据某个数据去宣告，中国经济完蛋了。或者根据某几个指标的转折和你说我们进入了周期的某个阶段。当你一开始去了解并深究这些言论的时候，你会很兴奋，似乎你掌握了某种屠龙之技。因为好像一些事情是理所当然的，市场经济有周期，掌握周期阶段就好像看穿了市场的发展一样。而另一些知识看起来似乎能预测经济，他们通常被叫做先行指标。举一个社融数据-人民币贷款的的例子:

![rmbdk](/assets/images/rmbdk.png)

如上图是所示，这个数据好像很难一眼看出趋势。在实际研究中，我们会用移动平均来进行降噪和平滑。我们先考虑 SMA3，即移动三个月平均，如下图所示：

![rmbdk3](/assets/images/rmbdk3.png)

人民币贷款数据看起来有一定的周期感了。我们再来看看SMA12，但是会发现是一个增长趋势的曲线了：

![人民币贷款SMA12](/assets/images/rmbdk12.png)

有些人告诉你，这也许是经济学研究的两个范畴，经济增长与经济周期（可以考虑阅读[经济增长与经济周期](https://www.htsec.com/jfimg/colimg/upload/20190826/29021566804092217.pdf))。但有时候一个问题是增长还是周期， 其实很依赖于我们所看待他们的窗口和数据处理技术，我们如果进一步把人民币贷款放长到从1999到现在，事情又会发生变化。因此，如何准确的分辨增长与周期并不容易。

另外一个是关于数据口径的问题，宏观经济的很多指标是同比口径（YoY），这些同比指标所画出的曲线，能用来刻画经济的周期性吗？而且，一个同比的曲线和一个环比的曲线如果具有相关性，我们可以说两者具有伴随现象吗？我们必须十分小心才能保障自己真正看懂了数据！

对宏观经济学的研究可以参考科学研究方法。可证伪性(falsification)是一个重要的判据。有一些人认为经济学无法有可证伪性，他们举了一些宏观经济学的基础理论，尽管这些理论难以证伪，但却却被认为有效的。（举个例子，上海所有的天鹅都是白色的，这个现象是可以观测的，因为你去公园去看天鹅，但你没办法证伪，因为你很难去找到上海所有的天鹅），在实践工作中，可证伪性的意义在于我们可以坚持我们的论断或者说服别人相信我们的论断。张五常的[经济解释](https://m.douban.com/book/subject/26636765/)的第一卷比较友好地介绍了可证伪性和经济学的关系，另外一个比较严肃的讨论是[The Popperian Legacy in Economics](https://www.cambridge.org/core/books/popperian-legacy-in-economics/496986539005F83983D38E34B735E138). Popper Karl 是科学研究方法的总结者，也是20实际最伟大的哲学家之一。


![falsifiction-circle](/assets/images/falsifiction-circle.png)

有一些很好的去了解和认识宏观经济学书籍的dataset：

- IMF Article IV https://www.imf.org/en/Publications/CR/Issues/2022/01/26/Peoples-Republic-of-China-2021-Article-IV-Consultation-Press-Release-Staff-Report-and-512248, 这是一份年报型数据
- Understanding China's Economic Statistics – Second Edition，介绍经济指标如何统计以及他们的重要性
- 解读中国统计指标：概念，方法和含义（第二版）中金，字典

很多统计局提供的关键指标不能反映你想要的特性，比如投资完成额就是一个不太有用的指标，因为完成额里有大量的重复计算。按照理论，我们认为利率和投资具有负相关性：
![rate and investment](https://www.economicshelp.org/wp-content/uploads/2008/04/mec-demand-investment.png.webp)

如果我们用投资完成额代替投资指标去度量和利率的关系，我们发现可能利率和投资无关。这正是因为投资完成额指标的历史遗留问题，我们需要一些努力来获得寻找不失真的指标，如下图所介绍的。

![vip split](/assets/images/vip-split.png)

因此，需要对指标进行拆分和解读才能获得想要的东西。
