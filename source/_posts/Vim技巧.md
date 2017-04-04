---
title: Vim技巧
date: 2017-04-03 10:24:36
tags:
---

## 1 命令快捷键总结
1. Ctrl+z 回到shell，在shell中输入fg回车回到vim
2. 要同时在多行相同位置插入复制的内容，首先 Ctrl + v 选择位置，然后输入 大写i 进入编辑模式，然后 shift+insert 粘贴，最后Esc退出visual block模式，插入成功
3. /xxx 查找xxx，按n/N 跳转到下一个/上一个匹配位置
4. :vsp xxx 垂直分割窗口打开xxx
	:sp xxx 水平分割窗口
5. :bn 和 :bp 切换上一个/下一个文件编辑窗口（或者Ctrl+ww）
6. \# 匹配上一个当前单词
   \* 匹配下一个当前单词
7. :1, 5 copy 20 将1至5行内容复制到20行位置
8. :d5 剪切从当前行开始的5行，用p粘贴到指定行
9. dw 剪切当前字符开始的一个词，使用 p 粘贴
10. cw 删除当前字符开始的单词，并进入插入模式（相当于替换当前单词内容）
11. 0 移动到本行最开头
	$ 移动到本行最尾部
	w 移动到下一个单词起始字符处
	e 移动到下一个单词结尾字符处
12. % 匹配括号进行移动，可以从 ‘{‘ 移动到对应的 ‘}’，或者反之（首先得保证当前光标在某个半括号上，适用于大中小括号）
13. u undo
14. Ctrl+r redo
15. Ctrl+p/n 自动补全，p表示向上查找匹配词，n表示向下查找；
16. :s/xxx/yyy/ 将当前行中的第一个「xxx」替换为「yyy」
	:s/xxx/yyy/g 将当前行中的所有「xxx」替换为「yyy」
	:s/xxx/yyy/gc 同上，但每次替换都会询问
	:%s/xxx/yyy/g 将整个文件中的所有「xxx」替换为「yyy」
17. Ctrl+o 回到上一次编辑处
	Ctrl+i 回到下一次编辑处
18. :x 等价于 :wq 等价于 ZZ 保存并退出
19. vim -d file1 file2 比较两个文件不同之处
20. 光标位置移动：
	a. 保持当前光标位置仅滚动屏幕内容：Ctrl+e/y
	b. 移动光标到上/下一个历史位置：Ctrl+o/i
21. visual、visual行/列模式：
	a. visual模式：normal模式下按v进入该模式，移动光标可以选中光标首尾的内容；
	b. visual line模式：即行模式，normal模式下按大写V（或shift+v）进入该模式，移动光标可以选中对应的行；
	c. visual block模式：即列模式，normal模式下按Ctrl+v进入该模式，移动光标可以选中对应列的内容；

## 2 ctags/taglist配置使用
**用途：**  
ctags是一个工具软件，可以为大型项目的源代码生成标签，便于阅读源码时进行跨文件跳转，就像在windows中使用source insight一样，非常方便。

**配置：**  
1. 生成标签文件
在项目根目录下执行如下命令：  
$ctags -R .  
其中-R表示递归地为当前目录及子目录中所有的源码文件生成标签文件tags，该文件会保存在当前目录下。  

2. 使用tag进行跳转
1) 用vim打开一个源码文件，随便找到一个变量或者函数调用位置  
2) 按Ctrl+]可以跳转到该变量/函数定义的位置(可能位于其他文件)，如果有多个文件都包含同名的定义，那么它会以列表形式显示，让你输入编号进行选择  
3) 按Ctrl+t跳回之前的位置  
注意：此时打开vim必须在tags文件所在目录，否则会提示找不到tags文件，对于大项目，你可能需要在任意目录下查看源码，这就需要在.vimrc文件中增加一行：  
set tags=tags;/   
该命令指示vim在当前目录找不到ctags文件时，自动去上层目录查找。  

3. taglist插件安装
taglist是一个vim插件，可以显示源码文件的梗概，如变量、方法列表。安装方法是到www.vim.org中搜索该plugin，然后下载、解压，它会自动解压到 ~/.vim/plugin 中，如果该目录不存在请手动创建。  
安装完成之后，利用vim打开一个源码文件，然后进入 :命令模式，输入：  
```
:Tlist  
```

即可打开taglist窗口，按F1可以看到操作帮助，为了方便，也可以自己设置打开/关闭taglist的快捷键，比如用F8作为快捷键，则需在.vimrc中添加：  
```
nnoremap <silent> <F8> :TlistToggle<CR>
```
这样就可以利用F8打开/关闭taglist了。  
ctags显示跳转列表需要在vimrc文件中添加一行：  
```
set cscopetag
```

## 3 .vimrc配置(含快捷键)
参考[文章1](http://www.360doc.com/content/13/0111/13/168576_259534618.shtml)
参考[文章2](http://www.cnblogs.com/moodlxs/archive/2012/03/24/2415526.html)
参考[文章3](http://vim.sourceforge.net/scripts/script.php?script_id=2506)
参考[文章4](https://zybuluo.com/uuprince/note/81709)

