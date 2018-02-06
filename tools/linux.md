# dpkg

查找文件所属的安转包

    dpkg -S /usr/bin/ag

列出安装包所安装的所有文件

    dpkg -L silversearcher-ag

# redirection

only print error output

    command ... 2>&1 1>/dev/null

# Set up Solarized for Gnome Terminal

[Set up Solarized for Gnome Terminal](http://www.webupd8.org/2011/04/solarized-must-have-color-paletter-for.html)


# cscope
cscope -bkq -i cscope.files


# git

显示倒数第二个提交中该文件的内容
git show HAED^:drivers/usb/dwc_otg/dwc_otg_driver.c

显示倒数第二个提交中该文件的改变
git show HAED^ drivers/usb/dwc_otg/dwc_otg_driver.c

git diff HEAD

git diff HEAD HEAD^ --stat
drivers/video/rockchip/Kconfig  | 1 +
drivers/video/rockchip/Makefile | 1 +

git log --stat

# remount root fs as writeable
mount -o remount,rw rootfs /

# linux picture viewer software
eog

#sed replace string in files
sed -i 's/regexp/replacement/g' filename

${parameter:-word} 如果parameter没有设置或者为空，该表达式的值是word，word不赋值给parameter，否则是parameter的值
${parameter:=word} 如果parameter没有设置或者为空，该表达式的值是word，word  赋值给parameter，否则是parameter的值

gdb ' finish ' Continue running until just after function in the selected stack frame returns. 

# ffmpeg convert nv12 to png
ffmpeg -pix_fmt nv12 -s 1296x972 -i preview_image_3.yuv picture.png

# virtualbox share folder can't create symbol link
VBoxManage setextradata "VM name" VBoxInternal2/SharedFoldersEnableSymlinksCreate/*sharename* 1
