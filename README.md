# Xapi-project 文档 [![Build Status](https://travis-ci.org/xapi-project/xapi-project.github.io.png)](https://travis-ci.org/xapi-project/xapi-project.github.io)

Xapi toolstack的架构文档.

## 查看此文档
此文档使用了markdown编写和Jekyll编译. 如果你希望查看本文档, 你可以通过Github或者如果你愿意，你可以浏览在Github Pages上的网页[http://xapi-project.github.io/].

## 为此文档做贡献
此文档使用markdown编写，可以以有意义的文件夹的层级结构存放。 

因为需要被Jekyll编译，每一页必须有YAML front-matter. 下面的代码需要在每一页的头部出现:

```yaml
---
layout: default
title: [title of page]
---
```

然后，如果你想让此页通过导航菜单访问，你需要往`_data/navbar.yml`添加:

```diff
  - title: Section Name
    docs:
    - foo.md
+   - path/to/your-new-doc.md
```

## 设计建议
在这个repository里还可以提供新功能的设计建议和已有功能的修改建议.更普通的API文档相比，这会通常很冗长，而且有争议. 因为还没有在toolstack总，所以我们希望把它独立出来. 要创建这样的文档请在页头添加:

```yaml
---
layout: default
title: [title of page]
design_doc: true
status: [propsed|confirmed|released (version#)]
revision: [iteration of document]
---
```

## 在pushing之前要预览文档

如果你希望在生成pushing之前预览整个网站，你可以在自己的机器上这样做.

安装Jekyll:

```
$ gem install jekyll
```

然后你可以在本地运行这个repository:

```
$ jekyll serve -w --baseurl '/xapi-project'
```

你应该可以访问这个网页 `localhost:4000/xapi-project`.

## 关于图片
如果你想贡献图片，考虑把它压缩了以保持这个repository尽可能的小:

```
convert -resize 900 -background white -colors 256 [input.png] [output.png]
```

如果你想加载一个动态图, 考虑使用
[js-sequence-diagrams](http://bramp.github.io/js-sequence-diagrams/) 然后检查创建好的.svg文件.

## 给网站的其他部分添加连接
相对路径应该可以工作，但是如果你想链接到外部的网址, 你可以添加 `{{site.baseurl}}` 然后它就可以在任意的repo或者本地访问了.

## 译者说明
Xapi是一个伟大的开源项目，它是XenServer的灵魂，希望未来它更加稳定，开放，兼容性更好.
