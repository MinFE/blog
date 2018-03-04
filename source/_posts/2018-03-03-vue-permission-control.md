---
title: 关于 vue 的路由权限管理
subtitle: 曾经在工作上对 vue 路由权限管理这方面有过研究，这几天又看到了几篇相关的文章，再加上昨天电面中又再一次提及到，就索性整理了一下自己的一些看法，希望对大家有帮助。
date: 2018-03-03
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/18-03-03/vue-router.jpg
categories: 经验总结
tags:
    - javascript
    - vue
    - 权限管理

author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---


### 前言

曾经在工作上对 vue 路由权限管理这方面有过研究，这几天又看到了几篇相关的文章，再加上昨天电面中又再一次提及到，就索性整理了一下自己的一些看法，希望对大家有帮助。

### 实现

大体上实现的思路很简单，先上图：

![permission-control][permission-control]

无非是将路由配置按用户类型分割为 `用户路由` 和 `基本路由`，不同的用户类型可能存在不同的 `用户路由`，具体依赖实际业务。

* `用户路由`: 当前用户所特有的路由
* `基本路由`：所有用户均可以访问的路由

实现控制的方式分两种：

1. 通过 [vue-router addRoutes][vue-router-methods] 方法注入路由实现控制
2. 通过 [vue-router beforeEach][vue-router-methods] 钩子限制路由跳转

* addRoutes 方式：

> 通过请求服务端获取当前用户路由配置，编码为 vue-router 所支持的基本格式（具体如何编码取决于前后端协商好的数据格式），通过调用 `this.$router.addRoutes` 方法将编码好的用户路由注入到现有的 vue-router 实例中去，以实现用户路由。

* beforeEach 方式

> 通过请求服务端获取当前用户路由配置，通过注册 `router.beforeEach` 钩子对路由的每次跳转进行管理，每次跳转都进行检查，如果目标路由不存再于 `基本路由` 和 当前用户的 `用户路由` 中，取消跳转，转为跳转错误页。

**以上两种方式均需要在 `vue-router` 中配置错误页，以保证用户感知权限不足。**

两种方式的原理其实都是一样的，只不过 `addRoutes 方式` 通过注入路由配置告诉 `vue-router` ：“当前我们就只有这些路由，其它路由地址我们一概不认”，而 `beforeEach` 则更多的是依赖我们手动去帮 `vue-router` 辨别什么页面可以去，什么页面不可以去。说白了也就是 `自动` 与 `手动` 的差别。说到这，估计大家都会觉得既然是 `自动` 的，那肯定是 `addRoutes` 最方便快捷了，还能简化业务代码，笔者一开始也是这么认为的，但是！很多人都忽略了一点：

![vue-router-addRoutes][vue-router-addRoutes]

**`addRoutes` 方法仅仅是帮你注入新的路由，并没有帮你剔除其它路由！**

设想存在这么一种情况：用户在自己电脑上登录了管理员账号，这个时候会向路由中注入**管理员的路由**，然后再退出登录，保持页面不刷新，改用普通用户账号进行登录，这个时候又会向路由中注入**普通用户的路由**，那么，在路由中将存在两种用户类型的路由，即使用户不感知，通过改变 url，普通用户也可以访问管理员的页面！

对于这个问题，也有一个解决办法：

``` javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

const createRouter = () => new Router({
  mode: 'history',
  routes: []
})

const router = createRouter()

export function resetRouter () {
  const newRouter = createRouter()
  router.matcher = newRouter.matcher
}

export default router
```

通过新建一个全新的 `Router`，然后将新的 `Router.matcher` 赋给当前页面的管理 `Router`，以达到更新路由配置的目的。

笔者做了一个[小demo][demo]，大家可以去[体验一下][experience]。

关于上述问题，在 [vue-router][vue-router-source] 的 github issues 下有过讨论，分别是：

* [Add option to Reset/Delete Routes #1436][#1436]
* [Feature request: replace routes dynamically #1234][#1234]

有兴趣的可以去围观下。


[permission-control]: http://oo12ugek5.bkt.clouddn.com/blog/images/18-03-03/route-permission-control.png
[vue-router-methods]: https://router.vuejs.org/zh-cn/api/router-instance.html#methods
[vue-router-addRoutes]: http://oo12ugek5.bkt.clouddn.com/blog/images/18-03-03/vue-router-addRoutes.png
[vue-router-source]: https://github.com/vuejs/vue-router
[demo]: https://github.com/MinFE/vue-router-premission-control-demo
[experience]: https://minfe.github.io/vue-router-premission-control-demo/
[#1436]: https://github.com/vuejs/vue-router/issues/1436
[#1234]: https://github.com/vuejs/vue-router/issues/1234