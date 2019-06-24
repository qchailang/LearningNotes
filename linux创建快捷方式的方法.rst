 Linux创建桌面快捷方式的三种方法

系统平台：Ubuntu12.04

Linux系统下创建桌面快捷方式可以更快地打开应用。这里总结了三种方法，也是之前查了些资料整理的，跟大家分享一下。顺便说一下，在Linux下打开应用后，左边那个应用栏（就是竖着的，我就把它叫做应用栏了）里会出现相应的程序缩略图，在缩略图上右键出来菜单，菜单里可以选择锁定到应用栏，以后就可以在这里单击应用缩略图来启动应用。

下面正式介绍三种在桌面上创建快捷方式的方法，创建后在桌面上可以看到相应软件的图标，双击就可以运行了。

第一种：

进入 /usr/share/applicatoions，找到所需的软件的快捷方式，拷贝到桌面就可以了。

这种方法不适用所有程序，有的程序不会在这里创建快捷方式。

第二种：

以软件Netanim为例，桌面新建空文件名为Netznim（不要双击打开，要先打开Editor，用它打开），打开Text Editor，打开刚刚创建的空文件，添加内容：

[Desktop Entry]

Type=Application

Version=0.9.4

Name=Netanim

Comment=Run Netanim

Icon=/usr2/ns-allinone-3.25/netanim-3.107/netanim-logo.png

Exec=/usr2/ns-allinone-3.25/netanim-3.107/NetAnim

Terminal=false

然后保存，就可以双击运行程序了。上面这段代码里，有的不是必须的，还有就是这里没有列出所有的配置（个人水平有限 ^_^ ）。里面的Icon指的是程序图标的绝对路径，Exec配置的是所要运行程序的绝对路径。如果想要了解更多的配置，可以查看别的更专业的资料，这里给个参考链接。

 第三种：

还是以Netanim为例，具体根据自己的进行修改

运用软链接的方法

在命令窗口中输入：

ln -s /usr2/ns-allinone-3.25/netanim-3.107/Netanim ~/Desktop

即可在桌面看到相应的快捷方式。这里前一个路径是程序的源路径，第二个是需要创建快捷方式的地方。