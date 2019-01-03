# Git工作流的最佳实践总结

Git作为一个目前非常流行的版本管理工具，深受开发者的喜爱。那么怎样才能将Git的作用发挥的更好呢？我根据实际的项目经验，归纳总结了以下Git工作流的最佳实践。这里所谓的最佳，是经过多次项目经验后，根据自身的情况总结出来的我认为最为合理的方案。如果您有不同的见解，欢迎留言讨论，我们共同进步：）
Git工作流的最佳实践方案包括如下四个步骤：

1\. 根据task创建对应的branch
---------------------

当我们开始针对一个task编码之前，首先第一步应该要创建一个新的branch，然后checkout到这个新的branch上开始编码。我们不应该直接在master上进行新的task的编码工作，尤其是在团队成员较多的情况下。团队的每个成员都应工作于自己新创建的branch上，而不会操作master分支，这样做的好处在于master始终处于一种“干净”的状态，不会因为多人的同时操作而造成过多的冲突，同时也降低了master被误操作的可能性。具体的操作如下：

    //切换到master分支；
    git checkout master 
    //拉取master远程分支的代码；
    git pull origin master 
    //创建新的分支并切换到新的分支上。
    git checkout -b <my branch name> 

2\. 在新建的分支上编码，push代码到远程分支上
--------------------------

分支创建完毕之后，我们就可以开始在branch上进行编码了。这是我们完成task的最主要的阶段，绝大部分的工作在此阶段完成，同时它应该也是持续时间最长的阶段。它的主要任务就是完成task的编码工作，并最终将代码push到当前分支对应的远程分支上去。  
首先看一下这个阶段Git工作的命令流，示例如下：

    //创建新的branch后的第一天工作结束时，首先将改动的代码放入index区；
    git add . 
    //然后提交代码到本地仓库；
    git commit  -m "The first commit message"
    //第二天开始工作前，切换到master分支；
    git checkout master 
    //从master的远程分支拉取代码；
    git pull origin master 
    //切换到task所在的本地分支；
    git checkout <my branch name> 
    //将master上的最新的代码合并到当前分支上，这里的-i的作用是将我们 当前分支之前的commit压缩成为一个commit，这样做的好处在于当我们之后创建pull request并进行相应的code review的时候，代码的改动会集中在一个commit，使得code review更直观方便；
    git rebase -i master  
    此处如果有冲突就解决才冲突git rebase --continue，解决不了就返回用命令git rebase --abort
    
    //第二天工作结束之后，将第二天的改动放入index区；
    git add . 
    //提交代码到当前branch的本地仓库；
    git commit  -m "The second commit message"
    //第三天开始工作前，
    git checkout master //同上第二天
    git pull origin master //同上第二天
    git checkout //同上第二天
    git rebase -i master  //同上第二天
    
    //第三天工作结束之后，
    git add . //同上第二天
    git commit  -m "The third commit message" //同上第二天
    ..........
    //最后，当task的所有编码完成之后，将代码push到远程分支。
    git push --set-upstream origin <my branch name>

通过以上的工作流可以看出，我们在完成task期间所有的代码都始终存放在我们新创建的branch的本地仓库上。只有当所有的编码工作完成之后，才会将最终的代码push到当前分支的远程仓库。这样做的好处在于，我们push到远程的代码，也就是之后会通过pull request被code review的代码，始终是针对一个单独task的完整代码，这将有利于之后code review的执行。不过这样做同时可能会存在一个缺点，那就是最终一次push的代码可能会非常庞大，这就要求我们对于task粒度的把握应该更合适。我们不应针对一个非常大的task创建branch，完成编码，而是应该尽可能的将task分解成一个个粒度较小的子task，进而针对子task创建branch完成编码的工作。这是一项非常有技巧的工作，需要丰富的实践经验，它也不是本节要讨论的内容，不再赘述。  
3\. 创建pull request, 进行code review  
（这块没搞过，但是查了资料）  
[http://blog.jobbole.com/76854/](http://blog.jobbole.com/76854/) Git工作流指南：Pull Request工作流

[http://www.egouz.com/topics/9303.html](http://www.egouz.com/topics/9303.html) BitBucket:免费源代码托管平台
--------------------------------------------------------------------------------------------------

当所有的代码都已经被push到远程分支后，这时我们还不可以将代码合并到master上去，我们应该要做的是创建pull request。pull request的作用在于它可以使得代码在merge到master分支之前，能够被团队成员code review，从而提高代码的质量以及降低出错的概率。实际项目中我们使用jira来帮助我们创建相应的pull request，当然Github本身就具备创建pull request的功能。创建pull request的操作非常简单，无非就是点击创建pull request的按钮，填写comment信息，并输入可以进行code review的成员名称。当pull request创建完成之后，所有可以进行code review的团队成员都会收到邮件通知，并通过相应的pull request的链接查看代码的改动，从而完成code review的工作。这个步骤没有实际的Git指令的操作。

4\. 合并代码到master，并删除之前创建的branch
------------------------------

当所有的reviewer都结束了code review，且都已经将pull request标注为approved状态的时候，我们就可以将branch合并到master上去，并最终push到远程master分支。示例如下：

    git checkout master //切换到master分支；
    git merge <my branch name>//合并之前创建的分支的代码到master分支上；
    git push origin master//将master的代码push到master的远程分支；
    git branch -d <my branch name>//删除之前创建的分支。

经过以上四个步骤之后，一个task的 Git的工作流就结束了。之后，我们可以愉快的开始下一个task了～～