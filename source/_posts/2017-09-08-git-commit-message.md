---
title: commit message 规范 及自动化 changelog
subtitle: 使用过 git 来做版本控制的同学应该都知道每次提交修改 `git commit` 时，git 总会强制要求填写提交说明，否则不允许提交。填写的格式不限，内容也不限，当然为了提高项目的可维护性，commit message 做适当的格式要求是必要的，不说 review 起来方便，就冲看起来格式规整这一点，看起来都舒服得多了
date: 2017-09-08, 00:52:53 GMT+0800
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/17-09-07/git.jpg
categories: 日常学习
tags:
    - 学习
    - 规范
    - 自动化
    - git
author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

### 前言

------

文章假设读者了解并使用过 git ，不熟悉的同学请移步 [pro git][proGit]。

使用过 git 来做版本控制的同学应该都知道每次提交修改 `git commit` 时，git 总会强制要求填写提交说明，否则不允许提交。填写的格式不限，内容也不限，当然为了提高项目的可维护性，commit message 做适当的格式要求是必要的，不说 review 起来方便，就冲看起来格式规整这一点，看起来都舒服得多了。

例子：

![bad-git-commit][gitCommit]

![angular-commit][angularCommit]

话题转回 commit message 规范，社区总结了许多 [commit 规范][commitNorm]，其中包括：

[commit-norm-list][commitNormList]

笔者倾向于使用 angular 的 commit message 规范，更系统，本文也将使用该规范为例。

### commit message 格式

commit message 主要包括三个部分：Header、Body、Footer。

格式为：
```
<type>(<scope>): <subject>

<body>

<footer>
```

除 Header 外，Body、Footer均为非必填项。

#### Header

Header 要求单行，其中包括 `<type>`、`<scope>`、`<subject>`三个部分。

##### `type`

用来标识 commit 的类型，总共有以下 11 个标识：

* `feat`: 添加了一个新功能
* `fix`: 修复了一个 bug
* `docs`: 文档发生修改
* `style`: 不影响代码运行的更改（空格，格式，缺少分号等）
* `refactor`: 重构代码且不引进新的功能或修复 bug
* `perf`: 代码优化
* `test`: 添加或修改测试用例
* `build`: 构建工具或外部依赖的更改（npm，webpack，gulp等）
* `ci`: 更改项目级的配置文件或脚本
* `chore`: 除上述之外的修改
* `revert`: 撤销改动先前的提交

> 特别注意：使用 `revert` 标识撤销 commit 时，`subject` 应为所撤销的 commit 的 message， Body 应包含 所撤销的 commit 的 hash。

格式如下：

```
revert: fix: change aa to bb
    
This reverts commit ca1b58f63fcaa2ae763a5604e5b46e802d83105c.
```

##### `scope`

用来标识改动所影响的范围，视项目而定。

##### `subject`

改动的简短描述，不超过 50 字符长度。

#### Body

本次 commit 的详细描述。

#### Footer

主要用于两种情况：

* 重大的不兼容改动: 用于给出改动说明及解决方案。
* 关联 issues: 用于关闭相应 issues。

### 高效使用

------

上边介绍了如何按规范写 commit message，那么下面教你如何利用工具更高效的使用这套规范。

首先我们来介绍几个工具包:

#### [commitizen][cz-cli]

commitizen 是用来快速生成指定的 commit message 规范的工具包，具体使用方式请参照 [官方文档][cz-cli] 或参照下文给出的 demo。

> 注： 官方推荐全局安装，然后使用全局 `git-cz` 命令来快速生成 commit message，但是由于笔者极不喜欢污染全局环境，所以以下例子中使用局部安装的方式来使用该包

安装相应依赖包：
``` shell
npm install --save-dev commitizen cz-conventional-changelog
```

添加如下启动脚本至项目根目录, 文件名假定为 `git-cz.js` ：

``` js
const path = require('path');
const bootstrap = require('commitizen/dist/cli/git-cz').bootstrap;

bootstrap({
    cliPath: path.join(__dirname, './node_modules/commitizen'),
    config: {
        path: path.join(__dirname, './node_modules/cz-conventional-changelog')
    }
});
```

添加 npm 脚本命令进 package.json 中:
``` js
{
    ...
    "srcipt": {
        ...
        "git-cz": "node run git-cz.js"
        ...
    } 
    ...
}
```

ok，做到这一步大功告成，使用 `npm run git-cz` 即可启动该工具包，但是：笔者认为键入 npm 命令启动总觉得于 git 分格格格不入，因此有了下面的另一步：

使用 git 别名来启动 npm 命令（关于 git 别名请参考 [pro git][pro-git-alias]）:

``` shell
git config alias.cz '!npm run git-cz'
```

然后就可以愉快的使用 `git cz` 快速生成符合规范的 commit message了。

#### [validate-commit-msg][validate-commit-msg]

用于检查 当前项目的 commit message 是否符合格式，下例中使用 [ghooks][ghooks] 配合使用

使用 npm 安装:

``` shell
npm install --save-dev validate-commit-msg ghooks
```

在项目根目录中新建 `.vcmrc` 配置文件（具体参数作用请参照该项目 REAMDE），文件内容如下：

``` js
{
    "types": [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "perf",
        "test",
        "build",
        "ci",
        "chore",
        "revert"
    ],
    "scope": {
        "required": false,
        "allowed": ["*"],
        "validate": false,
        "multiple": false
    },
    "warnOnFail": false,
    "maxSubjectLength": 100,
    "subjectPattern": ".+",
    "subjectPatternErrorMsg": "subject does not match subject pattern!",
    "helpMessage": "",
    "autoFix": false
}
```

将以下配置写入 package.json 文件的环境配置中:

``` js
{
    ...
    "config": {
        ...
        "ghooks": {
            "commit-msg": "validate-commit-msg"
        }
        ...
    }
    ...
}
```

ok，配置完成后，该项目每次 commit 时，都将启动该脚本进行检测，如 commit message 格式不正确，本次 commit 失败。

> 注: 有兴趣的同学可以去阅读 ghooks 源码，以及了解什么是 git hooks。

#### [standard-version][standard-version]

用于依据 commit message 生成 changelog 以及版本发布。

*注：*默认情况下 changelog 只根据 type 为 `feat` 和 `fix` 类型的 commit message 来生成。

使用 npm 安装:

``` shell
npm install --save-dev standard-version
```

添加 npm 脚本命令进 package.json 中:
``` js
{
    ...
    "srcipt": {
        ...
        "release": "standard-version"
        ...
    } 
    ...
}
```

运行 `npm run release` 即可快速生成 changelog 以及生成相应版本 tag，具体使用方式请查阅该项目介绍。



综合上述关于 commit message 的规范以及日常开发中对 git 的使用，笔者写了一个小 [demo][demo]，具体内容如下：

* 使用 `git cz` 快速生成符合规范的 commit message
* 使用 git hooks commit-msg 配合 `validate-commit-msg` 校验 commit message 格式，保证格式统一
* 使用 git hooks post-merge 及相应脚本对合并到 master 分支操作进行绑定，当从其它分支合并到 master 分支时，自动生成 changelog 及相应版本的tag

甚至可以利用 git hooks 做更多自动化的工作，有兴趣的同学可以去尝试下。


### 参考文献

------

* [Commit message 和 Change log 编写指南][Commit message 和 Change log 编写指南]
* [［译］AngularJS Git提交信息规范][［译］AngularJS Git提交信息规范]
* [Pro Git book][proGit]

[proGit]: https://git-scm.com/book/zh/v2
[gitCommit]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-09-07/git-commit-1.png
[angularCommit]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-09-07/angular-commit.png
[commitNorm]: https://github.com/conventional-changelog/conventional-changelog
[commitNormList]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-09-07/commit-norm-list.png
[cz-cli]: https://github.com/commitizen/cz-cli#making-your-repo-commitizen-friendly
[pro-git-alias]: https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-Git-%E5%88%AB%E5%90%8D
[validate-commit-msg]: https://github.com/conventional-changelog/validate-commit-msg
[ghooks]: https://github.com/ghooks-org/ghooks
[standard-version]: https://github.com/conventional-changelog/standard-version
[demo]: https://github.com/MinFE/git-commit-lint
[Commit message 和 Change log 编写指南]: http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html
[［译］AngularJS Git提交信息规范]: https://segmentfault.com/a/1190000004282514