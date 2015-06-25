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
