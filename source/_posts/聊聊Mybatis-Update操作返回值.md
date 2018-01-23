---
title: 聊聊Mybatis Update操作返回值
tags:
  - Mybatis
  - MySQL
  - JDBC
date: 2017-09-06 20:00:03
---


{%note info%}

后端的数据持久化使用的是 Mybatis ，在做高并发下账户增减余额的时候，打算使用乐观锁来解决这个问题。在获取update操作的返回值时遇到了一个问题，似乎 Mybatis 进行 update 操作得到的 `int` 返回值并不是影响的行数。这下就尴尬了。

{%endnote%}


一般而言，我们知道当我们使用 Mybatis 在 `mapper` 接口中定义 `insert` `delete` 等操作，定义一个 `int` 类型的返回值，通过该值是否为 0 来判断数据库中受影响的行数进而判断操作是否成功。

<!--more-->

------------



到底 `update` 返回值代表什么呢？我们来验证一下便知道了，假设有如下一张表以及两条数据：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja7y057g3j20x004yjst.jpg "数据库更新之前记录")

我们来编写一个简单的单元测试用例来验证下,首先使用 mybatis 简单的写个 mapper 进行更新操作，其中 xml 中的内容为：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja7tn625wj210q0ekgnn.jpg "mapper 更新语句")

数据库连接配置为：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja7wi2iv9j20se0k076a.jpg "数据库连接配置")

接来下，我们来编写一个简单的单元测试来验证下：**update 的返回值是不是受影响的记录的条数**，对应的单元测试代码如下：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja83t5sf3j20sg0oggoq.jpg "单元测试代码")

由单元测试代码可以得知，我们将要把数据库中两条记录的 `phone` 字段的值由 `12345678` 修改为 `66666666` ,正常情况下，`resultCode` 将会返回 2 。因为 `update` 操作影响到数据库中这 2 条记录，这和我们期望 2 是相符合的。那么一切正常的情况下，这次单元测试将会通过，那么我们运行看看结果：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja8bdifgjj20jq0423yx.jpg "单元测试通过")

单元测试通过了，再查看数据库中的记录：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja8ifpsyyj20x206u3zr.jpg "更新后的数据库记录")

这说明 mybatis 的 update 更新操作返回值的确是返回受影响的行数......真的是这样吗？

![](https://ws1.sinaimg.cn/large/694830ebgy1fja8m25otsj204305h0sm.jpg)

我们知道，当数据库中的记录被修改之后，再次执行重复的 `update` 操作将不会影响到新的行数，为了验证我说的话，我们试试：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja8rrcju0j20ne03i74t.jpg)

那么，按照这个逻辑：我们再次执行这个单元测试应该是，`resultCode` 返回的应该是 `0` ，和我们的期望的数字 `2` 不一致，将会导致测试不通过。再次运行单元测试：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja8xe6fwgj20ju050t9b.jpg "单元测试依然通过")

居然还是 `passed` ,看到这里聪明的你已经看出来了，<font color='red'>默认情况下，mybatis 的 `update` 操作返回值是记录的  `matched` 的条数，并不是影响的记录条数。</font>

-------------------

严格意义上来将，这并不是 mybatis 的返回值，mybatis 仅仅只是返回的数据库连接驱动（通常是 `JDBC` ）的返回值，也就是说，如果驱动告知更新 2 条记录受影响，那么我们将得到 mybatis 的返回值就会是 2 和 mybatis 本身是没有关系的。



道理我都懂，如果我们非得要 mybatis 的 `update` 操作明确的返回受影响的记录条数，有没有什么办法呢？

当然是有的。

通过对 `JDBC` URL 显式的指定 **`useAffectedRows`**选项，我们将可以得到受影响的记录的条数：

```java
jdbc:mysql://${jdbc.host}/${jdbc.db}?useAffectedRows=true
```

我们对我们的数据库连接配置稍做修改，添加 `useAffectedRows` 字段：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja9nv1e5xj20u80kcdhz.jpg "修改之后的数据库连接配置")

此时，mybatis 的 update 操作返回的应该是受影响的条数了，我们再次运行单元测试试试看：

![](https://ws1.sinaimg.cn/large/694830ebgy1fja9psbcjpj21hy08eabi.jpg "单元测试不通过")

`update` 操作返回的是受影响的记录条数，我们知道为 `0` 和我们预期的 `2` 不一致，自然而然单元测试不通过咯～。

##### 相关阅读

- ["Return number of changed rows"](http://mybatis-user.963551.n3.nabble.com/Return-number-of-changed-rows-td3888464.html)
- ["mybatis实现乐观锁"](http://www.peristblog.com/c?id=10164)