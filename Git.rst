版本库(Repository)又名仓库，可以简单理解成一个目录，里面存放着各种文件，可以被Git进行管理，每个文件的修改、删除，Git都能跟踪。

创建一个版本库::

 $ mkdir repository
 $ cd repository
 $ git init

配置仓库::

 $ git config --global user.name "Your Name"
 $ git config --global user.email email@example.com

简单来说就是先创建目录，然后切到目录下，再把它变成Git可以管理的仓库。此时在这个目录下多了一个.git目录, 在文件夹下是看不到这个目录的，因为它默认是隐藏的，但你可以通过命令行进行访问。

显示中文文件名::

$ git config --global core.quotepath false

添加多个远程仓库

第一种方式：

添加一个远程库 名字不能是origin

git remote add xxxx https://qchailang:abc@gitee.com/qchailang/LearningNotes.git
查看远程库及地址
git remote -v 

拉，推

git pull xxxx    远程分支名：本地分支名

git push xxxx   本地分支名：远程分支名


第二种方式：（好处时，推送时，可以同时推送到另外一个库）

git remote set-url --add origin https://qchailang:abc@gitee.com/qchailang/LearningNotes.git

推送

git remote -v

git push origin master:master


3.取消本地目录下关联的远程库：

git remote remove origin

一般我们开发项目的时候，从远端clone下来，会默认有一个origin 远程库，输入命令

git remote

这里写图片描述

这个远程仓库我们平时可以提交代码。

现在呢，我们这边分内网和外网，部署在阿里云的代码也需要推送到内网的远程库，所以现在的需求如下：

平时提交代码的同时，还要提交一份到自动化部署的远程库，由于分支命名限制，本地还要先合并代码到一个本地的分支（dev）然后由本分支推送到远程库，下面开始我们的表演。

git remote add originName address

执行成功之后再执行

git remote就可以看到两个远程仓库了，名字是可以随便起的，确保自己可以区分就好。

这里写图片描述

地址可以是ssh，也可以是https，保证自己有对应的权限就好了。

现在你就拥有了向另一个远端推送代码的权利。

下面说一下分支，本地有的分支才可以推送到远端，比如你本地有master和prod分支，你可以直接把这两个分支推送过去。

git push originName master

如果远端的分支你没有呢？那么你本地就要新建一个分支

git checkout -b dev

这时候你就切出来了一个dev分支，这个dev分支的内容和你切换分支之前的内容是一致的。

这时候就可以

git push originName dev

这时，新的远端就有了这一分支和内容。以后再推送的时候就可以直接运行这一命令，而且，你向之前分支的推送也不要直接用

git push了，因为这个指令会向两个分支同时推送。

所以要使用

git push origin dev声明远程仓库。

如果你本地开发的分支不是dev，那样就可以先切换到dev，然后把本地的代码和你开发的分支merge一下就可以提交了。
