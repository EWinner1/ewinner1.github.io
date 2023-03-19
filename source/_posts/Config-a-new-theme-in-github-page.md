---
title: Config a new theme in github page
date: 2023-03-19 12:46:11
tags:
categories: [Github pages] #分类
# sticky: #优先级
thumbnail: "/images/How-to-config-a-theme.png"
---

# 选择一个你喜欢的主题

安装一个新的主题首先当然是选择一个你喜欢的样式, 对于 hexo, 你可以在[这里](https://hexo.io/themes/)寻找到你喜欢的主题的样式, 这里选择的是[Redefine theme](https://github.com/EvanNotFound/hexo-theme-redefine), 其具有不错的样式和中文文档。

# 开始安装

推荐使用`npm`进行安装。

```Bash
npm install hexo-theme-redefine@latest
```

{% notel blue 如何更新？ %}
我们可以通过`npm`安装最新版本

```Bash
npm install hexo-theme-redefine@latest
```

需要注意的是在进行更新后要检查`\node_modules\hexo-theme-redefine\_config.yml`和当前主题的配置文件`_config.redefine.yml`的关键信息是否一致, 否则可能导致构建错误。
{% endnotel %}
在安装完成后, 要将`_config.yml`中的`theme: xxxx`改为`theme: redefine`

# 开始配置

完整的配置方案请点击[这里](https://redefine-docs.ohevan.com/docs/intro), 这里只简单的介绍一些基本的配置。

## base_info

```
base_info:
  title: Theme Redefine # 网站标题
  author: The Redefine Team # 作者名称
  url: https://xxxx # Base URL
```

## style

### primary_color

设置网站的主题色

### avatar

设置作者的头像, 可以使用外链或者本地链接

```Bash
avatar: /images/avatar.svg
avatar: https://raw.githubusercontent.com/EvanNotFound/hexo-theme-redefine/main/source/images/avatar.svg
```

### favicon

设置网站显示在 Title 的 logo, 也可以使用外链或者本地链接

### right_side_width

设置右侧目录模块的宽度。一般情况下, 你无需修改。如需设置, 请保持单位为`px`

### first_screen

开启后将显示在网站首页。配置为`enable`开启首屏
`background_image`首屏背景图片, 可使用本地图片或图片外链 URL
{% notel blue Notice %}
（如果你的 Hexo 博客的网址位于子目录, 比如 https://example.com/blog, 请使用 图片外链 URL）
{% endnotel %}

```
first_screen:
    enable: true
    background_image:
      light: /images/wallhaven-wqery6-light.webp # 为亮色模式的背景图, 使用相对路径或者外链(如果网址位于子目录, 请使用外链)
      dark: /images/wallhaven-wqery6-dark.webp # 为暗色模式的背景图, 使用相对路径或者外链(如果网址位于子目录, 请使用外链)
    title_color:
	  font_sizes: # 设定首屏的字体大小
      light: "#fff" # 首屏标题的文字颜色 (light mode)
      dark: "#d1d1b6" # 首屏标题的文字颜色 (dark mode)
    font_sizes:
      title: 2.8rem # 设定首屏标题的字体大小
      subtitle: 1.5rem # 设定首屏副标题的字体大小
    line_height: 1.2 # 首屏标题字体高度
    title: Theme Redefine # the title in the middle of the first screen. HTML supported (e.g. svg html code of your logo)
    subtitle: # 副标题（有打字效果）
      enable: false
      list: [] # 副标题的句子内容, 可以填写多个句子, 比如 ['This is the first sentence', 'This is the second one']
    custom_font: # 首屏自定义字体
      enable: false
      font_family: # 字体名称
      font_url: # 到字体 css 样式表的链接, 比如 Google Fonts 的 https://fonts.googleapis.com/css2?family=Roboto&display=swap
```

### scroll:

```
progress_bar: # reading progress bar
	enable: true
percent: # reading progress percent
    enable: true
```

## Custom

```
custom: # custom font for the whole site
font:
    chinese: # 自定义中文字体
		enable: false
		font_family: # 字体名称
     	font_url: # 字体URL
    english: # custom font for English
		enable: false
		font_family: # 字体名称
      	font_url: # 字体URL
```
