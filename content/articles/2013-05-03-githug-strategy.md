---
title: githug通关攻略
date: 2013-05-03 
category: linux
tags: git
---

这货叫做`githug`，而不是大家熟悉的`github`，其主要目的是通过游戏的形式来让我们练习git的使用。
<!-- excerpt -->

<br/>

##安装githug

githug是ruby写的一个应用。所以先要安装ruby，然后输入


    :::sh
    gem install githug

然后就可以在你觉得合适的目录输入`githug`

    :::sh
    ********************************************************************************
    *                                    Githug                                    *
    ********************************************************************************
    No githug directory found, do you wish to create one? [yn]  y

强烈建议把git的编辑器转换为vim

    :::sh
    git config --global core.editor "/usr/local/bin/vim"

<br/>

##Level 1

    :::sh
    Level: 1
    Difficulty: *
    
    A new directory, git_hug, has been created; initialize an empty repository in it

开始总是简单的，这题是如何新建一个仓库

    :::sh
    cd git_hug
    git init
    githug
    
<br/>

##Level 2

    :::sh
    Level: 2
    Difficulty: *
    
    There is a file in your folder called README, you should add it to your staging area
    Note: You start each level with a new repo. Don't look for files from the previous one

基础题，教你如何提交文件到暂存区

    ::: sh
    git add README
    githug

<br/>

##Level 3

    :::sh
    Level: 3
    Difficulty: *

The README file has been added to your staging area, now commit it.

放到暂存区的文件需要`commit`，才能提交到本地版本库，可以加`-m`，写上批注

    :::sh
    git commit -m "init"
    githug

<br/>

##Level 4

    :::sh
    Level: 4
    Difficulty: *

    Set up your git name and email, this is important so that your commits can be identified

设置自己的用户名和邮箱。

    :::sh
    #如果想作为全局设置 可以加上 --global
    git config user.name xxx
    git config user.email "xxx@xxx.com"
    githug

<br/>

##Level 5

    :::sh
    Level: 5
    Difficulty: *

    Clone the repository at https://github.com/Gazler/cloneme

获取远程版本库到本地

    :::sh
    git clone https://github.com/Gazler/cloneme
    githug

<br/>

##Level 6

    :::sh 
    Level: 6
    Difficulty: *

    Clone the repository at https://github.com/Gazler/cloneme to 'my_cloned_repo'

下载版本库的同时改变目录的名字为`my_cloned_repo`

    :::sh
    git clone https://github.com/Gazler/cloneme my_cloned_repo

<br/>

##Level 7

    :::sh
    Level: 7
    Difficulty: **

    The text editor 'vim' creates files ending in .swp (swap files) for all files that
    are currently open.  We don't want them creeping into the repository.  Make this 
    repository ignore .swp files.

如果你想忽略某些文件或者目录，可以在工作目录里添加`.gitignore`文件并编写过滤规则。这里需要忽略所有后缀为swp的文件

    :::sh
    vim .gitignore

    :::sh .gitignore
    *.swp

<br/>

##Level 8

    :::sh
    Level: 8
    Difficulty: *

    There are some files in this repository, one of the files is untracked, which file is it?

这题要求我们找出还没有被放进暂存区的文件。首先我们需要使用`git status`看看文件没有添加，然后githug 答题就可以了。

    :::sh
    git status
    githug
    database.yml

<br/>

##Level 9

    :::sh
    Level: 9
    Difficulty: **

    A file has been removed from the working tree, however the file was not removed 
    from the repository.  Find out what this file was and remove it.

工作目录已经把文件删除了，但是暂存区还有记录，现在我们需要把文件在暂存区删除，避免提交到版本库中。

    :::sh
    git status #先看看有哪个文件需要执行删除操作
    git rm deleteme.rb
    githug

<br/>

##Level 10

    :::sh
    Level: 10
    Difficulty: **

    A file (deleteme.rb) has accidentally been added to your staging area, find out 
    which file and remove it from the staging area.  *NOTE* Do not remove the file system, only from git.

有一个文件deleteme.rb已经放到暂存库中，但我们后悔了，想先不提交这个文件，也就是改变其为untracked状态，我们可以这么做

    :::sh
    git status # 确定是哪个文件
    git rm --cache deleteme.rb
    git status # 确定其状态
    githug

<br/>

##Level 11

    :::sh
    Level: 11
    Difficulty: **

    You've made some changes and want to work on them later. You should save them, but don't commit them

想暂存到目前为止作出的修改，但是不想提交到版本库，git提供了一个命令`git stash`，所有修改会提交到栈中，相关的恢复操作可以查看帮助文档

    :::sh
    git stash
    githug

<br/>

##Level 12

    :::sh
    Level: 12
    Difficulty: ***

    We have a file called oldfile.txt. We want to rename it to newfile.txt and stage this change.

如果想改变文件名字，而且同时令暂存区产生同样的变化，可以这样

    :::sh
    git mv oldfile.txt newfile.txt

<br/>

##Level 13

    :::sh
    Level: 13
    Difficulty: **

    You will be asked for the first 7 chars of the hash of most recent commit. 
    You will need to investigate the logs of the repository for this.

查看最近提交的7位哈希值

    :::sh
    git log
    githug
    ac91aad #注意每个人不一样

<br/>

##Level 14

    :::sh
    Level: 14
    Difficulty: **

    We have a git repo and we want to tag the current commit with new_tag.

为版本库添加一个tag，通常我们会为发布的版本添加一个tag以作标识。

    :::sh
    git tag new_tag
    githug

<br/>

##Level 15

    :::sh
    Level: 15
    Difficulty: **

    The README file has been committed, but it looks like the file `forgotten_file.rb`
    was missing from the commit.  Add the file and amend your previous commit to include it.

有时候，我们在提交代码到版本库后才发现还有一个文件需要修改才能提交，这时候我们不想再增加一条提交记录，想在原来的提交记录里面加上这个文件，我们就可以像下面这样做，但是如果提交到远程版本库就不可能这样做了。

    :::sh
    git add forgotten_file.rb
    git commit --amend #跳到令一个页面 <C-x> 退出即可

<br/>

##Level 16

    :::sh
    Level: 16
    Difficulty: **

    There are two files to be committed.  The goal was to add each file as a separate commit, 
    however both were added by accident.  Unstage the file `to_commit_second` using the reset command (don't commit anything)

不小心把两个文件都提交上暂存库了，现在想取消一个提交，如何做？

    :::sh
    git status #确定哪个文件
    git rm --cache to_commit_second.rb
    githug

<br/>

##Level 17

    :::sh
    Level: 17
    Difficulty: **

    You committed too soon. Now you want to undo the last commit, while keeping the index.

git总有后悔药可吃，这次我们需要撤销最后一次提交，也就是回退到执行commit命令之前。

    :::sh
    git reset --soft HEAD~1

`git reset` 就是版本回退，`--soft`就是本地目录不会变化，只是暂存发生变化

<br/>

##Level 18

    :::sh
    Level: 18
    Difficulty: ***

    A file has been modified, but you don't want to keep the modification.  Checkout the `config.rb` file from the last commit.

本地目录的文件已经发生变化，但是你又不想修改这个文件了，想还原回去，可以这样做。

    :::sh
    git checkout config.rb
    githug

<br/>

##Level 19

    :::sh
    Level: 19
    Difficulty: **

    This projects has a remote repository.  Identify it.

想知道版本库是否有远程仓库，远程仓库叫什么别名？可以这样查看

    :::sh
    git remote -v
    githug
    my_remote_repo

<br/>

##Level 20

    :::sh
    Level: 20
    Difficulty: **

    The remote repositories have a url associated to them.  Please enter the url of remote_location

想知道remote_location这个远程仓库的地址，简单。

    :::sh
    git remote -v
    githug
    https://github.com/githug/not_a_repo

<br/>

##Level 21

    :::sh
    Level: 21
    Difficulty: **

    You need to pull changes from your origin repository.

这个任务是让你同步远程版本库，先要知道远程版本库的名字，`git remote -v`可得

    :::sh
    git remote -v
    git pull origin master
    githug

<br/>

##Level 22

    :::sh
    Level: 22
    Difficulty: **

    Add a remote repository called `origin` with the url `https://github.com/githug/githug`

向本地仓库添加远程版本库，名叫`origin`

    :::sh
    git remote add origin https://github.com/githug/githug

<br/>

##Level 23

    :::sh
    Level: 23
    Difficulty: ***

    Your local master branch has diverged from the remote origin/master branch. 
    Rebase your commit onto origin/master and push it to remote.

合并分支，我们可以使用rebase，这个题目里，本地master提交了Thrid commit，而远程分支也提交了Fourth commit，现在需要把远程更新合并到本地，可以使用下面的命令。

    :::sh
    git rebase origin/master
    git push origin master

当然我们可以使用`git pull`之类的，但是那样会产生新提交，而如果我们想保持提交的一致性，可以使用这个。

<br/>

##Level 24

    :::sh
    Level: 24
    Difficulty: **

    There have been modifications to the app.rb file since your last commit.  Find out which line has changed.

要查看文件哪一行有改动，使用`git diff <文件名>` 

    :::sh
    git diff app.rb
    githug
    26

看到23行向下数第三行开始有改动，所以是23+3

<br/>

##Level 25

    :::sh
    Level: 25
    Difficulty: **

    Someone has put a password inside the file 'config.rb' find out who it was

有人在config.rb里面填写了明文密码，不可容忍，要揪出是谁干的。

    :::sh
    git blame config.rb

    :::sh
    ^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000  1) class Config
    70d00535 (Bruce Banner      2012-03-08 23:07:41 +0000  2)   attr_accessor :name, :password
    97bdd0cc (Spider Man        2012-03-08 23:08:15 +0000  3)   def initialize(name, password = nil, options = {})
    ^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000  4)     @name = name
    97bdd0cc (Spider Man        2012-03-08 23:08:15 +0000  5)     @password = password || "i<3evil"
    00000000 (Not Committed Yet 2013-05-03 17:02:57 +0800  6) 
    09409480 (Spider Man        2012-03-08 23:06:18 +0000  7)     if options[:downcase]
    09409480 (Spider Man        2012-03-08 23:06:18 +0000  8)       @name.downcase!
    09409480 (Spider Man        2012-03-08 23:06:18 +0000  9)     end
    70d00535 (Bruce Banner      2012-03-08 23:07:41 +0000 10) 
    ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 11)     if options[:upcase]
    ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 12)       @name.upcase!
    ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 13)     end
    ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 14) 
    ^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000 15)   end
    ^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000 16) end

看到原来是Spider Man干的好事，汗。

<br/>

##Level 26

    :::sh
    Level: 26
    Difficulty: *

    You want to work on a piece of code that has the potential to break things, create the branch test_code

建立分支，必会的内容

    :::sh
    git branch test_code

<br/>

##Level 27

    :::sh
    Level: 27
    Difficulty: **

    Create and switch to a new branch called 'my_branch'.  You will need to create a branch like you did in the previous level

建立分支并切换到分支，也很简单

    :::sh
    git checkout -b my_branch

<br/>

##Level 27

    :::sh
    Level: 28
    Difficulty: **

    You need to fix a bug in the version 1.2 of your app. Checkout the tag v1.2

切换到v1.2这个tag

    :::sh
    git checkout v1.2

<br/>

##Level 29

    :::sh
    Level: 29
    Difficulty: ***

    You forgot to branch at the previous commit and made a commit on top of it. Create branch 'test_branch' at the commit before the last

提交了代码到版本库才记起要在前一个版本创建分支？有办法解决。

    :::sh
    git log #查看前一个分支的hash值，前7位就可以了
    git branch test_branch -v ee7937e

<br/>

    ##Level 30

    :::sh
    Level: 30
    Difficulty: **

    We have a file in the branch 'feature'; Let's merge it to the master branch

合并分支

    :::sh
    git merge feature

<br/>

##Level 31

    :::sh
    Level: 31
    Difficulty: ***

    Your new feature isn't worth the time and you're going to delete it. But it has 
    one commit that fills in README file, and you want this commit to be on the master as well

你不想要new-feature分支，但是想要其中一个文件README，如何做？

    :::sh
    git checkout new-feature
    git log

可以看到在第三次提交的时候加上了README.md，我们正是需要这个文件。我们需要记下其前7位哈希值。

    :::sh
    git checkout master
    git cherry-pick ca32a6d
    githug

这样就可以单独合并另一个分支的其中一次提交了。


<br/>

    ##Level 32

    :::sh
    Level: 32
    Difficulty: ***

    Correct the typo in the message of your first commit.

修改之前提交的批注。很多时候我们都需要用到的。

    :::sh
    git rebase -i HEAD~2 

跳转到编辑界面，把`First coommit`前面的`pick`改为`reword`，这一步修改了，**批注才会被修改**。

<br/>

##Level 33

    :::sh
    Level: 33
    Difficulty: ****

    You have committed several times but would like all those changes to be one commit

将提交合并，这是另一个非常实用的功能。

    :::sh
    git rebase -i HEAD~3

将后面两项的`pick`改为`s`，然后保存，接着可以再修改评论，退出后发现提交已经合并了。

<br/>

##Level 34

    :::sh
    Level: 34
    Difficulty: ***

    Merge all commits from the long-feature-branch as a single commit.

    合并分支时，把分支的多次提交变成一次提交。

    :::sh
    git merge --squash long-feature-branch
    git commit -m"add file 3"

<br/>

##Level 35

    :::sh
    Level: 35
    Difficulty: ****

    You have committed several times but in the wrong order. Please reorder your commits

提交的顺序错了，这个也可以纠正，很神奇。

    :::sh
    git rebase -i HEAD~3

把`Second commit`和`Third commit`调换一下位置就可以了。

<br/>

##Level 36

    :::sh
    Level: 36
    Difficulty: ***

    A bug was introduced somewhere along the way.  You know that running 'ruby prog.rb 5' should output 15.  You can also run 'make test'.  What are the first 7 chars of the hash of the commit that introduced the bug.

这个技巧对找出错误提交很有用，其原理相当于截半查找法。

我们从First commit到HEAD进行查找，

    :::sh
    git bisect start
    git bisect good f608824
    git bisect bad master

这三步的意思是告诉bisect从哪个提交开始，到哪个提交结束。

    :::sh
    ruby prog.rb 5 #看结果是否 15 
    git bisect good #bad还是good看你的运行结果

bisect开始运行，它会自动跳到good和bad的提交之间，让你验证这个提交是否错误的提交。若是错误的，输入`git bisect bad`，否则输入`git bisect good`，周而复始，直到准确定位到提交。

    :::sh
    18ed2ac1522a014412d4303ce7c8db39becab076 is the first bad commit
    githug
    18ed2ac

<br/>

##Level 37

    :::sh
    Level: 37
    Difficulty: ****

    You've made changes within a single file that belong to two different features, 
    but neither of the changes are yet staged. Stage only the changes belonging to the first feature.

feature.rb 文件只想提交其中一部分修改，可以如何做？

    :::sh
    git add feature.rb -e

删除不想提交的行就可以了。

<br/>

##Level 38

    :::sh
    Level: 38
    Difficulty: ****

    You have been working on a branch but got distracted by a major issue and forgot the name of it. Switch back to that branch

忘记上一个提交的branch是什么？下面的答案可以帮你

    :::sh
    6876e5b HEAD@{1}: checkout: moving from solve_world_hunger to kill_the_batman
    git checkout solve_world_hunger

<br/>

##Level 39

    :::sh
    Level: 39
    Difficulty: ****

    You have committed several times but want to undo the middle commit.

想把中间的提交干掉，一样可以

    :::sh
    git revert df2c0c6

注意到撤销操作也作为一个新的提交，而不是在提交中删除。

<br/>

##Level 40

    :::sh
    Level: 40
    Difficulty: ****

    You decided to delete your latest commit by running `git reset --hard HEAD^`. 
    (Not a smart thing to do.)  You then change your mind, and want that commit back.
    Restore the deleted commit.

用狠方法将最新一次提交干掉了，还能找回来吗？答案是可以的。

    :::sh
    git reflog
    git checkout 7775caf

<br/>

##Level 41

    :::sh
    Level: 41
    Difficulty: ****

    You need to merge the current branch (master) with 'mybranch'. But there may be 
    some incorrect changes in 'mybranch' which may cause conflicts.
    Solve any merge-conflicts you come across and finish the merge.

合并分支的时候产生冲突如何处理？把冲突修改好，再提交冲突文件就好了。

    :::sh
    git merge my_branch
    vim poem.txt
    git add poem.txt
    git commit -m"merge mybranch"

<br/>

##Level 42

Final Level 该干嘛干嘛去。

