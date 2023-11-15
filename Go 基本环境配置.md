# GOROOT

GOROOT 是 Go 程序安装位置，等同于 JAVA_HOME。

# GOPATH

GOPATH 是程序或依赖包安装目录，等同于 Maven 的 repository。

# GOPROXY

GOPROXY 用于设置 Go 依赖包镜像地址，加快依赖下载。推荐 [https://proxy.golang.com.cn/](https://proxy.golang.com.cn/)。

一键配置：

Bash (Linux or macOS)：

```bash
# 配置 GOPROXY 环境变量
export GOPROXY=https://proxy.golang.com.cn,direct
# 还可以设置不走 proxy 的私有仓库或组，多个用逗号相隔（可选）
export GOPRIVATE=git.mycompany.com,github.com/my/private
```

PowerShell (Windows)：

```PowerShell
# 配置 GOPROXY 环境变量
$env:GOPROXY = "https://proxy.golang.com.cn,direct"
# 还可以设置不走 proxy 的私有仓库或组，多个用逗号相隔（可选）
$env:GOPRIVATE = "git.mycompany.com,github.com/my/private"
```