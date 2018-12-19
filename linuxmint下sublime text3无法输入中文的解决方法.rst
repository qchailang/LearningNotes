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
