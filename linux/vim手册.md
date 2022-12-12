# vim 手册
{docsify-updated}

Mac 下可以使用 vimtutor 学习 vim。 直接在终端输入命令 vimtutor 即可。

1. 统计字符串出现的次数： `:%s/pattern//gn`
2. 命令模式下，开启行号显示：`set number`
3. 命令模式下，关闭行号显示：`set nonumber` ，
4. 删除全部内容: `ggdG`


使用linux下自带的vim工具，能快速的修改jar包内的内容。
1.vim springboot.jar
2.光标往下移动（可以通过输入/abc来搜索），找到要修改的文件 Enter进入
3.编辑后:wq保存退出，注意这里只是推出编辑的文件，到编辑jar里面，:q 退出jar编辑就可以了