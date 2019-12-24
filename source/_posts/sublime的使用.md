---
title: sublime的使用
date: 2019-12-24 22:28:21
tags: sublime
---

#### sublime插件的安装使用

[参考](<https://blog.csdn.net/wxl1555/article/details/69941451/>)

##### 1.直接安装

安装Sublime text 3插件很方便，可以直接下载安装包解压缩到Packages目录（菜单->preferences->Browse Packages）。

##### 2.使用Package Control组件安装

也可以安装package control组件，然后直接在线安装：

按Ctrl+ `(此符号为tab按键上面的按键) 调出console（注：避免热键冲突）
粘贴以下代码到命令行并回车：

```javascript
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

按回车键，会看到下面出现东西在左右摆动，说明正在下载。

##### 3. 下载完成之后重启Sublime Text 3。

##### 4. 如果在Perferences->中看到package control这一项，则安装成功。

##### 5.用Package Control安装插件的方法：

* 按下Ctrl+Shift+P调出命令面板
* 输入install 调出 Install Package 选项并回车，
* 然后在列表中选中要安装的插件。



其它：

##### JSFormat  JS代码格式化插件。

使用方法：使用快捷键ctrl+alt+f