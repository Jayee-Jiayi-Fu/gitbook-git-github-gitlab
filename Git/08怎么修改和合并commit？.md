# 08. 怎么修改、合并、删除 commit？

## 修改 commit

### 修改最近一次 commit 的 message

```shell
git commit --amend
```

此时若出现编辑器错误提示，可指定使用 vim 编辑器：

```shell
git config --global core.editor vim
```

在打开的 commit 对象文件中，可看到该 commit 的相关信息，并可直接修改 message 内容。修改完成后，Git 会生成一个新的 commit 来替代原来的 commit，所以你会看到 commit 的指针号已改变。

![编辑commit的message](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/编辑commit的message.JPG)

你也可以直接在命令行中快捷修改：

```shell
git commit --amend -m "new message"
```

### 修改最近一次 commit 的快照

若希望修改最近一次 commit 的文件内容，但又不想留下多一个 commit 记录，可以在变更文件并`add`进暂存区后执行下面命令:

```shell
git commit --amend --no-edit
```

它会新建一个 commit 替代原来的，并且拥有一样的 message，历史记录上的 commit 数不变。

### 修改老旧 commit 的 message

修改任何非最近的 commit，都需要变基操作。找到需要修改的 commit，基于它的父 commit 执行指令：

![变基操作](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/变基操作.JPG)

```shell
git rebase -i 11fbaa2
```

执行上面指令后，Git 将打开策略文件如下。按照策略，把需要修改的 commit 前的`pick`改为`r`，保存退出。

![编辑变基策略文件](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/编辑变基策略文件.JPG)

然后 Git 会打开修改了策略的 commit 对象文件，在这个文件中修改 message 为“change readme content and message”，保存退出。操作完成后终端给出成功提示信息如下：

```shell
[detached HEAD 6ff26b9] change readme content and message
 Date: Tue Feb 4 08:58:35 2020 +0800
 1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/master.
```

上面信息表示，Git 会先分离头指针，作出修改后，从修改位置起往后的全部记录都会用新的 commit 代替。原来的 HEAD 指针也会随之指向新的 commit。

之所以会产生新的 commit，是因为，基于某个父 commit 的子 commit 对象，它的 message 改变了，所以变成了新的 commit，拥有新的指针 ID。它后面的 commit，则是由于 commit 对象的 parent 属性中，保存的前一个 commitID 改变了，所以生成新的 commit。就像多米诺牌，一直延续到最近的历史也被新 commit 替换后才停下。

![变基后的历史](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/变基后的历史.JPG)

使用`reflog`命令可以看到这个过程：

![reflog查看变基过程](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/reflog查看变基过程.JPG)

变基操作适用于 commit 还没贡献享到团队的集成分支上，仅在个人工作区修改的情况。如果 commit 已贡献出去，就不应该随意地进行变基操作，否则会对团队成员造成很大影响。

## 合并 commit

### 把连续多个 commit 整理成一个

现在有如下历史记录：

```shell
4e76288 (HEAD -> master) add index
3c5a670 change readme again
9d32c55 change readme
2e98b74 add readme
765d707 rm readme
6ff26b9 change readme content and message
11fbaa2 add readme
```

现在要把除首尾两个 commit 外的所有 commit 合并。合并 commit 也是一个变基操作，所以找到需要修改的最早 commit 的父 commit，执行命令：

```shell
git rebase -i 11fbaa2
```

随后在打开的策略文件中改`pick`为`squash(或简写s)`，表示将对应 commit 合并到前一个 commit，保存退出。

![合并连续commit策略](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/合并连续commit策略.JPG)

Git 会创建一个新的 commit，保存所有合并 commit 的内容，你可以在该 commit 对象文件中编辑它的 message：

![编辑合并的commit](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/编辑合并的commit.JPG)

保存退出后，在查看历史，发现分支已合并。当然，除父 commit 外，所有往后的 commit 都是新的。

![合并后的历史](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/合并后的历史.JPG)

### 把不连续的多个 commit 整理成一个

现有目录如下：

```shell
3030aca (HEAD -> master) add index
988b256 a complete readme...(略)
11fbaa2 add readme
```

现在需要把首尾两个 commit 合并成一个，也是执行变基操作：

```shell
git rebase -i 11fbaa2
```

但由于基底已是最早的 commit，在策略文件中不会显示，所以需要手动添加 commit 记录，再把需要合并的 commit 移动到它下面，改策略为`s`。保存退出：

![合并不连续commit的策略](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/合并不连续commit的策略.JPG)

由于没有基底，即新 commit 的前一个初始 commit 为空，导致了一个冲突。`git status`给出提示信息如下：

![合并不连续commit冲突提示](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/合并不连续commit冲突提示.JPG)

按照提示执行命令以解决冲突:

```shell
git rebase --continue
```

然后就可以在新 commit 对象文件中编辑 message 信息。最用运行结束后，看到的提交历史是这样子的：

![合并不连续commit后的历史](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/合并不连续commit后的历史.JPG)

## 删除 commit

删除最近几个 commit，等同于回到之前某次提交，并清空工作区和暂存区：

```
git reset --hard f09122c
```
