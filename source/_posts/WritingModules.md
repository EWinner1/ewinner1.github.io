---
title: 常用写作模块
categories: [Blog]
date: 2023-03-25 13:18:33
tags:
excerpt: 常用写作模块测试
---

# 大号提示块

```Markdown
{% notel default 信息 %}
换行测试
换行测试
换行测试
{% endnotel %}

{% notel blue 提示 %}
换行测试
换行测试
换行测试
{% endnotel %}

{% notel red 自定义标题 %}
换行测试
换行测试
换行测试
{% endnotel %}
```

效果如下
{% notel default 信息 %}
换行测试
换行测试
换行测试
{% endnotel %}

{% notel blue 提示 %}
换行测试
换行测试
换行测试
{% endnotel %}

{% notel red 自定义标题 %}
换行测试
换行测试
换行测试
{% endnotel %}

# 小号提示块

```Markdown
{% note [样式/颜色] [可选: 自定义图标] %}
笔记内容
{% endnote %}
```

- [样式/颜色] 可以为 success default primary info warning danger tip question 以及 blue red 等颜色
- [可选: 自定义图标] 选项可选，请填写 Fontawsome 的图标名称后半部分，比如 fa-image

效果如下
{% note  %}
默认 提示块标签
{% endnote %}

{% note default  %}
default 提示块标签
{% endnote %}

{% note primary  %}
primary 提示块标签
{% endnote %}

{% note success  %}
success 提示块标签
{% endnote %}

{% note info  %}
info 提示块标签
{% endnote %}

{% note warning  %}
warning 提示块标签
{% endnote %}

{% note danger  %}
danger 提示块标签
{% endnote %}

{% note red fa-bolt%}
自定义提示块标签
{% endnote %}

# Buttons 按钮模块

```Markdown
{% btn [可选大小]::[名称]::[url]::[可选图标] %}
```

- 不设置任何参数的 {% btn 按钮:: / %} 适合融入段落中。

regular 按钮适合独立于段落之外：

{% btn regular::示例博客::https://www.ohevan.com::fa-solid fa-play-circle %}

{% btn regular::示例博客::https://www.ohevan.com::fa-solid fa-play-circle %}

large 按钮更具有强调作用，建议搭配 center 使用：

{% btn center large::开始使用::https://redefine-docs.ohevan.com::fa-solid fa-download %}

- `[可选大小]`:`center, regular, large, center large, center regular`
- `[可选图标]`:[Fontawesome](https://fontawesome.com/search) 图标名称，比如 fa-solid fa-house

# 折叠模块

```Markdown
{% folding [颜色]::[标题] %}
需要写的内容
{% endfolding %}
```

- `[颜色]`:`yellow, blue, green, red, orange, pink, cyan, white, black, gray`
  效果
  {% folding [颜色]::[标题] %}
  需要写的内容
  {% endfolding %}

# 分栏模块

语法

```
{% tabs 页面内不重复的ID %}
<!-- tab 栏目1名称 -->
内容
<!-- endtab -->
<!-- tab 栏目2名称 -->
内容
<!-- endtab -->
{% endtabs %}
```

效果如下
{% tabs First unique name %}

<!-- tab First Tab-->

**This is Tab 1.**

<!-- endtab -->

<!-- tab Second Tab-->

**This is Tab 2.**

This is Tab 2.

<!-- endtab -->

<!-- tab Third Tab-->

**This is Tab 3.**

This is Tab 3.

This is Tab 3.

<!-- endtab -->

{% endtabs %}
