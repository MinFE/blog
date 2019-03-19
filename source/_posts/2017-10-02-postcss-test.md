---
title: postcss 小试
subtitle: css 后处理器的小小尝试
date: 2017-10-02
cover: https://blog.static.minfive.com/post/17-10-02/postcss.jpg
categories: 日常学习
tags:
    - postcss
author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

### What is postcss

* PostCSS 为基于 JavaScript 开发的 css 后处理器
* PostCSS 提供了一个解析器，它能够将 CSS 解析成抽象语法树（AST）
* PostCSS 的实际强悍之处为插件系统，依赖插件系统可以对 CSS 做任意处理

![图片][image]

### What can it do

* 对 CSS 文件做任意后期处理（即 PostCss 称为 css 后处理器的原因），例如：autoprefixer
* 扩展 css 语法 例如：cssnext

### how to use

[doc.usage][doc.usage]

### webpack配置

``` javascript
use: [
	'css-loader',
	{
		loader: 'postcss-loader',
		options: {
			plugins: function() {
				return [
			    	require('postcss-import'),
			    	require('postcss-css-variables')({ preserve: false }),
					require('postcss-cssnext')({
    					browsers: [
    						"> 1%",
    					    "last 2 versions"
    					]
    				})
				]
			}
		}
	}
]
```

## Features CSS

### 未来语法

* 嵌套
* 自定义属性
* ...

基于 PostCSS 可以将未来语法转换为现阶段的语法。

[features][features]

### note

* PostCss 的处理顺序依赖插件的顺序，错误的顺序可能带来意想不到的编译结果
* PostCss 总体还不大成熟，对于自定义属性仅在 `:root` 下的支持比较完美，局部作用域总会出现意想不到的问题


[image]: http://oo12ugek5.bkt.clouddn.com/postcss-demo/239162490-562dd5c1849a6_articlex.png
[doc.usage]: https://github.com/postcss/postcss#usage
[features]: http://cssnext.io/features/