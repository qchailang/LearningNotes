版本库(Repository)又名仓库，可以简单理解成一个目录，里面存放着各种文件，可以被Git进行管理，每个文件的修改、删除，Git都能跟踪。

创建一个版本库::

 $ mkdir repository
 $ cd repository
 $ git init

配置仓库::

 $ git config --global user.name "Your Name"
 $ git config --global user.email email@example.com

简单来说就是先创建目录，然后切到目录下，再把它变成Git可以管理的仓库。此时在这个目录下多了一个.git目录, 在文件夹下是看不到这个目录的，因为它默认是隐藏的，但你可以通过命令行进行访问。

