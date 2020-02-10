# 02. 配置SSH秘钥及同步仓库

## 账号注册与SSH秘钥配置

到[GitHub官网]()注册账号后，相当于拥有了一个远端存储服务器来读写数据。那么就得先设置好传输协议。常用的是SSH协议。官方文档里给出了详细的[配置教程](https://help.github.com/cn/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)。

这里列出关键步骤：

* 检查现有 SSH 密钥

  ```shell
  # 查看id_rsa.pub公钥文件是否已存在
  ls ~/.ssh -al
  ```

* 没有则生成新SSH秘钥

  ```shell
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```

* 将SSH秘钥添加到ssh-agant

  ```shell
  # 在后台启动 ssh-agent
  eval $(ssh-agent -s)
  # 将 SSH 私钥添加到 ssh-agent
  ssh-add ~/.ssh/id_rsa
  ```

* 将 SSH 密钥复制到剪贴板

  ```shell
   clip < ~/.ssh/id_rsa.pub
  ```

* **个人GitHub主页**> **Settings（设置）**> **SSH and GPG keys**  >  **New SSH key** 或**Add SSH key**:

  在 "Title"（标题）字段中，为新密钥添加描述性标签，将密钥粘贴到 "Key"（密钥）字段。

  

## 本地仓库同步到Git

### 添加远端仓库

```shell
git remote add local_name_for_remote_repo remote_repo_SSH_address
```

### 拉取远端数据

只拉取远端数据，不与本地分支合并：

```shell
git fetch local_name_for_remote_repo
```

先拉取远端数据，再与本地分支合并：

```shell
git pull local_name_for_remote_repo branch
```

若出现关于“fast-forwards ”错误提示，表示本地该分支不是基于远端的同名分支做变更，远端不接受push操作。有两种方式解决：一是rebase，二是merge。

执行merge操作时，若出现“refusing to merge unrelated histories”错误提示，表示两个待合并commit并不基于相同的历史。执行下面命令，继续合并，生成新commit：

```shell
# checkout到待合并分支后
git merge --allow-unrelated-histories ocal_name_for_remote_repo/branch
```

### 推送数据到远端

推送指定分支到某个远端仓库：

```shell
git push local_name_for_remote_repo branch
```

推送本地所有分支到远端仓库：

```shell
git push --all local_name_for_remote_repo
```

