# SVN：

##### 华锐的SVN管理文档，GIT用来管理代码，因为git的branch拉分支速度很快，操作很简单，轻指针。git可以直接在远程仓库进行可视化图形界面代码的修改。

SVN源于CVS，优于CVS：

+ 原子提交
+ 重命名、复制、删除文件等动作都保存在版本历史记录当中
+ 对于二进制文件，使用了节省空间的保存方法：
+ 目录也有版本历史

检出checkout和导出export的区别：检出checkout下来的文件夹有隐藏文件.svn，文件夹的文件是受svn控制的

检出的文件左下角有符号：

+ √ 表示和仓库的一致
+ ！表示修改了
+ +表示增加了
+ -表示减少了

未被控制的文件add以后就能被控制了。

SVN由于是集中式控制，所以只有add和commit两步操作。commit就直接将本地的文件提交到了仓库。

删除文件或目录，不能直接用Windows的删除命令来删除，那样只是暂时的删除。update之后那些文件又会出来，只能用svn的delete命令才能永久删除。

**commit的时候一定要写日志！**commit最好原子提交，把整个顶层目录都提交上去





# GIT：

分布式控制软件，不同于SVN集中式（一个仓库，多个工作目录），运行速度非常快

checkout代码下来进行修改后的代码提交之前需要跟仓库里的最新节点更新一下，因为在自己checkout之后可能也有人checkout了一份并修改了提交，这样代码就会有冲突。

冲突：两个协同开发人员对同一个文件都进行了操作。

当更新了最新的一个节点之后，自己新修改的文件去哪了？**先commit到本地仓库再pull。**

add . 到本地缓存，commit到本地仓库，push到远程仓库

- 使用git，必须要让git知道你是谁，第一件事就是设置名字和邮箱：chenlei chenlei@af.local

> 工作区：.git文件夹所在目录
>
> 暂存区：缓存
>
> 工作库：本地仓库
>
> 对象库：.git文件夹下数据
>
> commit -m 加注释
>
> commit -am 直接add并commit且加注释
>
> git status
>
> git fetch + git marge = git pull
>
> git diff
>
> git reset:将当前branch重置到另一个commit id里去，会把当前的commit给删掉。
>
> git stash：为什么不用commit呢？因为提交是原子操作，只做到一半的代码提交是编译不过的，就提交不到本地仓库里。
>
> git revert:**相比reset工作上用的更多**，用来撤销一个已经提交的快照，reset是把指针向后移动，把当前commit给干掉，revert是向前移动，保留当前的commit，去到之前的另一个快照。revert和reset都是在本地操作的，**本地分支可以用reset，远程分支一定要用revert*
>
> git log:
>
> git cherry-pick:可以选择某一个分支中的一个或几个commit来进行操作。全部合并用marge，部分合并用cherry-pick。
>
> git blame filename:查看文件的每个部分是谁修改的。

另外，我们在使用git pull命令的时候，可以使用--rebase参数，即git pull --rebase,这里表示把你的本地当前分支里的每个提交(commit)取消掉，并且把它们临时 保存为补丁(patch)(这些补丁放到".git/rebase"目录中),然后把本地当前分支更新 为最新的"origin"分支，最后把保存的这些补丁应用到本地当前分支上。

**推荐git pull --rebase**

###### git stash:

> 1 当正在dev分支上开发某个项目，这时项目中出现一个bug，需要紧急修复，但是正在开发的内容只是完成一半，还不想提交，这时可以用git stash命令将修改的内容保存至堆栈区，然后顺利切换到hotfix分支进行bug修复，修复完成后，再次切回到dev分支，从堆栈中恢复刚刚保存的内容。
> 2 由于疏忽，本应该在dev分支开发的内容，却在master上进行了开发，需要重新切回到dev分支上进行开发，可以用git stash将内容保存至堆栈中，切回到dev分支后，再次恢复内容即可。
> 总的来说，git stash命令的作用就是将目前还不想提交的但是已经修改的内容进行保存至堆栈中，后续可以在某个分支上恢复出堆栈中的内容。这也就是说，stash中的内容不仅仅可以恢复到原先开发的分支，也可以恢复到其他任意指定的分支上。git stash作用的范围包括工作区和暂存区中的内容，也就是说没有提交的内容都会保存至堆栈中。

###### git rebase:![git rebase](images\git rebase.png)

rebase会直接把子分支连接到master分支上，以前的commit也会连接上去，这样看下来历史commit会比较清楚。**不要在公共的分支上使用rebase！！！！！！**![git pull](images\git pull.png)

![checkout&branch](images\git diff.png)

![git reset和git revert的区别](images\git reset和git revert的区别.png)

###### git cherry-pick <commit ids>

> git cherry-pick可以选择某一个分支中的一个或几个commit(s)来进行操作合并到当前分支。例如，假设我们有个稳定版本的分支，叫v2.0，另外还有个开发版本的分支v3.0，我们不能直接把两个分支合并，这样会导致稳定版本混乱，但是又想增加一个v3.0中的功能到v2.0中，这里就可以使用cherry-pick了,其实也就是对已经存在的commit 进行再次提交.
>
> C2<---C3     Δ = C3-C2，cherry-pick就是把Δ的变化作用于当前分支上。通常用于特性分支。

###### git tag

每一次的commit都可以设置一个tag标签，push commit的时候也要push tag，tag一般用作版本发布的时候较多。

checkout到A分支之后，从该分支checkout到一个新的分支B，B分支上的代码就是A分支上的。

fetch --all  默认把所有最新的分支都更新下来

git stash之后把工作目录里的改动都存到堆栈里，checkout到一个B分支的时候，再在A的代码的基础上pop出来对代码进行改动，因为master和V3.1.10.1.0的代码不一样，还得对一下两者代码的区别再提交。

提交之前先pull --rebase，以防有他人提交了新的节点

> 二者对比可知，rebase没有产生新的节点，使用rebase的git演进路线(提交树)是一直向前的，这样在版本回退时也很容易，用merge的git路线是跳跃的，如果版本回退你也找不到自己想要的版本，如果在merge时出现了冲突那就麻烦了，当前merge就不能继续进行下去，需要手动修改冲突内容后，**add，commit, push**. 而rebase 操作的话，会中断rebase,同时会提示去解决冲突。**解决冲突后, 再执行 git rebase –continue 继续操作**，再push.
> ————————————————

# Jenkins：

> CI(Continuous Integration)：持续集成
>
> CD(Continuous Delivery):持续交付
>
> CD(Continuous Deployment):持续部署

![CICD](images\CICD.png)

###### CI：

**持续集成**指的是，频繁地（一天多次）将代码集成到主干。将软件个人研发的部分向软件整体部分交付，频繁进行集成以便更快地发现其中的错误。

两个好处：

> 1. 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易；
>
> 2. 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

CI并不能消除bug，而是让他们非常容易发现和改正。CI的目的就是让产品快速迭代，同事还能保持高质量。它的核心措施是代码继承到主干之前必须通过自动化测试，只要有一个测试用例失败，就不能集成。

###### CD:

**持续交付**指的是频繁地将软件的新版本交付给质量团队或者用户以供评审。如果评审通过，代码就进入生产阶段。

### 常驻branch

###### develop：

> 顾名思义即持续开发的分支，我们希望每个开发组都在这个分支上保持线性的持续小步迭代，正常的CodeReview WorkFlow和开发级的自动CI也在这里进行。当开发完一个迭代(Sprint)，开发小组有信心转测时，就将代码合并到 **release** 分支，并要求打一个alpha级的Tag(如5.2.0-alpha)。

###### **release:**

> 顾名思义即用于发布过程的分支，包括开发转测(实际上我们认为这里的测试集成测试)、测试和BugFix以及发布上线的过程，当发布成功时要打一个发布beta Tag(如5.2.1-beta)，并将代码合并到 **master** 分支。

###### master:

> 即有质量保证的、可安全运行的分支，**禁止直接代码提交**，避免被污染，**仅用于代码合并和归集**，在这个分支上的代码应该永远是可用的、稳定的。当需要拉一个特别的开发分时，应该基于 **master**。

当需要fix线上的一个问题时，应该基于上线时的那个**beta Tag**做hotfix。当出现线上Bug需要hotfix时，我们需要在上次上线的Tag处拉一个临时的 hotfix 分支进行修正，或者在**未被其他开发迭代污染**的release分支上直接hotfix上线并合并到master和develop，然后打一个新的Tag(如5.2.2-beta）

