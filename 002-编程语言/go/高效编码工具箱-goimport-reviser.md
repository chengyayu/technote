# 针对 goimports 弊端的改进方法

### 1、本地安装 [goimports-reviser](https://github.com/incu6us/goimports-reviser) (`goimports` 改进版) 

```go
go install -v github.com/incu6us/goimports-reviser/v3@latest
```

### 2、配置 `Goland File Watcher` (监控文件保存时触发某种行为)

创建一个新的 `File Watcher`，关键参数如下：
- Name：`goimports-reviser`
- File type: `Go files`
- Scope: `Current File`
- Program: `goimports-reviser`
- Arguments: `-file-path $FilePath$ -local $ModuleName$ -rm-unused`

### 3、关闭 Goland 默认自动开启的 `Actions On Save` 

- gofmt
- go important 

因为不关闭会有同时修改文件的冲突，另外在第二步配置的 `file watcher` 已经具备**代码格式化**和**引入包分组排序**功能。

### 4、未来可能的继续改进：配置 `golangci-linter`。

如果我们在基本的“代码格式化和引入包分组排序”规范下，想继续增加各种 `静态检查 linter` 还可以用同样的方式增加一个 `File Watcher` （`golangci-linter`）。
