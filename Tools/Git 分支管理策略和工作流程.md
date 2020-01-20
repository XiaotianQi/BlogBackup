## Git 分支管理策略

首先，项目存在两个长期分支。

- 主分支`master`
- 开发分支`develop`

前者用于存放对外发布的版本，任何时候在这个分支拿到的，都是稳定的分布版；后者用于日常开发，存放最新的开发版。

其次，项目存在三种短期分支。

- 功能分支（feature branch）
- 补丁分支（hotfix branch）
- 预发分支（release branch）

一旦完成开发，它们就会被合并进`develop`或`master`，然后被删除。

### 主分支 Master

首先，代码库应该有一个、且仅有一个主分支。所有提供给用户使用的正式版本，都在这个主分支上发布。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20branch%20master.png)

Git主分支的名字，默认叫做Master。它是自动建立的，版本库初始化以后，默认就是在主分支在进行开发。 

### 开发分支 Develop

主分支只用来分布重大版本，日常开发应该在另一条分支上完成。我们把开发用的分支，叫做Develop。 

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20branch%20master.png)

这个分支可以用来生成代码的最新隔夜版本（nightly）。如果想正式对外发布，就在Master分支上，对Develop分支进行"合并"（merge）。 

```bash
# 创建Develop分支
git checkout -b develop	master	

# 开发完成后,对Develop分支进行合并
git checkout master
git merge --no-ff develop
```

默认情况下，Git执行"快进式合并"（fast-forward merge），会直接将Master分支指向Develop分支。 

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20branch%20dev%201.png)

使用`--no-ff`参数后，会执行正常合并，在Master分支上生成一个新节点。为了保证版本演进的清晰，希望采用这种做法。 

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20branch%20dev%202.png)

### 临时性分支

前面讲到版本库的两条主要分支：Master和Develop。前者用于正式发布，后者用于日常开发。其实，常设分支只需要这两条就够了，不需要其他了。

但是，除了常设分支以外，还有一些临时性分支，用于应对一些特定目的的版本开发。临时性分支主要有三种：

  * 功能（feature）分支

  * 预发布（release）分支

  * 修补bug（fixbug）分支

这三种分支都属于临时性需要，使用完以后，应该删除，使得代码库的常设分支始终只有Master和Develop。

**3.1 功能分支** 

第一种是功能分支，它是为了开发某种特定功能，从Develop分支上面分出来的。开发完成后，要再并入Develop。 

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20branch%20feature.png)

功能分支的名字，可以采用`feature-*`的形式命名。 

```bash
# 创建一个功能分支
git checkout -b feature-x develop
　
# 开发完成后,将功能分支合并到develop分支
git checkout develop
git merge --no-ff feature-x

# 删除feature分支
git branch -d feature-x
```

**3.2 预发布分支** 

第二种是预发布分支，它是指发布正式版本之前（即合并到Master分支之前），可能需要有一个预发布的版本进行测试。 

预发布分支是从Develop分支上面分出来的，预发布结束以后，必须合并进Develop和Master分支。它的命名，可以采用`release-*`的形式。 

```bash
# 创建一个预发布分支
git checkout -b release-1.2 develop

# 确认没有问题后,合并到master分支
git checkout master
git merge --no-ff release-1.2

# 对合并生成的新节点,做一个标签
git tag -a 1.2

# 再合并到develop分支
git checkout develop
git merge --no-ff release-1.2

# 删除预发布分支
git branch -d release-1.2
```

**3.3 修补bug分支** 

最后一种是修补bug分支。软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。

修补bug分支是从Master分支上面分出来的。修补结束以后，再合并进Master和Develop分支。它的命名，可以采用`fixbug-*`的形式。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20branch%20fixbug.png)

```bash
# 创建一个修补bug分支
git checkout -b fixbug-0.1 master

# 修补结束后,合并到master分支
git checkout master
git merge --no-ff fixbug-0.1
git tag -a 0.1.1

# 再合并到develop分支
git checkout develop
git merge --no-ff fixbug-0.1

# 删除修补bug分支
git branch -d fixbug-0.1
```

***

## Git 工作流程

介绍三种广泛使用的工作流程：

* Git flow
* Github flow
* Gitlab flow

三种工作流程，有一个共同点：都采用"功能驱动式开发"（Feature-driven development，简称FDD）。

它指的是，需求是开发的起点，先有需求再有功能分支（feature branch）或者补丁分支（hotfix branch）。完成开发后，该分支就合并到主分支，然后被删除。

***

### Git flow

 最早诞生、并得到广泛采用的一种工作流程，就是[Git flow](http://nvie.com/posts/a-successful-git-branching-model/) 。 

**特点**：

首先，项目存在两个长期分支。

- 主分支`master`
- 开发分支`develop`

前者用于存放对外发布的版本，任何时候在这个分支拿到的，都是稳定的分布版；后者用于日常开发，存放最新的开发版。

其次，项目存在三种短期分支。

- 功能分支（feature branch）
- 补丁分支（hotfix branch）
- 预发分支（release branch）

一旦完成开发，它们就会被合并进`develop`或`master`，然后被删除。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20gitflow.png)

**评价**：

Git flow的优点是清晰可控，缺点是相对复杂，需要同时维护两个长期分支。大多数工具都将`master`当作默认分支，可是开发是在`develop`分支进行的，这导致经常要切换分支，非常烦人。

更大问题在于，这个模式是基于"版本发布"的，目标是一段时间以后产出一个新版本。但是，很多网站项目是"持续发布"，代码一有变动，就部署一次。这时，`master`分支和`develop`分支的差别不大，没必要维护两个长期分支。

***

### Github flow

 [Github flow](http://scottchacon.com/2011/08/31/github-flow.html) 是Git flow的简化版，专门配合"持续发布"。它是 Github.com 使用的工作流程。 

**流程**：

它只有一个长期分支，就是`master`，因此用起来非常简单。

官方推荐的[流程](https://guides.github.com/introduction/flow/index.html)如下。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20githubflow.png)

* 第一步：根据需求，从`master`拉出新分支，不区分功能分支或补丁分支。

* 第二步：新分支开发完成后，或者需要讨论的时候，就向`master`发起一个[pull request](https://help.github.com/articles/using-pull-requests/)（简称PR）。

* 第三步：Pull Request既是一个通知，让别人注意到你的请求，又是一种对话机制，大家一起评审和讨论你的代码。对话过程中，你还可以不断提交代码。

* 第四步：Pull Request被接受，合并进`master`，重新部署后，原来你拉出来的那个分支就被删除。（先部署再合并也可。）

**评价**：

Github flow 的最大优点就是简单，对于"持续发布"的产品，可以说是最合适的流程。

问题在于它的假设：`master`分支的更新与产品的发布是一致的。也就是说，`master`分支的最新代码，默认就是当前的线上代码。

可是，有些时候并非如此，代码合并进入`master`分支，并不代表它就能立刻发布。比如，苹果商店的APP提交审核以后，等一段时间才能上架。这时，如果还有新的代码提交，`master`分支就会与刚发布的版本不一致。另一个例子是，有些公司有发布窗口，只有指定时间才能发布，这也会导致线上版本落后于`master`分支。

上面这种情况，只有`master`一个主分支就不够用了。通常，你不得不在`master`分支以外，另外新建一个`production`分支跟踪线上版本。

***

### Gitlab flow

 [Gitlab flow](http://doc.gitlab.com/ee/workflow/gitlab_flow.html) 是 Git flow 与 Github flow 的综合。它吸取了两者的优点，既有适应不同开发环境的弹性，又有单一主分支的简单和便利。它是 Gitlab.com 推荐的做法。

**上游优先**

 Gitlab flow 的最大原则叫做"上游优先"（upsteam first），即只存在一个主分支`master`，它是所有其他分支的"上游"。只有上游分支采纳的代码变化，才能应用到其他分支。 

**持续发布**：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20gitlabflow1.png)

对于"持续发布"的项目，它建议在`master`分支以外，再建立不同的环境分支。比如，"开发环境"的分支是`master`，"预发环境"的分支是`pre-production`，"生产环境"的分支是`production`。

开发分支是预发分支的"上游"，预发分支又是生产分支的"上游"。代码的变化，必须由"上游"向"下游"发展。比如，生产环境出现了bug，这时就要新建一个功能分支，先把它合并到`master`，确认没有问题，再`cherry-pick`到`pre-production`，这一步也没有问题，才进入`production`。

只有紧急情况，才允许跳过上游，直接合并到下游分支。

**版本发布**：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20gitlabflow2.png)

对于"版本发布"的项目，建议的做法是每一个稳定版本，都要从`master`分支拉出一个分支，比如`2-3-stable`、`2-4-stable`等等。

以后，只有修补bug，才允许将代码合并到这些分支，并且此时要更新小版本号。

***

## Git 使用规范流程

团队开发中，遵循一个合理、清晰的Git使用流程，是非常重要的。否则，每个人都提交一堆杂乱无章的commit，项目很快就会变得难以协调和维护。

 下面是[ThoughtBot](https://github.com/thoughtbot/guides/tree/master/protocol/git) 的Git使用规范流程。 

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/git%20protocol.png)

* 避免在源代码中包含特定于您的开发环境或过程的文件
* 合并后删除本地和远程的 feature branches
* 在 feature branch 中执行进行开发
* 经常性地 rebase，从而整合上游的变更
* 使用 pull request 进行 code reviews

### 新建功能分支

首先，每次开发新功能，都应该新建一个单独的分支（参考**Git分支管理策略**）。

```bash
# 创建一个基于 master 的本地 feature branch
$ git checkout master
$ git pull
$ git checkout -b myfeature
```

### 分支 commit

 分支修改后，就可以提交commit了。 

```bash
$ git add .
$ git status
$ git commit --verbose
```

提交commit时，必须给出完整扼要的提交信息，下面是一个**提交信息范本**。

```text
Present-tense summary under 50 characters

* More information about commit (under 72 characters).
* More information about commit (under 72 characters).

http://project.management-system.com/ticket/123
```

第一行是不超过50个字的提要，然后空一行，罗列出改动原因、主要变动、以及需要注意的问题。最后，提供对应的网址（比如Bug ticket）。

分支开发完成后，很可能有一堆commit，但是合并到主干的时候，往往希望只有一个（或最多两三个）commit，这样不仅清晰，也容易管理。

那么，怎样才能将多个**commit合并**呢？这就要用到 git rebase 命令。

```bash
$ git rebase -i origin/master
```

### 与主干同步

分支的开发过程中，要经常与主干保持同步。

```bash
$ git fetch origin
$ git rebase origin/master
```

### 推送到远程仓库

合并commit后，就可以推送当前分支到远程仓库了。

```bash
$ git push --force origin myfeature
```

`git push`命令要加上`force`参数，因为`rebase`以后，分支历史改变了，跟远程分支不一定兼容，有可能要强行推送。

### code review

提交到远程仓库以后，就可以发出 `Pull Request` 到master分支，然后请求别人进行代码review，确认可以合并到master。 

***

参考：

[Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)，阮一峰

[Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)