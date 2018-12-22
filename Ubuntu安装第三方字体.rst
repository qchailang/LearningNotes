*******************
Ubuntu/Mint 安装第三方字体
*******************
#. sudo mkdir -p /usr/share/fonts/winFonts
#. 将第三方字体解压到此目录
#. 改变权限
 | sudo chmod 644 /usr/share/fonts/winFonts/*
4. 开始安装
 | cd /usr/share/fonts/winFonts/
 | sudo mkfontscale **//创建字体的fonts.scale文件，它用来控制字体旋转缩放**
 | sudo mkfontdir **//创建字体的fonts.dir文件，它用来控制字体粗斜体产生**
 | sudo fc-cache -fv **//建立字体缓存信息， 让系统认识新安装的字体**