---
title: Goland 使用Go template代码提示
author: 路磨平
type: post
date: 2022-06-13T09:14:13+00:00
categories: [Coding Golang]

---

## 为什么写？

在论坛上搜索到的博客都是设置Goland识别模板文件为HTML。这样的确可以提示html便签，但并没有和数据结构联动的提示。所以有此文，记录分享一下怎么配置go template。

## Setting

进入Setting-Editor-FileTypes页面

![fileTypes页面](/images/fileTypes.png "fileTypes")

在左侧找到Go template files，我们需要加上*.tmpl。<figure class="wp-block-image size-large is-resized">

![template files页面](/images/templateFiles.png "template files")
默认的Go template files只有 *.gohtml，我们需要添加tmpl文件，有两种添加方式：

* 点击+，添加 *.tmpl
* 点击左下角 Associate File Types with Goland，在弹窗中勾选Go template files，保存

![Associate File Types页面](/images/associateType.png "Associate File Types")

保存后，Goland就可以识别tmpl文件，并给出代码提示了。

![目录树中的tmpl文件页面](/images/tmplIcon.png "目录树中的tmpl文件")
目录树中的tmpl文件图标显示为go template了

## 利用Goland编写Go template

我们用HTML模板举例，example.tmpl 初始化为 ：

```html
<!DOCTYPE HTML>
<html lang="en-us">
<head>
    <title>
    </title>
</head>
</html>
```

数据结构student ：

```go
package main

type student struct {
	Name string
	Age  int8
}
```

我们要在title里使用student的字段怎么做呢？

首先键入 {{.}}

```html
<!DOCTYPE HTML>
<html lang="en-us">
<head>
    <title>{{.}}}</title>
</head>
</html>
```

在{{.}}范围内使用快捷键 alt + Enter(windows版，mac版可参考官方文档)，弹出Specify dot type按钮。

![Specify dot type按钮](/images/dotType.png "Specify dot type按钮")

点击此按钮后，自动生成 `{{- /\*gotype: \*/ -}}` 注释， 光标在gotype之后弹出数据类型供选择，我们选择student

![](/images/student.png)

之后在title标签里我们就可以直接引用到student的字段了<figure class="wp-block-image size-large">

![](/images/codeComplete.png)
除了使用快捷键生成gotype之外，我们也可以手动键入

```{{- /*gotype: */ -}}```

_将光标置于gotype: 之后，使用快捷键Ctrl + Space可以弹出建议的数据结构。_

_PS：JetBrains系的Ctrl + Space是代码补全快捷键(Basic Completion)，但windows系统输入法占据了此快捷键，无法在IDE里使用。我们有两个方法：_

1. _修改IDE的快捷键_
2. _使用IDE提供的另一个快捷键Second Basic Completion (Ctrl + Alt + Space)_