# 移动：
跳到行头：0 跳到行尾： $ 跳到行头有字符的地方：_

向右移动一个单词的距离(以空格结尾)：w 

向左移动一个单词的距离(以空格结尾)：b
跳到下一个指定字符x：f+x

跳到指定字符串abc：/abc

跳到某一行24：24G

使当前行置于页面中间：zz  置于底部： zb  置于顶部：  zt

跳转一页：ctrl+f  往上跳转一页：ctrl+b  往上跳转半页： ctrl+u

# 编辑：
行首插入：I  行尾插入：A

从当前光标删到行末：D

删除多行：d+num+j  删除多行同时进入编辑模式：c+num+j

删除一个单词同时进入编辑模式：cw

换行并且不进入编辑模式：ctrl+enter

将下一行和当前行合并，中间空格隔开：J

当光标位于单词中间想要删除整个单词时：diw(i理解为inside)  di(   //删除括号里的内容

# 选择：
整行选择：V

矩形块选择：ctrl+v

# 插件(适用于VScode搭载Vim插件的情况)
**easy motion插件**：
<leader><leader> s 字符  //跳转到指定字符

**surround插件**：
ds 符号  //删除成对的""或者[]等

ds 符号1 符号2  //替换成对的""或者[]等

# 相关链接
[bilibili: VS code × Vim](https://www.bilibili.com/video/BV1MX4y1b7nM/?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click&vd_source=19e15f25ad57f3a7f216153afc1963a8)