---
title: es6引入css
date: 2019-12-24 22:26:40
tags: JavaScript
---

假设我们有一个名为`style.css`的css文件

```css
.default {
    cursor:null;
}

.drag {
    cursor:move;
}

.rotate {
    cursor: url('../../assets/mouse/closedhand.cur') 8 8, default;
}
```

在其它js文件中如何引入并使用这个css文件呢，总不能一直只使用js来编写吧

假设我们在`EventManager.js`文件中要使用，有两种方式引入：

```javascript
import './style/style.css';
```

```javascript
require('./style/style.css');
```

然后在代码中就可以直接使用了，如：

```javascript
...
this._dom.className='rotate';
...
this._dom.className='drag';
```

此时，css的效果就出来了。

