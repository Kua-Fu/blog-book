# 混合语言域

通常,那些从源数据中获得的多种语言混合在一个域中的文档会超出你的控制， 例如从网上爬取的页面：

```{ "body": "Page not found / Seite nicht gefunden / Page non trouvée" }```

正确的处理多语言类型文档是非常困难的。即使你简单对所有的域使用 standard （标准）分析器， 但你的文档会变得不利于搜索，除非你使用了合适的词干提取器。当然，你不可能只选择一个词干提取器。 词干提取器是由语言具体决定的。或者，词干提取器是由语言和脚本所具体决定的。像在 每种书写方式一种词干提取器 讨论中那样。 如果每个语言都使用不同的脚本，那么词干提取器就可以合并了。

假设你的混合语言使用的是一样的脚本，例如拉丁文，你有三个可用的选择：

* 切分到不同的域

* 进行多次分析

* 使用 n-grams
