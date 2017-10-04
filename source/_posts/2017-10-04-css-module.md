---
title: css modules 小试
subtitle: css modules 顾名思义，即 CSS 的模块化。使得 CSS 也能适用软件工程方法。
date: 2017-10-04
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/post/17-10-04/css-modules.png
categories: 日常学习
tags:
    - css
    - css modules
author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

### 定义

一个css模块即一个定义好了所有样式（类）和动画名称的本地css文件

> 官方推荐仅使用类来定义样式

CSS Modules 会编译成一种低层级的ICSS，但它的格式与正常css格式相似

``` css
/* style.css */
.className {
    color: green;
}
```

当使用js模块导入css模块时，它输出一个属性与本地样式名称相对应的对象

``` javascript
import styles from './style.css';

element.innerHTML = '<div class="' + styles.className + '">';
```

### 命名

---

对于本地类名建议使用驼峰命名，但并非强制

> 关于使用驼峰命名的方式主要是为了更好在js中导入并使用css模块

可以为css-loader增加camelCase参数来实现自动转换

``` javascript
{
    test: /\.css$/,
    loader: 'style!css?modules&camelCase',
}
```

### 作用域

`:global`: 切换到当前选择器所在全局作用域下
`:local`: 切换到局部作用域下

如果切换到全局模式下，定义的样式将允许在全局作用域中使用

Example:
``` css
.localA :global .global-b .global-c :local(.localD.localE) .global-d
```


### 组合

用于组合其它选择器

``` css
.className {
    color: green;
    background: red;
}

.otherClassName {
    composes: className;
    color: yellow;
}
```

允许拥有多个组成规则，但组成规则必须先于其它规则。当一个类composes另外一个类时，css模块对外的接口为当前类名，允许添加多个类名。

组成规则允许使用多个类：`composes: classNameA classNameB`

### 依赖

允许compose其它CSS Modules的类名

``` css
.otherClassName {
    composes: className from './style.css';
}
```

注意：
1. 当从不同的文件composes多个类时，compose的顺序是不确定的，因此，需要确保composes的类没有定义相同的属性值
2. compose不应该循环嵌套，因为Elsewise是无法确定这是组成规则还是属性，模块系统将会发出一个错误。
3. 最好的方式就是基本类与依赖分离

### 为什么使用CSS Modules

模块化和可重复使用的css
+ 解决命名冲突
+ 显式依赖
+ 没有全局作用域

### 编译结果
默认为哈希字符串
![image](https://sfault-image.b0.upaiyun.com/127/065/1270651827-571acfceca712)

允许自定义配置
![image](https://sfault-image.b0.upaiyun.com/431/079/431079432-571acff0052ea)

