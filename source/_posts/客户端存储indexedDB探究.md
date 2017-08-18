---
title: 客户端存储indexedDB探究
subtitle: Indexed Database API,是在浏览器中保存结构化数据的一种数据库。
date: 2016-05-15
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/dbdm-large.png
categories: 日常学习
tags:
    - 学习
    - javascript
author:
  nick: minfive
  github: 'https://github.com/Mrminfive'
---

### 前言

趁着最近清闲，把先前落下的红宝书下半部分给看完了，最感冒的是客户端存储，WebSocket以及canvas（WebGL），WebSocket这个我就不说了，剩下的一个是我认为将来会在移动应用上大显身手的东西，一个则是前端可视化大趋势下必不可少的东西，canvas（WebGL）的话因为暂时还没入坑，所以先不讲，以后研究完了会补上来，这篇文章就主讲客户端存储。

> 文章基于《javascript高级程序设计》23章以及MDN技术文档加上个人的一些见解，结构主体是技术文档，穿插一些个人踩坑的总结，如见解有误，请指出，谢谢！

------

### 简介

`IndexedDB`本片文章的主人公，在红包书中的解释是:”Indexed Database API,是在浏览器中保存结构化数据的一种`数据库`“，对的！你没看错，就是数据库，而且是一种类SQL的结构型数据库，最大的特色就是用对象来存储数据，容量以及数据存储查询速度远比storage（local/session）好的多，但缺点就是支持程度还不是很好。

### 使用

由于目前兼容性的问题，所以使用前需要对浏览器提供的API进行兼容：

``` js
window.indexedDB = window.indexedDB || window.mozIndexedDB || window.webkitIndexedDB || window.msIndexedDB;
window.IDBTransaction = window.IDBTransaction || window.webkitIDBTransaction || window.msIDBTransaction;
window.IDBKeyRange = window.IDBKeyRange || window.webkitIDBKeyRange || window.msIDBKeyRange;
```

#### 基本使用模式

1. 打开数据库并且开始一个事务。
2. 创建一个 object store。
3. 构建一个请求来执行一些数据库操作，像增加或提取数据等。
4. 通过监听正确类型的 DOM 事件以等待操作完成。
5. 在操作结果上进行一些操作（可以在 request 对象中找到）

> 记住一点，任何对indexedDB数据库的操作都是异步进行的，任何操作都会有两个必定的事件权柄：`success`和`error`，以下的操作均要事情设置相应的监视事件防止出现错误，由于个人比较懒，在这里展示就不加上了

#### 打开数据库

和mysql/sql数据库的使用一样，indexedDB的使用依旧是从打开数据库开始的，打开方式如下：

``` js
var request = indexedDB.open(DBname, version);
// DBname：数据库名； version：要打开的数据库版本号（可不传值，默认为最新版本）
```

> 版本号只能使用int类型，而不能使用浮点数，不然会导致错误

与其它操作不同，open操作特有两个权柄：`upgradeneeded`（版本更新）和`blocked`（没有关闭连接尝试连接，（不常用））;

依据基本使用模式，正确的打开姿势应为：

``` js
var request = indexedDB.open(DBname, version);

request.onerror = function() {};
request.onsuccess = funciton() {};
request.onupgradeneeded = function() {};
request.onblocked = function() {};

// 打开的数据库为request触发的成功事件中的event.target.result;

// 以后的操作参照这种方式创建监视事件
```

> 注：这里有一个坑，upgradeneeded（版本改动），故名思意是版本改动才会触发的事件，在MDN上的解释是“在数据库第一次被打开时或者当指定的版本号高于当前被持久化的数据库的版本号时，这个版本改动事务将被创建。”，而`任何对数据库结构产生改变的操作都应该在这个事件内执行`，至于怎么改变数据库结构，请看下文。

#### 删除数据库

``` js
indexedDB.deleteDatabase(DBname);
```

#### 对象存储空间

关于对象存储空间，咱们可以把它看成是数据库的一个表，存储空间（表）的使用如下：

##### 打开存储空间

``` js
var store = db.objectStore(storeName);
```

##### 创建存储空间

``` js
var store = db.createObjectStore(storeName, {keyPath: keyName， autoIncrement: true});
// 创建存储空间
```

创建存储空间必须设置keyPath（可以把它看成是存储空间的主键），createObjectStore方法的第二个参数对象就是对keyPath键的配置，配置对象中除了指定键为keyPath，还有另外一个属性`autoIncrement`,这个属性在MDN文档上的解释是

![createStore][1]

通俗化讲就是`autoIncrement`属性是用来设置在被指定为keyPath的键为空时，是否自动生成keyPath值（这里有一点：不启用自动填补keyPath值时，因为keyPath是必须的，如果传入的数据中被指定为keyPath的键为空时，会导致创建存储空间失败，触发error事件）

##### 创建索引

什么是索引：

当用户想要通过其它键去访问查询存储空间中的数据时，索引就派上用场了，索引可以看成是存储空间的一个副本，只不过把主键替换成你想要的键，但它并不真实存在，只是给存储空间的访问查询提供了便利。

创建方式：

``` js
var store = db.createObjectStore(storeName, {keyPath: keyName， autoIncrement: true}；
store.createIndex(indexName, keyName, { unique: false});
// createIndex有三个参数，第一个为索引名，第二个为设置索引的键，第三个为配置对象，unique表示是否允许重复，是否允许重复要根据存储空间的数据而定
```

##### 填充数据

向存储空间中填充数据有两个方法：add方法和put方法，两个方法都是向存储空间中填充数据，但add相当于插入新值，当存储空间中已有相同数据（即keyPath值相同）时会返回错误，put相当于更新原有的值，与add方法相反，put方法会更新已有的数据。

使用方式为：

``` js
store.add(obj);
store.put(obj);
// obj为格式正确的数据对象
```

##### 修改存储空间结构

对于创建索引和设置keyPath的操作均视为改变数据库结构的操作，因此这些操作必须在upgradeneeded事件下进行，那么问题来了，如果你想重新配置数据库，那么应该怎么去触发这个事件呢？

解决方法有很多种，在这里介绍两种比较实用的

``` js
// 第一种，关闭现有数据库连接，打开新版本号的数据库
var request = indexedDB.open(dbName, version),
    db;
request.onsuccess = function(event) {
    db = event.target.result;
}

// 这里设置延时，因为数据库操作是异步进行的
setTimeOut(function() {
    db.close();
    
    request = indexedDB.open(dbName, version + 1);
    request.onsuccess = function(event) {
        db = event.target.result;
    }
    requese.onupgradeneeded(event) {
        // 这里是更改数据库结构的代码
    };
    
}, 200);


// 第二种，与第一种原理相同，只不过换了另外一种方式
var request = indexedDB.open(dbName, version),
    db;
request.onsuccess = function(event) {
    db = event.target.result;
}

setTimeOut(function() {
    var deleteDbRequest = db.deleteDatabase(dbName);
    deleteDbRequest.onsuccess = function (event) {
        var openRequest = localDatabase.indexedDB.open(dbName,1);

        openRequest.onsuccess = function(event) {
            db = openRequest.result;
        };  
        openRequest.onupgradeneeded = function (evt) {
            // 这里是更改数据库结构的代码
        };
    }
}
```

### 事务

在红宝书中的解释是“跨过创建对象存储空间这一步之后，接下来的所有操作都是要通过事务来完成，任何时候，只要想读取或修改数据，都要通过事务来组织所有操作”，简单点说就是事务就是一个工具，通过这个工具，你才能任意的去数据库中获取你想要的东西。

> 这里需要注意一下，事务接收的DOM事件与其它操作有点不同，它接收三个事件：error（失败），abort（中止），complete（完成），至于传递额事件对象及其使用方法请参考[MDN文档][2]

事务的所有方法及属性如下：

![transaction][3]

#### 创建方法

在数据库中调用transaction（）方法进行创建事务，该方法可指定两个参数，第一个参数是事务可操作的存储空间，该参数可为数组，传递多个存储空间，第二个参数是事务的访问模式（也就是访问权限），模式有三种：读写（readwrite）、只读（readonly）、版本改变（versionchange），不设置访问模式的话默认是readonly。

``` js
// 不带任何参数，表示创建一个能访问数据库中所有存储空间的事务，但这个事务只有读取的权限
var transaction = db.transaction();

// 指定一个存储空间
var transaction = db.transaction(storeName);

// 指定多个存储空间
var transaction = db.transaction([storeName1, storeName2]);

// 指定一个存储空间并设置读写权限
var transaction = db.transaction(storeName, "readwrite");
```

#### 使用方法

在创建了事务后，可以使用objectStore（）方法传入存储空间的名称，获取存储空间对象，这个基于事务创建的存储空间对象会比createObjectStroe（）方法创建的存储空间对象多一些可使用的方法：get[取值]、delete[删除指定数据对象]、clear[清空存储空间中所有数据]等常用方法外，还有其它方法，所有方法请见下图：

![store][4]

> 这部分只展示一些常用的方法，其余方法请自行参考[文档][2]

``` js
var store = db.transaction(tableName, "readwrite").objectStore("tableName"),
    request;
// 获取指定keyPath值的数据
request = store.get(keyPathValue);

request.onsuccess = function(event) {
    console.log("数据为" + JSON.stringify(event.target.result));
};

// 删除指定数据
request = store.delete(keyPathValue);

request.onsuccess = function() {
    console.log("删除成功");
};

// 清空存储空间
request = store.clear();

request.onsuccess = function() {
    console.log("清空了存储空间");
};
```

### 游标

上边说的所有操作，基本上和普通的数据库没有太大差别，而游标却是indexedDB最具特色的一个东西，与键范围结合使用将大大加快数据检索速度，同时操作也将更为方便。
游标的话可以看成是一个指针，指向存储空间的某个位置，每个位置上均有一个数据对象，然后可以对数据对象进行操作，游标可以按你意愿进行移动，访问你想访问的数据。
键范围的话则是在存储空间中再划分出来一个范围进行检索，缩小了检索的范围。

#### 创建方式

使用openCursor（）方法进行创建，该方法接收两个参数，第一个参数为键范围实例，第二个参数为游标的移动方向。两个参数均有默认值。

移动方向有4个设置常量：

> 1. IDBCursor.NEXT(0)：下一项，为默认值。
> 2. IDBCursor.NEXT_NO_DUPLICATE(1)：下一个不重复的项。
> 3. IDBCursor.PREV(2)：前一项。
> 4. IDBCursor.PREV_NO_DUPLICATE：前一个不重复的项。

#### 游标自身（IDBCursor实例）

IDBCursor实例有4个属性，分别是：

1. direction：数值，表示游标移动的方向。
2. key：对象的键。
3. value：实际的数据对象。
4. primaryKey：游标使用的键（即当前存储空间或索引中设置的主键）。

> 以上4个属性均为只读

另外，IDBCursor实例还有几个常用方法：

1. update（dataObj）：更新当前游标所对应的数据对象值，dataObj为传递的新数据。
2. delete（）：删除当前游标所对应的数据。
3. continue（key）：移动游标到结果集中的下一项。参数key是可选的，不指定这个参数，游标移动到下一项；指定这个参数，游标会移动到指定键的位置。
4. advance（count）：向前移动count指定个项数。

> 这里有一点要注意，每一次移动游标成功后，触发的success事件的event.target.result为IDBCursor实例自己，但当移动方向在结果集中再无下一项，仍然移动光标的话，event.target.result的值将为undefined。

使用方式：

``` js
var store = db.transaction(tableName).objectStore(storeName),
    request,
    cursor;

request = store.openCursor(); // 在这里可以传入配置好的键范围

// 读取属性值及使用方法
request.onsuccess = function(event) {
    cursor = event.target.result;
    
    if (cursor) {
        // 必须进行判断
        console.log("数据对象为" + JSON.stringify(cursor.value));
        cursor.continue(); // 游标移动到下一项
    }
};
```

#### 键范围

使用IDBKeyRange对象创建相应的键范围
IDBKeyRange对象有4中不同的创建方法：only（）、lowerBound（）、upperBound（）以及bound（）；

1. only：只取得主键值为指定值得数据。传递一个参数keyPathValue（主键值）；
2. lowerBound：从主键值为指定主键值的对象开始。传递两个参数：第一个keyPathValue（主键值），第二个为布尔值，表示是否包含满足条件的开始对象，true表示不包含，false表示包含，默认为false。
3. upperBound：到主键值为指定主键值的对象为止。传递参数与lowerBound方法相同，第二个参数为true代表不包含起止对象。
4. bound：最直接的方法，允许设置起始对象，结束对象，以及是否包含起止对象。

> 由于个人比较懒，在这里就不给大家撸代码了，直接贴上MDN文档中的代码，希望大家勿怪哈A_A

![IDBKeyRange][5]

### 总结

以上为个人总结的一些关于indexedDB操作的一些常用方式以及填的一些坑，希望对大家学习有帮助，如果有什么疑问错误的话，请指出，大家共同学习。另外，关于indexedDB更详细的操作方法请查询MDN文档或W3C官网，这里只是讲解一些常见的操作。
唉，写这篇博客真心累啊，期间从广州跑回学校，又从学校跑回广州，来来回回，千山万水！ - -宝宝心里苦啊···唉，还是希望这篇文章对大家有帮助吧。


[1]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-08-07/createObjectStore.png
[2]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
[3]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-08-07/transaction.png
[4]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-08-07/store.png
[5]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-08-07/IDBKeyRange.png