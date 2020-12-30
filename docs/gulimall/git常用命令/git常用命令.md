> [Git官网文档](https://git-scm.com/docs)

```shell
$ git --help // 查看帮助
$ git -- help -a // 列出所有可用命令
$ git [commad] --help // 查看命令帮助
```

## 一、远程仓库

检出仓库

```shell
$ git clone git://github.com/jquery/jquery.git
```

查看远程仓库

```shell
$ git remote -v
```

添加远程仓库（把一个已有的本地仓库与远程关联）

```shell
$ git remote add [name] [url]
```

删除远程仓库

```shell
$ git remote rm [name]
```

修改远程仓库（更改现有远程仓库的 URL）

```shell
$ git remote set-url --push [name] [newUrl]
```

拉取远程仓库

```shell
$ git pull [remoteName] [localBranchName]
```

推送远程仓库

```shell
$ git push [remoteName] [localBranchName]
```



## 二、分支操作

查看远程仓库所有分支

```shell
$ git branch -a
```





## 代理操作

全局代理

```shell
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

针对 GitHub 的代理

```shell
#只对github.com
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
#取消代理
git config --global --unset http.https://github.com.proxy
```

取消代理

```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```




## reset-重置当前HEAD指向

