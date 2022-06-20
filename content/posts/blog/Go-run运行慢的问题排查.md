---
title: go run运行慢的问题排查
author: 路磨平
type: post
date: 2022-06-14T09:47:41+00:00
categories: [Coding Golang]

---

## 背景

> 平台：windows 10
>
> Go version: go1.17.1 windows/amd64
>
> 两个项目，web 和 cache。 web速度正常，cache的速度特别慢。

## 排查过程

---
***确认是哪个阶段慢***

go run 有两个阶段：编译、运行。[go run官方文档](https://pkg.go.dev/cmd/go@go1.18.3#hdr-Compile_and_run_Go_program)

分别执行go build 以及 运行exe，确认是go build速度慢。

---
***go build 问题排查***

我们可以使用go build -x
命令输出执行的命令（已隐藏无关信息）：[go build官方文档](https://pkg.go.dev/cmd/go@go1.18.3#hdr-Compile_packages_and_dependencies)

```
PS ..\GolandProjects\build&gt; go build -x
WORK=..\AppData\Local\Temp\go-build2303024183 
mkdir -p $WORK\b001\ 
cat &gt;$WORK\b001\importcfg.link &lt;&lt; 'EOF' # internal 
packagefile build=..\AppData\Local\go-build\79\7918a7fb2cf49bb2c34e5af59ffa66b9a34c7def29743cf9f0c3449bc6e35f42-d
packagefile fmt=C:\Program Files\Go\pkg\windows_amd64\fmt.a
...
...
...
packagefile crypto/ed25519/internal/edwards25519/field=C:\Program Files\Go\pkg\windows_amd64\crypto\ed25519\internal\edwards25519\field.a
EOF
mkdir -p $WORK\b001\exe\
cd .
"C:\\Program Files\\Go\\pkg\\tool\\windows_amd64\\link.exe" -o "$WORK\\b001\\exe\\a.out.exe" -importcfg "$WORK\\b001\\importcfg.link" -buildmode=pie -buildid=BRRbYa3TpeS646_mNlcN/8SIvV6pb9zai7hzVb8u-/gJFsZ-PRPWOOn_1LxoK5/BRR
bYa3TpeS646_mNlcN -extld=gcc "..\\AppData\\Local\\go-build\\79\\7918a7fb2cf49bb2c34e5af59ffa66b9a34c7def29743cf9f0c3449bc6e35f42-d"
"C:\\Program Files\\Go\\pkg\\tool\\windows_amd64\\buildid.exe" -w "$WORK\\b001\\exe\\a.out.exe" # internal
cp $WORK\b001\exe\a.out.exe build.exe
rm -r $WORK\b001\
```

执行过程中观察到有两条命令很慢

* "C:\Program Files\Go\pkg\tool\windows_amd64\buildid.exe"; -w "$WORK\b001\exe\a.out.exe" #internal
* cp $WORK\b001\exe\a.out.exe build.exe

第一条命令使用了buildid（<a rel="noreferrer noopener" href="https://pkg.go.dev/cmd/buildid" target="_blank">官方文档</a>
），更新了a.out.exe的哈希值

第二条命令是拷贝

可能的排查方向：

1. 网络：排除。上诉两条命令不需要联网

2. 文件：排除。重启了电脑之后，go build还是很慢。go build创建的文件也是随机的，没有系统初始化时即占用文件的可能性

3. Go环境变量或者版本：排除。相同的环境下，web的速度正常，cache的速度慢。

陷入瓶颈。。。

登上万能的搜索引擎（PS：用的是bing，抵制百度）

![论坛讨论图片](/images/bbs.png "论坛讨论")

论坛上说的是360扫描了编译后的文件，而我们这里速度慢的命令也是对exe的操作，很有可能也是由于安全扫描造成的。（PS：我的电脑上没有360，但是有公司的安全软件）

那为什么web项目的速度正常呢？

带着疑问，我把速度慢的cache项目加了一句代码

```
http.ListenAndServe(":9999", nil)
```

执行go build，速度正常。

作为对照，把原本速度正常的web项目删除相同代码

执行go build， 速度很慢。

# 结论

安全软件会扫描go build 生成的exe文件，导致速度变慢。

个人电脑的话，关闭安全软件就行了。