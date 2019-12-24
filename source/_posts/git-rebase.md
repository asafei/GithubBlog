---
title: git rebase
date: 2019-12-24 22:35:01
tags: git
---
## git rebase命令

#### 一、rebase的作用

先看例子，假设我们的操作如下：

```java
1、master分支已有文件master.txt；
2、从master拉取分支B，并创建文件b1.txt,add-commit;
3、从master拉取分支C，并创建文件c1.txt,add-commit;
4、切换分支B，并创建文件b2.txt,add-commit;
5、切换分支C，并创建文件c2.txt,add-commit;

6、在分支C，合并master分支，然后将结果合并到master；
7、在分支B，合并master分支，然后将结果合并到master；
```

此时结果怎么样呢？会发现分支B和分支C的提交，相互交叉，按照DATE排序的，如下：

```shell
commit 17efaf6eeb99ecf9e6af5e78781279ae2e88af86 (HEAD -> master, B)
Merge: e791fb5 4d25aef
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 12:06:34 2019 +0800

    Merge branch 'master' into B

commit 4d25aef5fdadd54bf68ab6a6cd0f83ade7a46fe7 (C)
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:56:24 2019 +0800

    [add]C添加文档2

commit e791fb50580a2eaecac426df07bec10fb3d59ddc
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:55:43 2019 +0800

    [add]B添加文件2

commit 1567f12c0739fa898a89348422038507ab2cfd01
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:54:44 2019 +0800

    [add]C添加文件1

commit cecc88eca0f9b8d37d2f92fd99250cbb5b0da7e7
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:53:38 2019 +0800

    [add]B添加文件1

commit 8207c1f12cc1430208e2273d9429c04675afddbc
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:52:44 2019 +0800

    [add]master添加文件1

```

如果我们不想让B和C的commit交叉，该怎么做呢，就可以使用rebase命令了，只需要修改第6、7步即可

~~7、在分支B，合并master分支，然后将结果合并到master；~~

```java
7、在分支B， rebase master分支，然后将结果合并到master；
```

此时我们再看一下log，就会返现日志是将C的commit完成之后，然后再有commitB中的提交

```she
commit 552d2ba40803fc9ab013b6571efc2cda58f71236 (HEAD -> master, B)
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:55:43 2019 +0800

    [add]B添加文件2

commit 07686a4be8a2ba5fb94810f1f23fdf96478a4193
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:53:38 2019 +0800

    [add]B添加文件1

commit 4d25aef5fdadd54bf68ab6a6cd0f83ade7a46fe7 (C)
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:56:24 2019 +0800

    [add]C添加文档2

commit 1567f12c0739fa898a89348422038507ab2cfd01
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:54:44 2019 +0800

    [add]C添加文件1

commit 8207c1f12cc1430208e2273d9429c04675afddbc
Author: yahui.yang <yahui.yang@ecarx.com.cn>
Date:   Thu Oct 10 11:52:44 2019 +0800

    [add]master添加文件1
```

什么原因呢？rebase命令会把分支B中的commit取消掉，将它们保存为补丁（patch），然后将分支B更新为master中最新的内容，最后将补丁应用到分支B中。

#### 二、冲突的解决

在merge过程中如果出现的冲突，一般需要add并且commit进行处理。

但是在rebase过程中若出现冲突，解决完冲突后并不需要add和commit，只需要执行

`git  rebase --continue`即可，

#### 三、rebase的取消

在任何时候都可以使用命令`git rebase --abort`来终止rebase的动作，并且分支会回到rebase之前的状态

参考：<http://gitbook.liuhui998.com/4_2.html>