---
title: emoji带来的思考
date: 2021-03-27 13:29:20
tags: ["前端", "后端"]
---

### 问题描述

之前开发的[Offertalk](https://www.hmbstudio.cn/)具有留言评论的功能，王保长说：现在而今眼目下，这种留言评论不能支持emoji【😀😄😆😝😆😸】那是说不过去的。后来开发的[树洞有你](http://qcblog.hmbstudio.cn/items/)具有文章发布的功能，同样也是需要emoji支持的。

今天跟老虎哥聊到他的游戏，[天天爱烹饪]() ，有一个问题，就是用户需要在昵称上加入emoji图标，这就出现了和我上述同样的问题。

我们最开始都简单的对字段进行了utf8编码设置，并未考虑到emoji这种情况

### 问题原因

UTF-8编码有可能是两个、三个、四个字节，其中Emoji表情是4个字节，有的甚至6个字节，而Mysql的utf8编码最多3个字节，所以导致了数据insert error。

### 解决方案

方案一：更改MySQL数据库表设置，设置utf8的超级：utf8mb4字符集

1. 修改mysql配置文件，重启mysql服务。

```mysql

[mysql]

default-character-set=utf8mb4

[mysqld]

character-set-client-handshake = FALSE

character-set-server = utf8mb4

collation-server = utf8mb4_unicode_ci

init_connect='SET NAMES utf8mb4'

```

2. 修改表中字段的字符集（坑）

```mysql

ALTER TABLE *** CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

ALTER TABLE **** CHANGE `user_name` `user_name` VARCHAR(50) CHARSET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '用户名。';

```

3. 程序的连接池设置为 utf-8

4. 坑及注意事项  

a.mysql数据库版本必须得是5.5.3及以后的

b.进行数据库配置更改修改多注点意，之前差点就把自己的线上环境的数据库玩崩了，定期备份很重要啊！😆

方案二：对需要emoji存入的数据进行base64转码存入，前端进行base64的解码即可

1.纯前端的base64编解码：众多浏览器原生实现了Base64编码、解码标准方法

```javascript

let nick = "我是柳博文😆"

let encode = window.btoa(encodeURIComponent("我是柳博文😆"));

console.log(decodeURIComponent(window.atob(encode)));

// 我是柳博文😆

```

2.接入层Node的base64编解码：

```javascript

let geek = "hello 柳博文😆";

let encode = Buffer.from(geek).toString('base64');// SGVsbG8g5p+z5Y2a5paH8J+Yhg==

console.log(Buffer.from('SGVsbG8g5p+z5Y2a5paH8J+Yhg==', 'base64').toString())

// Hello 柳博文😆

```

3.服务层Golang的base64编解码：

```Golang

package main

import (

    "encoding/base64"

    "fmt"

    "log"

)

func main() {

    input := []byte("hello world")

    // 编码

    encodeString := base64.StdEncoding.EncodeToString(input)

    fmt.Println(encodeString)

    // 对上面的编码结果进行base64解码

    decodeBytes, err := base64.StdEncoding.DecodeString(encodeString)

    if err != nil {

        log.Fatalln(err)

    }

    fmt.Println(string(decodeBytes))

    // 如果要用在url中，需要使用URLEncoding

    uEnc := base64.URLEncoding.EncodeToString([]byte(input))

    fmt.Println(uEnc)

    uDec, err := base64.URLEncoding.DecodeString(uEnc)

    if err != nil {

        log.Fatalln(err)

    }

    fmt.Println(string(uDec))

}

// aGVsbG8g5p-zIPCfmIY=

// hello 柳 😆

```

### 思考总结

就日常开发中，必定会遇见许多问题，有些问题是开发性及时问题能够快速的解决，但有些问题可能就会使整个软件架构问题，例如上述游戏出现的字符编码问题，就不是很合适进行底层数据库的修改，因为**目前有个索引，换mb4就超过了最大允许1000位索引**，更改会带来巨大的成本，我理解为这可能就是架构层面的问题了。但是，也不妨从方案二在非数据层进行问题的解决，这也不失为一个好的解决方案。有时候会体感到一个软件或者系统的结构是一个正三角的造型，我们可以从各种层面解决问题，用户接触到的只是冰山一角。但作为开发者的时候，有时候又会体感到软件生命周期是一个倒三角造型，哈哈哈，奇妙！
