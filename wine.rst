一、安装Wine（来自官方安装命令）
=============================

1、对于64位系统，添加 32 位架构支持（对于32位的，似乎可以忽略此命令，不过加上也没有错）

终端下执行：sudo dpkg --add-architecture i386

2、添加软件源

终端下逐条执行（默认当前路径为～，执行以下命令过程中，请勿切换路径）：
::

    wget -nc https://dl.winehq.org/wine-builds/Release.key
    sudo apt-key add Release.key
    sudo apt-add-repository https://dl.winehq.org/wine-builds/ubuntu/

3、更新

终端下执行：sudo apt update

4、安装Wine

终端下执行：（三选一）

稳定版：sudo apt install --install-recommends winehq-stable

开发版：sudo apt install --install-recommends winehq-devel

阶段版：sudo apt install --install-recommends winehq-staging

成功安装后，Wine对应的将安装到 /opt/winehq-stable，或/opt/wine-devel，或/opt/wine-staging路径下。

二、安装Wine依赖环境
===================

1、安装winetricks（Wine的辅助配置工具，超级便利）

终端下执行：sudo apt install --install-recommends winetricks

2、安装字体（解决Wine及初始配置界面乱码）

刚安装完Wine后，初始执行界面一般会出现诸如问号方块之类的乱码，为了便于阅读，需完善安装缺失的默认字体。

将以下simfang.ttf、simhei.ttf、simkai.ttf、simsun.ttc字体文件复制到Wine安装路径下的字体目录/opt/wine-stable/share/wine/fonts即可，你也可以将更多字体复制到该目录下。

就我个人而言，我将以上字体文件及微软雅黑字体文件msyh.ttf、msyhbd.ttf 复制到 Wine 字体目录即解决了界面乱码的问题。
 3、安装Wine依赖

一般而言，安装完Wine后，初始执行winecfg或wine或winetricks，会要求下载安装 wine-mono 和 wine-gecko，这是一个相对漫长的过程，并且中途还可能出错，有可能需要反复多次才能下载安装成功。这些依赖文件是下载安装在：~/.cache/wine 目录下的。

在此，我提供一个快速的解决办法：从其它人那里获取或用快速下载工具直接下载下来后，复制到 ~/.cache/wine 路径下即可，如果目录不存在，请自行创建。

最好是将包含wine-mono和wine-gecko的wine目录直接复制到~/.cache/下，合并或覆盖wine目录。

Wine安装使用（适用Linux Mint 19与Ubuntu 18.04下）

4、安装Wine中Windows程序依赖

在此处，必须借助winetricks这个强大的辅助工具了。

你可以在终端下执行 winetricks，在界面中选择相关的依赖库，但相对快捷的是直接将所需依赖作为参数传递给 winetricks ，如下：（对于网络上网友提供的安装方式，本人实际经验是逐个安装有更高的成功率）

终端下分别执行：

winetricks corefonts colorprofile

winetricks fontfix fontsmooth-gray fontsmooth-rgb fontsmooth-bgr

winetricks gdiplus

winetricks d3dx9

winetricks riched20 riched30

winetricks mfc40 mfc42

winetricks vcrun6 vb6run vcrun2003 vcrun2005 vcrun2008

winetricks msxml3 msxml4 msxml6

一般来说，以上依赖也够了，对于其它的依赖，请自行再安装。

这些依赖，将会下载安装到：~/.cache/winetricks

下载和安装将会花费不少时间，中途还可能出现各种问题，还需要反复多次才能成功，在此，分享一个最快速的方法：

即：从其它网友处获取这些依赖或用下载工具下载后复制到该目录下，便能省却过程中的诸多烦恼。

最好是将整个winetricks目录复制到~/.cache/下，合并或覆盖winetricks目录。　　　　

特别注意：以复制方式下载安装，是省却了下载过程中的诸多问题，仍然需要执行以上winetricks命令，将依赖信息合并到Windows的注册表中，路径是在：~/.wine，

另外，对于Windows应用程序所需的字体，是保存在：~/.wine/drive_c/windows/Fonts 路径中，你可以自行将所需的字体文件复制到该目录下即可。

如下：

Wine安装使用（适用Linux Mint 19与Ubuntu 18.04下）

我个人的下载安装清单如下：

Wine安装使用（适用Linux Mint 19与Ubuntu 18.04下）

5、Wine设置

在终端下执行：winecfg，即可打开wine设置窗口界面。
 6、至此，Wine 就完成了比较完善的安装了，就可以根据自己的喜好安装Windows应用程序了。

7、通过Wine安装Windows应用程序有多种方式，一般来说，可以通过鼠标右键点击Windows应用程序，在右键菜单中选择Wine来安装，但更推荐的方式是以Wine添加-删除程序去完成，如下：

终端下执行：wine uninstaller

Wine安装使用（适用Linux Mint 19与Ubuntu 18.04下）

稍等即可出现添加-删除程序的界面，如下：

Wine安装使用（适用Linux Mint 19与Ubuntu 18.04下）

通过界面中的“安装”按钮选择待安装的应用程序，根据安装向导逐步完成安装。

 