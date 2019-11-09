![五星红旗](ChinaFlag.png)

------

# 介绍

一款 jekyll 主题（[GitHub 地址](https://github.com/TMaize/tmaize-blog)），简洁纯净(主题资源请求<20KB)，未引入任何框架，秒开页面，支持自适应，支持全文检索,可以到[TMaize Blog](http://blog.tmaize.net/)查看主题效果。


# 项目配置

1. 修改`pages/about.md`中关于我的内容

2. 修改`_config.yml`文件，具体作用请参考注释

3. 清空`post`,` _posts`目录下所有文件，注意是清空，不是删除这两个目录

4. 网站的 logo 和 favicon 放在了`static/img/`下，替换即可，大小无所谓，图片比例最好是 1:1

# 使用

文章放在`_posts`目录下，命名为`yyyy-MM-dd-xxxx.md`（md或html），文章资源放在`posts`目录，

如:文章文件名是`2019-05-01-theme-usage.md`，
则该篇文章的资源需要放在`posts/2019/05/01`下,
在文章使用时直接引用即可。
写作的时候会提示资源不存在忽略即可!

```text
# 文章的开头
---
layout: mypost
title: 标题
categories: [分类1, 分类2]
---

文章内容

![这是图片](xxx.png)

```

# 示例

在帖子的开头使用YAML变量。

文件放在`_posts`文件夹内，
文件名格式为`yyyy-mm-dd-title.md`。

### 文本

```markdown

---
layout: mypost
title: 标题
categories: [分类1, 分类2]
---

文本内容
    
```

### 图片

```markdown

---
layout: mypost
title: 标题
categories: [分类1, 分类2]
---

![图片描述](ChinaFlag.png)
▲ 图片说明

```

### 视频

```markdown

---
layout: mypost
title: 标题
categories: [分类1, 分类2]
---

<video 
    src="1001.mp4" 
    width="960" 
    height="640"
    preload="auto"
    controls="controls">
</video>

```

### 嵌入

```markdown

---
layout: mypost
title: 标题
categories: [分类1, 分类2]
---

<iframe
  style="position: fixed; left: 0;"
  src="//www.xuexi.cn/lgpage/detail/index.html?id=7759802990375452621"
  frameborder="0"
  width="100%"
  height="660px"
  scrolling="auto"
  seamless
></iframe>

```
