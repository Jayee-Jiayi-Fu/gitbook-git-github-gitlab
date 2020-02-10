# 01. 安装Git及最小配置

git的安装请参考这份[文档]([https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git](https://git-scm.com/book/zh/v2/起步-安装-Git))，不在赘述。

查看当前Git版本：

```shell
git --version
```

Git安装完成后，必须配置用户名和邮箱 ：

```shell
git config user.name 'your_name'
```

```shell
git config user.email 'your_email'
```

配置有三个作用域，不同作用域优先级不同（local>global>system）：

```shell
#--local 只对当前仓库有效（默认作用域）
git config --local user.name 'your_name'
#--global 对当前用户所有仓库有效
git config --global user.name 'your_name'
#--system 对系统所有登录用户有效
git config --global user.name 'your_name'
```

显示config配置列表：

```shell
git config --local --list
```

指定显示某个config的值：

```shell
git config --local user.name
```



