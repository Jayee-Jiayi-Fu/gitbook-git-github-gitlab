# 05. 三个对象:commit、tree 和 blob

## 探秘隐藏的.git 文件

.git 中存放了 Git 进行版本控制所需的文件。其中最重要的有几个：`HEAD`、`config`、`refs/`、`objects/`。

![dot_git_list](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/dot_git_list.JPG)

- `HEAD`文件中存放着一个引用，指向当前正在使用的分支：

  ```shell
  #HEAD>>
  ref:refs/heads/master
  ```

- `config`文件中存放当前项目的配置信息。`config`命令的`--local`作用域修改的就是这里的数据：

  ```shell
  #config>>
  ...
  [user]
  	name = your_name
  	email = your_email
  ```

- `refs`文件夹中存放了与 heads(分支)、tags（标签）等相关信息。上面 HEAD 文件的引用就是指向此处：

  ```shell
  #refs/>>
  heads/
  	|-master
  	|-new_branch
  tags/
  	|-v1.0.0
  ```

  `heads/master`中存放的是一个指针：

  ```shell
  ce1ed38a...(略)
  ```

  我们通过下面命令查看该指针的类型：

  ```shell
  git cat-file -t ce1ed38a
  ```

  得到输出如下。也就是说，这个 HEAD 指针指向一个 commit 对象：

  ```shell
  commit
  ```

  我们再通过下面命令查看这个 commit 对象的内容：

  ```shell
  git cat-file -p ce1ed38a
  ```

  得到输出如下。commit 中可以看到提交的相关信息，而且它包含一个**tree**对象：

  ```shell
  tree ce1ed38a...(略)
  author author_name <author_email> ...
  committer committer_name <committer_email> ...

  message of this commit
  ```

  同理，`tags/v1.0.1`中也是一个指针，指向的对象是 tag 类型。tag 对象内容是一个 commit 对象。

- `objects/`中的两字符目录存放了 Git 所跟踪的对象。Git 会进行自我梳理操作，当松散文件较多时就进行打包并存放到`pack/`目录。

![objects目录](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/dir_objects_list.JPG)

 进入其中一个两字符目录，里面存放的是以哈希值命名的文件。外层目录的两个字符+内层文件名的哈希值组成一个完整的指针。例如`f3`里存放了一个名为`7d0f449...(略)`的文件，就组成完整指针`f37d0f449...(略)`。

 尝试查看这个指针的类型是`tree`，内容包含了一个`blob`类型的对象，指向某个文件：

```shell
100644 blob b72e61...(略) style.css
```

 你可能已经察觉出来了，可以仍未 tree 对象表示目录，blob 对象表示文件。commit 对象包含的 tree 对象，可视为一个顶级快照目录。

## tree、commit 和 blob 的关系

假设当前有如下目录：

```
readme.md
index.html
styles/
	|-style.css
images/
	|-git-logo.png
```

 在添加完 style.css 文件后，进行历史提交，就可获得一指针。通过这个指针，我们查到它是一个 commit 对象。除了父历史，作者、提交者等信息外，每个 commit 都会包含一个 tree。这个 tree 可视为该次提交的项目快照。

 由 tree 的指针再查找其对象，发现包含里面还包含了两个子 tree 和两个 blob 文件，分别指向了 images、styles 目录和 readme、index.html 文件。子树对象又包含各自的 blob 文件。打开 blob 文件，正是上面给出的工程文件。

 可见，在 Git 的每次历史提交中，文件目录 以 tree 对象保存，文件则以 blob 对象保存在 tree 中，所有的 tree 和 blob 又保存在一个顶级 tree 中，该顶级 tree 则属于一次历史 commit。

![tree&commit&blob](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-git-github-gitlab/tree&commit&blob.JPG)

## 小练习：数一数 tree 的个数

> 新建的 Git 仓库，有且仅有 1 给 commit，仅仅包含/doc/readme，请问含有多少给 tree，多少给 blob？

一个 commit 中含有一个顶级 tree，包含当前快照。快照中 doc 目录保存在一个 tree 里，一个 readme 就是一个 blob。所以一共是两个 tree，和一给 blob。

 整个操作过程如果对照着 objects/目录查看，就可知道，在`add`操作时，Git 先创建了文件 blob 对象。在 commit 后，再创建 commit 对象，顶级 tree 对象和`doc/`目录 tree 对象。你可尝试运行下面命令，看看`objects/`中是否共有这四个对象。

```
find ./git/objects -type f
```
