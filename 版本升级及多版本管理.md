# 前言

关注 Go Blog 的同学可能都会注意到，官方每次在发布新版本时都会提供类似下面的版本升级策略：

![go-upgrade-1655691183Ka99Uw](http://blog-media.knowledge.ituknown.cn/Golang-Upgrade/go-upgrade-1655691183Ka99Uw.png)

这是 Go 官网提供的一种多版本管理方式，也是升级 Go 的主要方式。

至于为什么需要安装多个版本的 Go，除了工作需要外，更多的可能是为了尝鲜。

现在就来看下如何升级 Go 版本:

# 如何实现版本升级

按照官网的方式，安装某个版本的 Go 跟安装普通依赖包如出一辙：

```bash
$ go install golang.org/dl/<go-version>
```

这个命令并不会真正的执行安装操作，只是将对应的 Go 版本的包装器下载到 `$GOPATH/bin` 目录下（可以理解为创建一个标记），想要真正的执行安装还需要执行下下面的命令才行：

```bash
$ <go-version> download
```

|**Note**|
|:-------|
|直接使用 `<go-version> download` 执行安装的前提是配置了环境变量（`export PATH=$PATH:$GOPATH/bin`），如果你没有配置环境变量需要使用 `$GOPATH/bin/<go-version> download` 代替！|

这个命令会将你指定的版本（`<go-version>`）下载到当前用户下的 sdk 目录中（可以理解为新的 `GOROOT`，只是没有添加到环境变量中）。另外，需要特别强调的是，这个 sdk 是默认的安装目录，没法修改。

--

这里要额外的说下，Go 多版本安装使用的是 go-dl 工具，对应的仓库地址是：[https://github.com/golang/dl](https://github.com/golang/dl)。

在这个仓库里你会看到很多以各个版本命名的目录（截图如下），也就是说每次发布新版本时就会在这个仓库里创建一个对应的版本目录，所以如果你不知道 go 有哪些版本可以通过这个仓库的目录查看。

![golang-dl-1655646062UWSaKv](http://blog-media.knowledge.ituknown.cn/Golang-Upgrade/golang-dl-1655646062UWSaKv.png)

# 版本升级

现在来看下版本升级示例，下面是我当前系统上的 Go 版本以及主要环境变量：

```bash
$ go version
go version go1.15.15 linux/amd64

$ go env
GOARCH="amd64"
GOOS="linux"
GOROOT="/usr/lib/golang/go"
GOPATH="/usr/lib/golang/gopath"
GOPROXY="https://proxy.golang.com.cn,direct"
...
```

现在我想要安装当前 go 的最新版本 `go1.18.3`，只需要执行下面的命令：

```bash
$ go install golang.org/dl/go1.18.3

# 或

$ go install golang.org/dl/go1.18.3@latest
```

这个命令执行完成后会将该版本的下载包装器下载到 `$GOPATH/bin` 目录下，文件名就是指定的 go 版本号 `go1.18.3`：

```bash
$ ls $GOPATH/bin | grep 18
go1.18.3
```

现在开始执行安装命令：

```bash
$ go1.18.3 download
```

然后就会显示下载进度条，当下载完成后就会看到有个解压操作。会将下载的这个 Go 版本安装到当前用户下的 sdk 目录：

```
.....
Downloaded  92.8% (131513376 / 141748419 bytes) ...
Downloaded  94.1% (133315632 / 141748419 bytes) ...
Downloaded  95.7% (135691296 / 141748419 bytes) ...
Downloaded  97.2% (137821232 / 141748419 bytes) ...
Downloaded  98.5% (139623520 / 141748419 bytes) ...
Downloaded 100.0% (141748419 / 141748419 bytes)
Unpacking /home/ituknown/sdk/go1.18.3/go1.18.3.linux-amd64.tar.gz ...   # 看这里
Success. You may now run 'go1.18.3'
```

之后就会看到 home 目录下多了个 sdk 目录，我们安装的这个版本的 Go 也在这个目录下：

```bash
$ ls ~/sdk/
go1.18.3
```

当然了，如果你现在想要再安装一个 `go1.10.7` 版本，按照上面的操作示例，最终这个 `go1.10.7` 也是会被安装到这个 sdk 目录。

之后就可以正常使用了，只需要在项目里配置下 `GOROOT` 变量即可，其他的照旧~

--

https://go.dev/dl

https://go.dev/doc/manage-install

https://medium.com/@sherlock297/go-install-version-is-required-when-current-directory-is-not-in-a-module-4ae2d1e0aa86