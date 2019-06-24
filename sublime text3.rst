注册
====
#. 修改/etc/hosts，添加内容：
::
	127.0.0.1 www.sublimetext.com
	127.0.0.1 sublimetext.com
	127.0.0.1 sublimehq.com
	127.0.0.1 telemetry.sublimehq.com
	127.0.0.1 license.sublimehq.com
	127.0.0.1 45.55.255.55
	127.0.0.1 45.55.41.223
#. 关闭 Sublime text3，并打开安装目录，备份一下 sublime_text
#. 打开网址：https://hexed.it

点击上面的 Open File，弹出的对话框中选择已安装后的 sublime_text.exe。
Ctrl+F 搜索查找 ，输入97 94 0D，然后点击按钮 Search Now。
然后在97 94 0D上面点击，替换为00 00 00即可！
最后点击上面的 Export 按钮导出，将完成后的文件复制到刚才的目录替换掉原来的文件即可！
#. 运行 sublime_text文件，填入注册码：

----- BEGIN LICENSE -----
TwitterInc
200 User License
EA7E-890007
1D77F72E 390CDD93 4DCBA022 FAF60790
61AA12C0 A37081C5 D0316412 4584D136
94D7F7D4 95BC8C1C 527DA828 560BB037
D1EDDD8C AE7B379F 50C9D69D B35179EF
2FE898C4 8E4277A8 555CE714 E1FB0E43
D5D52613 C3D12E98 BC49967F 7652EED2
9D2D2E61 67610860 6D338B72 5CF95C69
E36B85CC 84991F19 7575D828 470A92AB
------ END LICENSE ------



**********************************
linuxmint下sublime text3无法输入中文的解决方法 
**********************************
首先保证你的电脑有c++编译环境
================
::

	sudo apt-get install build-essential
	sudo apt-get install libgtk2.0-dev

在～（/home）目录新建一个名为sublime-imfix.c的文件
===================================
::

	#include <gtk/gtkimcontext.h>
	 
	void gtk_im_context_set_client_window (GtkIMContext *context,
	 
	         GdkWindow    *window)
	 
	{
	 
	 GtkIMContextClass *klass;
	 
	 g_return_if_fail (GTK_IS_IM_CONTEXT (context));
	 
	 klass = GTK_IM_CONTEXT_GET_CLASS (context);
	 
	 if (klass->set_client_window)
	 
	   klass->set_client_window (context, window);
	 
	 g_object_set_data(G_OBJECT(context),"window",window);
	 
	 if(!GDK_IS_WINDOW (window))
	 
	   return;
	 
	 int width = gdk_window_get_width(window);
	 
	 int height = gdk_window_get_height(window);
	 
	 if(width != 0 && height !=0)
	 
	   gtk_im_context_focus_in(context);

	}

将上述文件编译成共享库libsublime-imfix.so
==============================
::

	gcc -shared -o libsublime-imfix.so sublime-imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC

将libsublime-imfix.so拷贝到sublime_text所在文件夹
========================================
::

	sudo mv libsublime-imfix.so /opt/sublime_text/

修改文件/usr/bin/subl的内容
====================
::

	sudo vi /usr/bin/subl

改成
::

	#!/bin/sh
	LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"

修改文件sublime_text.desktop的内容
===========================
[Desktop Entry]中的Exec项
----------------------
::

	Exec=/opt/sublime_text/sublime_text %F
改成
::

	Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"

[Desktop Action Window]中的Exec项
------------------------------
::

	Exec=/opt/sublime_text/sublime_text -n
改成
::

	Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"

[Desktop Action Document]中的Exec项
--------------------------------
::

	Exec=/opt/sublime_text/sublime_text --command new_file
改成
::

	Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"
