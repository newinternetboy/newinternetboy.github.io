---
title: "jekyll-mistakes-构建博客"
date: 2023-11-10 19:30:19 +0800
category: 博客构建
toc: true
---
# 如何归档？
1. 新建_page目录，创建对应归档类型的md文件，比如我想按照年来归档博客，则需要创建_page/year-archive.md
```md
---
title: "Posts by Year"
permalink: /year-archive/
layout: posts
author_profile: true
---
```
此时目录_pages对于jekyly来说还是不可见状态，需要在_config.yml配置文件中新增配置
```yml
include: ["_pages"]
```

2. 新建_data目录，创建导航配置文件navigation.yml
```yml
# main links
main:
  - title: "首页"
    url: /
  - title: "年度归档"
    url: /year-archive/
```