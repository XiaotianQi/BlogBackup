Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方法。**直接记录快照，而非差异比较。** 每次提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 快照流。近乎所有操作都是本地执行。

## 概述

### 文件状态 & 工作区域

* 已修改(modified)------工作区(Workspace)
* 已暂存(staged)---------暂存区(Index/Stage)
* 已提交(committed)---仓库(Repository)

 工作目录下的每一个文件都不外乎这两种状态：已跟踪或未跟踪。 

* 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。 
* 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 
* 编辑过某些文件之后，由于自上次提交后对它们做了修改，Git 将它们标记为已修改文件。逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。

所以使用 Git 时，文件的生命周期如下：  

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%201.png)

基本的 Git 工作流程如下：

1. 在工作目录中修改文件；
2. 暂存文件，将文件的快照放入暂存区域；
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20remote.png)

***

**获取 Git 项目仓库**

有两种取得 Git 项目仓库的方法。

* 第一种是在现有项目或目录下导入所有文件到 Git 中；
* 第二种是从一个服务器克隆一个现有的 Git 仓库。 

创建一个项目目录，并进入该目录：

```bash
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

想要对项目进行版本管理，第一件事就是在项目所在目录使用`git init`命令，进行初始化。

该命令将创建一个名为 `.git` 的子目录，这个子目录含有初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干，用来保存版本信息。  但是， 仅仅是做了一个初始化的操作，项目里的文件还没有被跟踪。 

```bash
$ ls .git
$ config  description  HEAD  hooks/  info/  objects/  refs/
```

或者使用`git clone`获取版本库。

***

### 暂存区

所有变动的文件，Git 都记录在一个区域，叫做"暂存区"（英文叫做 index 或者 stage）。等到变动告一段落，再统一把暂存区里面的文件写入正式的版本历史。

通过 `git add` 命令来实现对指定文件的跟踪，添加指定文件到暂存区。

`git add` 是个多功能命令：

* 跟踪新文件
* 把已跟踪的文件放到暂存区
* 合并时，把有冲突的文件标记为已解决状态

`git checkout -- <file>`

* 撤销文件在工作区的修改,从而与上次在暂存区保存的内容保持一致
* 参考暂存区，修改工作区

`git reset HEAD <file>`

* 取消已经暂存的文件,将其由暂存区恢复到工作区

`git checkout [commit] [file]`

* 恢复某个commit的指定文件到暂存区

***

**状态**

通常用`git status`来回答这两个问题：

* 当前做的哪些更新还没有暂存？ 
* 有哪些更新已经暂存起来准备好了下次提交？ 

`git status` 命令的输出十分详细，但其用语有些繁琐。如果你使用 `git status -s` 命令或 `git status --short` 命令，将得到一种更为紧凑的格式输出。运行 `git status -s` ，状态报告输出如下：

```bash
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

* 新添加的未跟踪文件前面有 `??` 标记
* 新添加到暂存区中的文件前面有 `A` 标记
* 修改过的文件前面有 `M` 标记。 `M` 有两个可以出现的位置：
  * 右边的 `M` 表示该文件被修改了但是还没放入暂存区
  * 左边的 `M` 表示该文件被修改了并放入了暂存区。 
  * 例如，上面的状态报告显示： `README` 文件在工作区被修改了但是还没有将修改后的文件放入暂存区,`lib/simplegit.rb` 文件被修改了并将修改后的文件放入了暂存区。而 `Rakefile` 在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录。

***

### 提交更新

现在的暂存区域已经准备妥当可以提交了。在此之前，请一定要确认还有什么修改过的或新建的文件还没有 `git add` 过，否则提交的时候不会记录这些还没暂存起来的变化。这些修改过的文件只保留在本地磁盘。所以，每次准备提交前，先用 `git status` 看下，是不是都已暂存起来了， 然后再运行提交命令 `git commit` 。通过设置用户名和 Email，保存快照的时候，会记录是谁提交的。

暂存区保留本次变动的文件信息，等到修改了差不多了，就要把这些信息写入历史，这就相当于生成了当前项目的一个快照（snapshot）。项目的历史就是由不同时点的快照构成。Git 可以将项目恢复到任意一个快照。快照在 Git 里面有一个专门名词，叫做 commit，生成快照又称为完成一次提交。

保存进暂存区以后，只要`git commit`命令，就同时提交目录结构和说明，生成快照。

```bash
$ git commit						# 启动文本编辑器以便输入本次提交的说明

$ git commit -m [message]			# 提交暂存区到仓库区,当前分支指针移向新创建的快照

$ git commit -a					# 跳过暂存区域,git add与git commit的合并操作
$ git commit -am [message]		# 同上

$ git commit --amend -m [message]	# 修改上一次 commit 的信息

$ $ git commit -v					# 提交时显示所有diff信息

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

提交后会显示，当前是在哪个分支（`master`）提交的，本次提交的完整 SHA-1 校验和是什么（`463dc4f`），以及在本次提交中，有多少文件修订过，多少行添加和删改过。

**NOTE**，提交时记录的是放在暂存区域的快照。 任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。

`git commit` 加上 `-a` 选项，Git 就会跳过暂存区域，自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤。但是，并不能提交未追踪的文件。

***

### 标签

Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。 

一个轻量标签很像一个不会改变的分支——它只是一个特定提交的引用。

然而，附注标签是存储在 Git 数据库中的一个完整对象。它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用 GNU Privacy Guard （GPG）签名与验证。通常建议创建附注标签，这样你可以拥有以上所有信息；但是如果你只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，轻量标签也是可用的。

**创建标签**

```bash
$ git tag [tag]					# 在当前commit,新建一个tag
$ git tag [tag] [commit] 		# 为指定commit,新建一个tag

$ git tag -a [tag] -m [message]	# 附注标签
```

```bash
$ git tag 			# 列出所有tag
$ git show [tag]	# 查看tag 信息与对应的提交信息
```

**推送标签**

默认情况下，`git push` 命令并不会传送标签到远程仓库服务器上。在创建完标签后，必须显式地推送标签到共享服务器上。 这个过程就像推送远程分支一样 `git push origin [tagname]`。 

如果想要一次性推送很多标签，也可以使用带有 `--tags` 选项的 `git push` 命令。 这将会把所有不在远程仓库服务器上的标签全部传送到那里。 

```bash
$ git push [remote] [tag]		# 提交指定tag
$ git push [remote] --tags		# 提交所有tag
```

**删除标签**

要删除掉本地仓库上的标签，可以使用命令 `git tag -d `。

应该注意的是上述命令并不会从任何远程仓库中移除这个标签，必须使用 `git push  :refs/tags/` 来更新远程仓库： 

```bash
$ git tag -d [tag]						# 删除本地tag
$ git push origin :refs/tags/[tagName]	# 删除远程tag
```

**检出标签**

如果想查看某个标签所指向的文件版本，可以使用 git checkout 命令。虽然说这会使仓库处于“分离头指针（detached HEAD）”状态（这个状态有些不好的副作用）。

在“分离头指针”状态下，如果做了某些更改然后提交它们，标签不会发生变化，但新提交将不属于任何分支，并且将无法访问，除非确切的提交哈希。因此，如果需要进行更改——比如说正在修复旧版本的错误——这通常需要创建一个新分支：

```bash
$ git checkout -b [branch] [tag]	# 新建一个分支,指向某个tag
```

当然，如果在这之后又进行了一次提交，`version2` 分支会因为这个改动向前移动，`version2` 分支就会和 `v2.0.0` 标签稍微有些不同，这时就应该当心了。 

***

## commit

在 Git 中任何 已提交的 东西几乎总是可以恢复的。 甚至那些被删除的分支中的提交或使用 `--amend` 选项覆盖的提交也可以恢复。然而，任何未提交的东西丢失后很可能再也找不到了。

### 提交历史

默认不用任何参数的话，`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。 正如你所看到的，这个命令会列出每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。

 git log 的常用选项，列出了目前涉及到的和没涉及到的选项，以及它们是如何影响 log 命令的输出的：

| 选项              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `-p`              | 按补丁格式显示每个更新之间的差异。                           |
| `-n`              | `n` 是整数，表示仅显示最近的若干条提交                       |
| `--stat`          | 显示每次更新的文件修改统计信息。                             |
| `--shortstat`     | 只显示 --stat 中最后的行数修改添加移除统计。                 |
| `--name-only`     | 仅在提交信息后显示已修改的文件清单。                         |
| `--name-status`   | 显示新增、修改、删除的文件清单。                             |
| `--abbrev-commit` | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。            |
| `--relative-date` | 使用较短的相对时间显示（比如，“2 weeks ago”）。              |
| `--graph`         | 显示 ASCII 图形表示的分支合并历史。                          |
| `--pretty`        | 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。 |

常用的选项是 `-p`，用来显示每次提交的内容差异。也可以加上 `-2` 来仅显示最近两次提交：`git log -p -2`。该选项除了显示基本信息之外，还附带了每次 commit 的变化。当进行代码审查，或者快速浏览某个搭档提交的 commit 所带来的变化的时候，这个参数就非常有用了。

也可以为 `git log` 附带一系列的总结性选项。比如说，如果想看到每次提交的简略的统计信息，你可以使用 `--stat` 选项。`--stat` 选项在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或是添加了。在每次提交的最后还有一个总结。 

**定制输出格式**

另外一个常用的选项是 `--pretty`。 这个选项可以指定使用不同于默认格式的方式展示提交历史。这个选项有一些内建的子选项供使用。比如用 `oneline` 将每个提交放在一行显示，查看的提交数很大时非常有用。另外还有 `short`，`full` 和 `fuller` 可以用，展示的信息或多或少有些不同。最有意思的是 format，可以定制要显示的记录格式。

`git log --pretty=format` 常用的选项，列出了常用的格式占位符写法及其代表的意义。

| 选项  | 说明                                        |
| ----- | ------------------------------------------- |
| `%H`  | 提交对象（commit）的完整哈希字串            |
| `%h`  | 提交对象的简短哈希字串                      |
| `%T`  | 树对象（tree）的完整哈希字串                |
| `%t`  | 树对象的简短哈希字串                        |
| `%P`  | 父对象（parent）的完整哈希字串              |
| `%p`  | 父对象的简短哈希字串                        |
| `%an` | 作者（author）的名字                        |
| `%ae` | 作者的电子邮件地址                          |
| `%ad` | 作者修订日期（可以用 --date= 选项定制格式） |
| `%ar` | 作者修订日期，按多久以前的方式显示          |
| `%cn` | 提交者（committer）的名字                   |
| `%ce` | 提交者的电子邮件地址                        |
| `%cd` | 提交日期                                    |
| `%cr` | 提交日期，按多久以前的方式显示              |
| `%s`  | 提交说明                                    |

**作者**和**提交者**之间差别：

* 作者指的是实际作出修改的人
* 提交者指的是最后将此工作成果提交到仓库的人。 

所以，当你为某个项目发布补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者。 

```bash
$ git shortlog -sn		# 显示所有提交过的用户，按提交次数排序
$ git blame [file]		# 显示指定文件是什么人在什么时间修改过
```

当 `oneline` 或 `format` 与另一个 `log` 选项 `--graph` 结合使用时尤其有用。 这个选项添加了一些 ASCII 字符串来形象地展示你的分支、合并历史。

**限制输出长度**

`-<n>` 选项的写法，其中的 `n` 可以是任何整数，表示仅显示最近的若干条提交。不过实践中我们是不太用这个选项的，Git 在输出所有提交时会自动调用分页程序，所以你一次只会看到一页的内容。

另外还有按照时间作限制的选项，比如 `--since` 和 `--until` 也很有用。\

| 选项                  | 说明                               |
| --------------------- | ---------------------------------- |
| `-(n)`                | 仅显示最近的 n 条提交              |
| `--since`, `--after`  | 仅显示指定时间之后的提交           |
| `--until`, `--before` | 仅显示指定时间之前的提交           |
| `--author`            | 仅显示指定作者相关的提交           |
| `--committer`         | 仅显示指定提交者相关的提交         |
| `--grep`              | 仅显示含指定关键字的提交           |
| `-S`                  | 仅显示添加或移除了某个关键字的提交 |

来看一个实际的例子，如果要查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试文件，可以用下面的查询命令：

```bash
$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
```

`git log`的运行过程是这样的：

- 查找HEAD指针对应的分支，eg:master
- 找到master指针指向的快照，eg:785f188674ef3c6ddc5b516307884e1d551f53ca
- 找到父节点（前一个快照）,eg:c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
- 以此类推，显示当前分支的所有快照

```bash
$ git show [commit]				# 显示某次提交的元数据和内容变化
$ git show --name-only [commit]	# 显示某次提交发生变化的文件
$ git show [commit]:[filename]	# 显示某次提交时，某个文件的内容
$ git reflog	# 显示当前分支的最近几次提交				
```

***

### rebase 合并提交

为了便于他人阅读你的提交，也便于`cherry-pick`或撤销代码变化，在发起Pull Request之前，应该把多个commit合并成一个。（前提是，该分支只有你一个人开发，且没有跟`master`合并过。） 

 这可以采用`rebase`命令附带的`squash`操作。

`git rebase`命令的`i`参数表示互动（interactive），这时git会打开一个互动界面，进行下一步操作。

下面采用[Tute Costa](https://robots.thoughtbot.com/git-interactive-rebase-squash-amend-rewriting-history)的例子，来解释怎么合并commit。

```bash
pick 07c5abd Introduce OpenPGP and teach basic usage
pick de9b1eb Fix PostChecker::Post#urls
pick 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend

# Rebase 8db7e8b..fa20af3 onto 8db7e8b
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

上面的互动界面，先列出当前分支最新的4个commit（越下面越新）。每个commit前面有一个操作命令，默认是pick，表示该行commit被选中，要进行rebase操作。

4个commit的下面是一大堆注释，列出可以使用的命令。

- pick：正常选中
- reword：选中，并且修改提交信息；
- edit：选中，rebase时会暂停，允许你修改这个commit（参考[这里](https://schacon.github.io/gitbook/4_interactive_rebasing.html)）
- squash：选中，会将当前commit与上一个commit合并
- fixup：与squash相同，但不会保存当前commit的提交信息
- exec：执行其他shell命令

上面这6个命令当中，squash和fixup可以用来合并commit。先把需要合并的commit前面的动词，改成squash（或者s）。

```bash
pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
s 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```

这样一改，执行后，当前分支只会剩下两个commit。第二行和第三行的commit，都会合并到第一行的commit。提交信息会同时包含，这三个commit的提交信息。

```bash
# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2nd commit message:
Fix PostChecker::Post#urls

# This is the 3rd commit message:
Hey kids, stop all the highlighting
```

如果将第三行的squash命令改成fixup命令。

```bash
pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
f 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```

运行结果相同，还是会生成两个commit，第二行和第三行的commit，都合并到第一行的commit。但是，新的提交信息里面，第三行commit的提交信息，会被注释掉。

```bash
# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2nd commit message:
Fix PostChecker::Post#urls

# This is the 3rd commit message:
# Hey kids, stop all the highlighting
```

[Pony Foo](http://ponyfoo.com/articles/git-github-hacks)提出另外一种合并commit的简便方法，就是先撤销过去5个commit，然后再建一个新的。

```bash
$ git reset HEAD~5
$ git add .
$ git commit -am "Here's the bug fix that closes #28"
$ git push --force
```

squash和fixup命令，还可以当作命令行参数使用，自动合并commit。

```bash
$ git commit --fixup  
$ git rebase -i --autosquash 
```

这个用法请参考[这篇文章](http://fle.github.io/git-tip-keep-your-branch-clean-with-fixup-and-autosquash.html)，这里就不解释了。

***

## 分支

几乎所有的版本控制系统都以某种形式支持分支。使用分支意味着你可以把工作从开发主线上分离开来，以免影响开发主线。 

在进行提交操作时，Git 会保存一个提交对象（commit object）。知道了 Git 保存数据的方式，我们可以很自然的想到——该提交对象会包含一个指向暂存内容快照的指针。但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象，

所谓分支（branch）就是指向某个快照的指针，分支名就是指针名。哈希值是无法记忆的，分支使得用户可以为快照起别名。而且，分支会自动更新，如果当前分支有新的快照，指针就会自动指向它。比如，master
分支就是有一个叫做 master 指针，它指向的快照就是 master 分支的当前快照。

Git 有一个特殊指针`HEAD`， 总是指向当前分支的最近一次快照。另外，Git 还提供简写方式，`HEAD^`指向 `HEAD`的前一个快照（父节点），`HEAD~6`则是`HEAD`之前的第6个快照。

每一个分支指针都是一个文本文件，保存在`.git/refs/heads/`目录，该文件的内容就是它所指向的快照的二进制对象名（哈希值）。

```bash
$ git branch					# 列出所有本地分支
$ git branch -r					# 列出所有远程分支
$ git branch -a					# 列出所有本地分支和远程分支

$ git branch [branch]				# 新建一个分支，但依然停留在当前分支
$ git checkout -b [branch]			# 新建一个分支，并切换到该分支
$ git branch [branch] [commit id]	# 由特定的commit建立新的分支
$ git checkout -b [branch] [tag]	# 新建一个分支，指向某个tag

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

$ git checkout [branch]			# 切换分支
$ git checkout - 				# 切换至之前分支
$ git branch -d [branch]		# 删除分支
$ git branch -D [branch]		# 强制删除分支

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]

$ git branch -m [branch1] [branch2]	# 分支重命名

$ git merge [branch]			# 合并指定分支到当前分支
$ git merge --no-ff [branch]	# 合并时,禁用 fast-foreword模式,且会增加1个 commit


$ git cherry-pick [commit]		# 选择一个commit，合并进当前分支
```

分支指针是动态的。原因在于，下面三个命令会自动改写分支指针。

```text
$ git commit：当前分支指针移向新创建的快照
$ git pull  ：当前分支与远程分支合并后，指针指向新创建的快照
$ git reset [commit_sha]：当前分支指针重置为指定快照
```

### Merge

Git有两种合并：一种是"直进式合并"（fast forward），不生成单独的合并节点；另一种是"非直进式合并"（none fast-forward），会生成单独节点。

前者不利于保持commit信息的清晰，也不利于以后的回滚，建议总是采用后者（即使用`--no-ff`参数）。只要发生合并，就要有一个单独的合并节点。

**fast forward**

* HEAD：指向当前分支
* master：指向提交

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%203.png)

`git merge` ,将dev分支合并到master分支：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%204.png)

master 与 dev 修改的是不同文件，那么使用 Fast-forward 模式进行和并。

master 与 dev 同时对同一个文件(eg:test.txt)进行修改，并且提交，那么就需要手动进行和并：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%205.png)

此时，如果将dev分支合并到master分支，那么会报错。必须对产生冲突的文件(eg:text.txt)进行处理，从而解决merge冲突。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%206.png)

解决冲突后，master分支领先dev分支一个commit。因此，如果此时，将master分支和并到dev时，将是 Fast-forward，不会产生merge冲突。

* fast-forward，并不会增加新的commit，而是将指针指向新的commit

***

**no-ff**

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20merge.png)

***

### stash

如果分支中的内容没有commit的话，那么当切换分支的时候会报错。此时，可以通过`git stash`保存状态，而且不会产生commit。

```bash
# 暂时将未提交的变化移除，稍后再移入
$ git stash					# 保存状态
$ git stash save [message]	# 保存状态

$ git stash pop					# 恢复状态,并删除状态
$ git stash apply				# 恢复状态,不删除状态
$ git stash drop stash@{id}		# 删除状态
$ git stash apply stash@{id}	# 恢复特定状态
$ git stash list				# 状态列表
```

***

### 版本回退

```bash
# 回退到上一版本
$ git reset --hard HEAD^		# 回退1个
$ git reset --hard HEAD^^		# 回退2个
$ git reset --hard HEAD~3		# 回退3个
$ git reset --hard commit_id	# 回退到某一个commit

# 获取操作信息
$ git reflog
```

* git log：记录 commit 信息
* git reflog：记录操作信息

***

## 远程仓库

管理远程仓库包括了解如何添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等等。 

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20remote.png)

**添加远程仓库**

```bash
$ git remote add <shortname> <url>
$ git remote add origin <url>			# origin是约定俗成的名称
```

**克隆远程仓库**

```bash
$ git clone
```

如果你使用 clone 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。

默认情况下，`git clone` 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支（或不管是什么名字的默认分支）。 运行 `git pull` 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。 

克隆版本库的时候，所使用的远程主机自动被Git命名为`origin`。如果想用其他的主机名，需要用`git clone`命令的`-o`选项指定。 

**拉取远程仓库中**

一旦远程主机的版本库有了更新（Git术语叫做commit），需要将这些更新取回本地，这时就要用到`git fetch`命令。 

```bash
$ git fetch [remote-name]
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据。执行完成后，将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

`git fetch`命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。默认情况下，`git fetch`取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。

```bash
$ git fetch [remote-name] [branch-name]
```

`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 git fetch 命令会将数据拉取到你的本地仓库——它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

`git pull`命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。

```bash
$ git pull [remote-name] [branch-name]:[local-branch-name]
```

比如，取回`origin`主机的`next`分支，与本地的`master`分支合并，需要写成下面这样。

```bash
$ git pull origin next:master
```

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

```bash
$ git pull origin next

# 等同于:
$ git fetch origin
$ git merge origin/next
```

上面命令表示，取回`origin/next`分支，再与当前分支合并。实质上，这等同于先做`git fetch`，再做`git merge`，可以理解为`pull = fetch + merge`。

如果当前分支与远程分支存在追踪关系，`git pull`就可以省略远程分支名。

```bash
$ git pull origin
```

上面命令表示，本地的当前分支自动与对应的`origin`主机"追踪分支"（remote-tracking branch）进行合并。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

```bash
$ git pull
```

上面命令表示，当前分支自动与唯一一个追踪分支进行合并。

如果远程主机删除了某个分支，默认情况下，`git pull` 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致`git pull`不知不觉删除了本地分支。

但是，你可以改变这个行为，加上参数 `-p` 就会在本地删除远程已经删除的分支。

```bash
$ git pull -p
# 等同于下面的命令
$ git fetch --prune origin 
$ git fetch -p 
```

在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在`git clone`的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的`master`分支自动"追踪"`origin/master`分支。

**推送到远程仓库**

`git push`命令用于将本地分支的更新，推送到远程主机。它的格式与`git pull`命令相仿。

```bash
$ git push <远程主机名> <本地分支名>:<远程分支名>
```

注意，分支推送顺序的写法是<来源地>:<目的地>，所以`git pull`是<远程分支>:<本地分支>，而`git push`是<本地分支>:<远程分支>。

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

```bash
$ git push origin master
```

上面命令表示，将本地的`master`分支推送到`origin`主机的`master`分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

```bash
$ git push origin :master
# 等同于
$ git push origin --delete master
```

上面命令表示删除`origin`主机的`master`分支。

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

```bash
$ git push origin
```

上面命令表示，将当前分支推送到`origin`主机的对应分支。

如果当前分支只有一个追踪分支，那么主机名都可以省略。

```bash
$ git push
```

如果当前分支与多个主机存在追踪关系，则可以使用`-u`选项指定一个默认主机，这样后面就可以不加任何参数使用`git push`。

```bash
$ git push -u origin master
```

上面命令将本地的`master`分支推送到`origin`主机，同时指定`origin`为默认主机，后面就可以不加任何参数使用`git push`了。

不带任何参数的`git push`，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。如果要修改这个设置，可以采用`git config`命令。

还有一种情况，就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用`--all`选项。

```bash
$ git push --all origin
```

上面命令表示，将所有本地分支都推送到`origin`主机。

如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做`git pull`合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用`--force`选项。

```bash
$ git push --force origin 
```

上面命令使用`--force`选项，结果导致远程主机上更新的版本被覆盖。除非你很确定要这样做，否则应该尽量避免使用`--force`选项。

最后，`git push`不会推送标签（tag），除非使用`--tags`选项。

```bash
$ git push origin --tags
```

`git push [remote-name] [branch-name]`。当想要将 master 分支推送到 `origin` 服务器时（再次说明，克隆时通常会自动帮你设置好那两个名字），那么运行这个命令就可以将你所做的备份到服务器：

```bash
$ git push origin master
```

只有当有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。

**查看远程仓库**

为了便于管理，Git要求每个远程主机都必须指定一个主机名。git remote命令就用于管理主机名。

如果想查看你已经配置的远程仓库服务器，可以运行 `git remote` 命令。它会列出你指定的每一个远程服务器的简写。如果你已经克隆了自己的仓库，那么至少应该能看到 origin ——这是 Git 给你克隆的仓库服务器的默认名字 。

可以指定选项 `-v`，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。

如果你的远程仓库不止一个，该命令会将它们全部列出。

如果想要查看某一个远程仓库的更多信息，可以使用 `git remote show [remote-name]` 命令。

它同样会列出远程仓库的 URL 与跟踪分支的信息。这些信息非常有用，它告诉你正处于 master 分支，并且如果运行 git pull，就会抓取所有的远程引用，然后将远程 master 分支合并到本地 master 分支。它也会列出拉取到的所有远程引用。

`git branch`命令的`-r`选项，可以用来查看远程分支，`-a`选项查看所有分支。本地主机的当前分支是`master`，远程分支是`origin/master`。 

**远程仓库的移除与重命名**

如果想要重命名引用的名字可以运行 `git remote rename [remote-name1] [remote-name2]` 去修改一个远程仓库的简写名。值得注意的是这同样也会修改你的远程分支名字。 

 如果因为一些原因想要移除一个远程仓库——你已经从服务器上搬走了或不再想使用某一个特定的镜像了，又或者某一个贡献者不再贡献了——可以使用 `git remote rm [remote-name]` 。

```bash
$ git clone <版本库的网址>				# 从远程主机克隆一个版本库
$ git clone <版本库的网址> <本地目录名>	 # 克隆,并指定不同的目录名
$ git clone -o <版本库名称> <版本库的网址># 自定义版本库名称。克隆版本库的时候，所使用的远程主机自动被Git命名为origin
```

```bash
$ git remote					  # 列出所有远程主机
$ git remote -v 				  # 查看远程主机的网址
$ git remote show <主机名>			# 查看该主机的详细信息
$ git remote add <主机名> <网址>    # 添加远程主机
$ git remote rm <主机名>			# 删除远程主机
$ git remote rename <原主机名> <新主机名>	# 重命名
```

```bash
$ git fetch <远程主机名>		  	  # 将远程主机更新取回本地,默认所有分支
$ git fetch <远程主机名> <分支名>	# 取回特定分支的更新
```

***

## 其他

### 配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。 

Git 自带一个 `git config` 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置： 

| 位置             | 命令                  | 作用                                 |
| ---------------- | --------------------- | ------------------------------------ |
| `/etc/gitconfig` | `git config --system` | 系统上每一个用户及他们仓库的通用配置 |
| `~/.gitconfig`   | `git config --global` | 只针对当前用户                       |
| `.git/config`    | `git config --local`  | 只针对该仓库                         |

 每一个级别覆盖上一级别的配置，所以 `.git/config` 的配置变量会覆盖 `/etc/gitconfig` 中的配置变量。 

```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"

# 配置默认文本编辑器,默认是vim
$ git config --global core.editor emacs
```

***

### 忽略文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。在这种情况下，我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件模式。要养成一开始就设置好 `.gitignore` 文件的习惯，以免将来误提交这类无用的文件。 

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 

* 星号（`*`）匹配零个或多个任意字符；
* `[abc]` 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。
* 使用两个星号（`*`) 表示匹配任意中间目录，比如 `a/**/z` 可以匹配 `a/z` , `a/b/z` 或 `a/b/c/z` 等。 

```text
*.txt		# 忽略所有以.txt结尾的文件
!test.txt	# 排除 test.txt 文件
/Test		# 仅仅忽略项目根目录下的 Test文件,不包括subdir/Test 文件
/*/test.txt # 忽略 子目录下的test.txt文件
/**/test.txt# 忽略所有目录下的test.txt文件
testdir/	# 忽略 test/ 目录下所有文件
doc/*.txt	# 忽略 doc/*.txt 文件,但不包括 doc/subdir/*.txt 下的文件
doc/*/*.txt	# 忽略doc子目录
doc/**/*.txt# 忽略doc下所有目录
```

***

### 恢复删除的文件

```bash
# 仅删除,未提交
$ git rm test.txt

# 恢复
$ git reset HEAD test.txt
$ git checkout -- test.txt
```

```bash
# 仅删除,未提交
$ rm test.txt

# 恢复
$ git checkout -- test.txt
```

```bash 
# 删除，且已 commit 
$ git rm test.txt
$ git commit -m "delete test.txt"

# 操作,通过指定 commit 找回文件
$ git log
$ git checkout commit-id test.txt
```

***

### 移除文件或跟踪

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会看到：`Changes not staged for commit`，也就是未暂存。

`git rm` VS `rm`：

`git rm` 作用如下：

- 删除文件
- 将被删除的文件纳入暂存区

通过系统的`rm`命令删除文件时，未纳入缓存区。如果想要 commit 提交的话，再运行 `git rm` 记录此次移除文件的操作 。或者，那么运行`git add`，将删除**操作**纳入缓存区。

下一次提交时，该文件就不再纳入版本管理了。如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 `-f`（译注：即 force 的首字母）。这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被 Git 恢复。 

另外一种情况是，想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当忘记添加 `.gitignore` 文件，不小心把一个很大的日志文件或一堆 `.a` 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 `--cached` 选项：

```bash
$ git rm --cached README
```

`git rm` 命令后面可以列出文件或者目录的名字，也可以使用 `glob` 模式。 比方说：

```bash
$ git rm log/\*.log
```

注意到星号 `*` 之前的反斜杠 `\`， 因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开。 此命令删除 `log/` 目录下扩展名为 `.log` 的所有文件。 类似的比如：

```bash
$ git rm \*~
```

该命令为删除以 `~` 结尾的所有文件。

`git mv` 与 `mv` 与上同理。

```bash
$ git mv [file-original] [file-renamed]	# 重命名文件,并且将这个文件放入暂存区
```

运行 `git mv` 就相当于运行了下面三条命令：

```bash
$ mv README.md README
$ git rm README.md
$ git add README
```

***

### 查看文件的修改内容

`git diff`，尽管 `git status` 已经通过在相应栏下列出文件名的方式回答了这个问题，`git diff` 将通过文件补丁的格式显示具体哪些行发生了改变。 

```bash
# 显示今天写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```

**工作区和暂存区之间的差别**

要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 `git diff`。 此命令比较的是工作目录中当前文件和暂存区域快照之间的差异， 也就是修改之后还没有暂存起来的变化内容。 

若要查看已暂存的将要添加到下次提交里的内容，可以用 `git diff --cached` 命令。（Git 1.6.1 及更高版本还允许使用 `git diff --staged`，效果是相同的，但更好记些。） 

**工作区和最新提交之间的差别**

```bash
$ git diff HEAD
```

**暂存区和最新提交之间的差别**

```bash
$ git diff --cached
```

***

### 撤消操作

```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

**重新提交**

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。此时，可以运行带有 `--amend` 选项的提交命令尝试重新提交：

```bash
$ git commit --amend
```

这个命令会将暂存区中的文件提交。 如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令），那么快照会保持不变，而你所修改的只是提交信息。

文本编辑器启动后，可以看到之前的提交信息。编辑后保存会覆盖原来的提交信息。

**取消暂存的文件**

恢复暂存区的指定文件到工作区。

使用 `git reset HEAD <file>...` 可以取消文件暂存。 

**撤消对文件的修改**

如果不想保留文件的修改怎么办？ 该如何撤消修改，将它还原成上次提交时的样子（或者刚克隆完的样子，或者刚把它放入工作目录时的样子）？  使用`git checkout -- <file>...`

***

**参考：**

[《Git Pro》](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)

[Git 原理入门](http://www.ruanyifeng.com/blog/2018/10/git-internals.html)，阮一峰

[Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)，阮一峰

