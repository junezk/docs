# 通过 41 个 问答方式快速了解学习 Git

> 作者：Duomly
>
> 译者：前端小智
>
> 来源：dev.to

------

[阿(a)里(li)云(yun)服务器很便宜火爆，今年比去年便宜，10.24~11.11购买是1年86元，3年229元，可以点击我 进行参与](https://www.aliyun.com/1111/2019/group-buying-share?ptCode=FBEDBE5CCBE365B176BB470C64C499DD647C88CF896EF535&userCode=pxuujn3r&share_source=copy_link)

------

**为了保证的可读性，本文采用意译而非直译。**

#### 1. 你最喜欢的 Git 命令是什么

个人比较喜欢 `git add -p.` 这增加了“补丁模式”的变化，这是一个内置的命令行程序。它遍历了每个更改，并要求确认是否要执行它们。

这个命令迫使咱们放慢速度并检查更改文件。作为开发人员，咱们有时常常急于提交，我自己也经常这样，做完运行 `git add .` 才发现把调试的代码也提交上去了。

#### 2. 为什么你更喜欢直接使用 git 命令

作为开发人员，咱们也经常使用其它命令来做其它事情，也不差用 `git` 的命令来做事。

此外，`git` 命令也是非常短的，非常容易学习，并且使用命令可以了解 `git` 的工作流程，这样也间接改进了开发工作流程。

#### 3. 如何使用 `stage` 命令

`stage`是`add .`的内置别名。

#### 4.如何在分支中保存更改并 `checkout` 到其他分支

因此，可以使用 `git stash` 临时存储更改或提交 WIP,目的是要有未修改前的环境。就我个人而言，我更喜欢使用 WIP 提交而不是 `stash`，因为它们更容易引用和共享。

**WIP = Work in Progress**

研发中的代码想存储起来，但是又避免研发中的代码被合并，开发就会创建一个WIP的分支

**WIP MR**

**WIP MR** 含义是 在工作过程中的合并请求，是一个我们在 GitLab 中避免 MR 在准备就绪前被合并的技术。只需要添加 `WIP:` 在 MR 的标题开头，它将不会被合并，除非你把 `WIP:` 删除。

#### 5.什么时候使用 git stash

发现有一个类是多余的，想删掉它又担心以后需要查看它的代码，想保存它但又不想增加一个脏的提交。这时就可以考虑 `git stash`。

#### 6.如何使用 git 命令

对任何命令使用 `--help`选项，例如，`git stash --help`。

#### 7. 什么是“ git flow”？

Git Flow 定义了一个项目发布的分支模型，为管理具有预定发布周期的大型项目提供了一个健壮的框架，是由 Vincent Driessen 提出的一个 git 操作流程标准、解决当分支过多时 , 如何有效快速管理这些分支。

#### 8.什么是 GitHub flow ？

GitHub flow，顾名思义，就是 GitHub 所推崇的 Workflow。（千万不要理解成 GitHub 上才能用的 Workflow）, 基本上，GitHub Flow 是`master/feature`分支工作流程的品牌名称。

GitHub flow 的核心优势在于其流程带来的自动化可能性，能够做到其它流程无法实现的检查过程，并极大简化开发团队的体力劳动，真正发挥自身的价值。

#### 9.你更喜欢哪种分支策略?

大多数 Git项目都是 “Git flow”。这些项目中只有少数需要这种策略，通常是因为它是版本化的软件。

`master/feature` 分支策略更易于管理，尤其是在刚入门时，如果需要，切换到 “`git flow`” 非常容易。

#### 10. git open 命令是做啥用的

这是一个单独的命令，可以作为 npm 包使用。

#### 11.当在其他分支中添加的文件仍然在工作分支中显示为未跟踪或修改时，如何重置分支

这通常是“工作索引”不干净时切换分支的结果。

在 git 中没有内置的方法来纠正这一点。通常通过确保提示符有一个 “`status`” 指示符并在每次更改分支时运行诸如 `git status` 之类的命令来避免这种情况。这些习惯会让咱们尽早发现这些问题，这样就可以在新的分支上 `stash` 或 `commit` 这些更改。

#### 12. 如何重命名分支?

```
git branch -m current-branch-name new-branch-name
复制代码
```

#### 13. 如何使用 cherry-pick

`git cherry-pick [reference]` 请记住，这是一个重新应用的命令，因此它将更改提交 SHA。

#### 14. 如果从一个分支恢复(例如 `HEAD~3`)，是否可以再次返回到 `HEAD`(比如恢复上一次更新)

在这种情况下，通过运行 `git reset --hard HEAD~1` 立即撤消还原提交（即 `HEAD` 提交）。

#### 15. 什么时候使用 `git pull` 和 `git fetch`？

`git pull`将下载提交到当前分支。记住，`git pull`实际上是 `fetch` 和 `merge` 命令的组合。

`git fetch`将从远程获取最新的引用。

一个很好的类比是播客播放器或电子邮件客户端。咱们可能会检索最新的播客或电子邮件(`fetch`)，但实际上尚未在本地下载播客或电子邮件附件(`pull`)。

#### 16. 为什么有时需要使用 `--force` 来强制提交更改

`rebase` 是一个可以重新提交的命令，它改变了 `SHA1` hash。如果是这样，本地提交历史将不再与其远程分支保持一致。

当这种情况发生时，`push` 会被拒绝。只有在被拒绝时，才应该考虑使用 `git push --force`。这样做将用本地提交历史覆盖远程提交历史。所以可以回过头来想想，想想为什么要使用 `--force`。

#### 17. 可以使用分支合并多个分支，然后将该分支发送给 `master` 吗？

当然可以，在大多数 git 工作流下，分支通常会累积来自多个其他分支的更改,最终这些分支会被合并到主分支。

#### 18. 应该从一个非常老的分支做一个 `rebase` 吗？

除非是迫不得已。

根据你的工作流，可以将旧的分支合并到主分支中。

如果你需要一个最新的分支，我更喜欢 `rebase`。它只提供更改且更清晰的历史记录，而不是来自其他分支或合并的提交。

然而，尽管总是可能的，但是使用 `rebase` 可能是一个痛苦的过程，因为每次提交都要重新应用。这可能会导致多重冲突。如果是这样，我通常使用`rebase --abort` 并使用 `merge` 来一次性解决所有冲突。

#### 19. 使用 `rebase -i` 时，`squash` 和 `fixup` 有什么区别

`squash` 和 fixup 结合两个提交。`squash` 暂停 `rebase` 进程，并允许咱们调整提交的消息。`fixup` 自动使用来自第一次提交的消息。

#### 20. 通常，当使用 `master` 重新建立功能分支时，对于每次提交都需要解决冲突？

是的。由于每次提交的更改都会在 `rebase` 期间重新应用，所以必须在冲突发生时解决它们。

这意味着在提交之前就已经有了提交冲突，如果没有正确地解决它，那么下面的许多提交也可能发生冲突。为了限制这一点，我经常使用 `rebase -i` 来压缩提交历史记录，以便更轻松地使用它。

如果许多提交之间仍然存在冲突，可以使用 `merge`。

#### 21.在与 master 合并之前，有必要更新我的分支吗

根据你的工作流，可以将旧的分支合并到主分支中。如果你的工作流仅使用 "`fast-forward`"合并，那么有必要在合并之前更新你的分支。

**Git fast forward 提交**

多人协同开发，使用 Git 经常会看到警告信息包含术语：`fast forward`, 这是何义？

简单来说就是提交到远程中心仓库的代码必须是按照时间顺序的。

比如 `A` 从中心仓库拿到代码后，对文件 `f` 进行了修改。然后 `push` 到中心仓库。

`B` 在 `A` 之前就拿到了中心仓库的代码，在 `A push` 成功之后也对 `f` 文件进行了修改。这个时候 `B` 也运行 `push` 命令推送代码。

会收到一个类似下面的信息：

```
chenshu@sloop2:~/work/189/appengine$ git pushTo 
ssh://csfreebird@10.112.18.189:29418/appengine.git ! [rejected]       
master -> master (non-fast-forward)error: failed to push some refs to 
'ssh://csfreebird@10.112.18.189:29418/appengine.git'To prevent you from losing 
history, non-fast-forward updates were rejectedMerge the remote changes (e.g. 'git 
pull') before pushing again.  See the'Note about fast-forwards' section of 'git push --help' for details.
复制代码
```

提醒你非快进方式的更新被拒绝了，需要先从中心仓库`pull`到最新版本，`merge`后再 `push`.

`fast forward` 能够保证不会强制覆盖别人的代码，确保了多人协同开发。尽量不要使用 `non fast forward`方法提交代码。

#### 22. 需要使用 GitKraken 这种可视化工具吗

我比较喜欢用命令方式使用 git，因为这使我能够完全控制管理变更，就像使用命令来改进我的开发过程一样。

当然，某些可视化操作(如管理分支和查看文件差异)在GUI中总是更好。我个人认为在合并过程中在浏览器中查看这些内容就足够了。

#### 23. 当提交已经被推送时，可以做一个 `--amend` 修改吗？

可以，`git commit –amend` 既可以对上次提交的内容进行修改，也可以修改提交说明。

#### 24.在做迭代内容时，当完成一个小功能需要先拉一个 `pull request` 请求，还是都做完这个迭代内容后在拉一个 `pull request` 请求

咱们通常做法是，完成一个迭代的内容后在拉一个 `pull request`。然而，如果你某个任务上花了很长时间，先合并做的功能可能是有益的。这样做可以防止对分支的依赖或过时，所以做完一个拉一个**请求**，还是全部做完在拉一个**请求**，这决于你正在进行的更改的类型。

#### 25. 在将分支合并到 `master` 之前，需要先创建一个 `release` 分支吗？

这在很大程度上取决于你们公司的部署过程。创建 `release` 分支对于将多个分支的工作分组在一起并将它们合并到主分支之前进行整体测试是有益的。

由于源分支保持独立和未合并，所以在最后的合并中拥有更大的灵活性。

#### 26. 如何从 master 获取一些提交?比方说，我不想执行最后一次提交，而是进行一次 `rebase`。

假设 `master` 分支是咱们的主分支，咱们不希望有选择地从它的历史记录中提取提交，这会以后引起冲突。

咱们想要 `merge` 或 `rebase` 分支的所有更改。要从主分支之外的分支提取选择提交，可以使用 `git cherry-pick`。

#### 27. 如何在 git 终端配置颜色

默认情况 下git 是黑白的。

```
git config --global color.status auto 
git config --global color.diff auto 
git config --global color.branch auto 
git config --global color.interactive auto
复制代码
```

配置之后，就有颜色了。

#### 28. 有没有更好的命令来替代 git push -force ?

实际上，没有其他方法可以替代 `git push—force`。虽然这样，如果正确地使用 `merge` 或 `rebase` 更新分支，则无需使用 `git push --force`。

只有当你运行了更改本地提交历史的命令时，才应该使用 `git push --force`。

#### 29. 当我在 `git rebase -` 选择`drop`时，是否删除了与该提交相关的代码？

是的。要恢复这段代码，需要在 `reflog` 的 `rebase` 之前找到一个状态。

#### 30. 如何自动跟踪远程分支

通常，当你 `checkout` 或创建分支时，Git 会自动设置分支跟踪。

如果没有，则可以在下一次使用以下命令进行更新时：`git push -u remote-name branch-name`。

或者可以使用以下命令显式设置它：`git branch --set-upstream-to = remote-name / branch-name`

#### 31. 在 `rebase` 分支之前更新分支，是一个好的习惯吗？

我认为是这样的，原因很简单，用`git rebase -i` 组织或压缩提交，首先在更新过程中提供更多的上下文。

#### 32. 有没有一种方法可以将提交拆分为更多的提交（与 `fixup/squash` 相反）？

可以在`rebase -i`过程中使用 `exec` 命令来尝试修改工作索引并拆分更改。还可以使用 `git reset` 来撤消最近的提交，并将它们的更改放入工作索引中，然后将它们的更改分离到新的提交中。

#### 33.有没有办法查看已修复的提交？

**git log**

查看日志，找到对应的修改记录，但是这种查找只能看到文件，而不是文件的内容。

**git blame 文件名**

查看这个文件的修改记录，默认显示整个文件，也可以通过参数 -L ,来检查需要修改的某几行。

如果查看之前提交的内容可以使用 `git show commitId`。

#### 34. rebase --skip 作用是啥？

咱们知道 `rebase` 的过程首先会产生 `rebase` 分支（`master`）的备份，放到（no branch ）临时分支中。再将支线分支（branch）的每一次提交修改，以补丁的形式，一个个的重新应用到主干分支上。这个过程是一个循环应用补丁的过程，期间只要补丁产生冲突，就会停止循环，等待手动解决冲突。这个冲突指的是上一个合并后版本与补丁之间的冲突。

`git rebase --skip` 命令，可以跳过某一次补丁（存在上一轮冲突的解决方案中，已经包含了这一轮的补丁内容，这样会使补丁无效，需要跳过），这个命令慎用。

#### 35. 如何删除远程分支？

可以使用:`git push origin:branch-name-to-remove` 或使用 `-d`选项:g`it push -d origin someother -branch-2` 来删除远程分支。

要删除对远程分支的本地引用，可以运行:`git remote prune origin`。

#### 36. `checkout` 和 `reset` 有什么区别

这两个命令都可以用来撤销更改。`checkout` 可能更健壮，因为它不仅允许撤消当前更改，而且还允许通过检索文件的旧版本撤消一组更改。

默认情况下，`reset`更适合于更改工作索引中更改的状态。因此，它实际上只处理当前的变化。

`git checkout -- file`；撤销对工作区修改；这个命令是以最新的存储时间节点（add和commit）为参照，覆盖工作区对应文件`file`；这个命令改变的是工作区。

`git reset HEAD -- file`；清空 `add` 命令向暂存区提交的关于 `file` 文件的修改（Ustage）；这个命令仅改变暂存区，并不改变工作区，这意味着在无任何其他操作的情况下，工作区中的实际文件同该命令运行之前无任何变化

#### 37. 在正常的工作流程中应该避免使用哪些命令

一些可能会破坏历史记录的内容，例如：

```
git push origin master -f (千万不要这样做)
git revert
git cherry-pick (changes from master)
复制代码
```

在正常的工作流程下，尽量避免直接使用git merge，因为这通常是通过拉请求(pull requests)构建到流程中的。

#### 38. 如果我有一个分支(B)指向另一个分支(A)，而我又有另一个分支(C)，它需要(A)和(B)及 mast 分支的代码，怎么个流程才能更新(C)？

这取决于几件事：

如果 `A` 和 `B` 可以合并到 `master`，刚可以将 `A` 和 `B` 合并到 `master` 中，然后用`master`的更新 `C`。

如果 `A` 和 `B` 不能合并到 `master`，可以简单地将 `B` 合并到 `C` 中，因为 `B` 已经包含了 `A` 的变更。

在极端的情况下，可以将 `A`、`B` 和 `master` 合并到 `C` 中。然而，为了避免冲突，合并的顺序可能很重要。

#### 39. 你使用的别名有哪些

我常用的一些 git 别名如下：

```
alias.unstage reset HEAD --
alias.append commit --amend --no-edit
alias.wip commit -m "WIP"
alias.logo log --oneline
alias.lola log --graph --oneline --decorate --all
复制代码
```

#### 40. 鲜为人知的 git 命令有哪些？

`git bisect` 是查找代码中存在的`bug`的救命工具。虽然只使用过几次，但它的精确度令人印象深刻，节省了大量时间。

`git archive` 是用于打包一组更改的好工具。这有助于与第三方或 `mico-deployment` 共享工作。

`git reflog` 可能是众所周知的，但值得一提，因为它提供了一种在出错时“撤消”命令的好方法。

#### 41. 你能推荐一些关于Git的书籍吗

我建议至少阅读[Pro Git](https://git-scm.com/book/en/v2)的前三章。这些年来，每看到一遍，或多或少都有收获。

<Git 学习指南> 也不错。

------

**代码部署后可能存在的BUG没法实时知道，事后为了解决这些BUG，花了大量的时间进行log 调试，这边顺便给大家推荐一个好用的BUG监控工具 [Fundebug](https://www.fundebug.com/?utm_source=xiaozhi)**。

原文：[dev.to/gonedark/42…](https://dev.to/gonedark/42-git-questions-answered-3npa)