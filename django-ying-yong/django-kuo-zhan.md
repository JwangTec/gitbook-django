---
title: Rest_Framework 扩展
date: '2019-06-21 19:01:53 +0800'
tags: Rest_Framework
categories: django
---

# get/put/post/delete区别

## 含义

| 名称 | url | 含义 |
| :--- | :--- | :--- |
| POST | /uri | 创建 |

DELETE \|/uri/xxx\| 删除  
PUT \|/uri/xxx\| 更新或创建  
GET \|/uri/xxx \|查看

## 区别

1.GET操作是安全的。

```text
所谓安全是指不管进行多少次操作，资源的状态都不会改变。比如我用GET浏览文章，不管浏览多少次，那篇文章还在那，
没有变化。当然，你可能说每浏览一次文章，文章的浏览数就加一，这不也改变了资源的状态么？这并不矛盾，因为这个改变不是GET操作引
起的，而是用户自己设定的服务端逻辑造成的。
```

2.PUT，DELETE操作是幂等的。

```text
所谓幂等是指不管进行多少次操作，结果都一样。比如我用PUT修改一篇文章，然后在做同样的操作，每次操作后的结果并没有不同，
DELETE也是一样。顺便说一句，因为GET操作是安全的，所以它自然也是幂等的。
```

3.POST操作既不是安全的，也不是幂等。

```text
比如常见的POST重复加载问题：当我们多次发出同样的POST请求后，其结果是创建出了若干的资源。
```

4.安全和幂等的意义在于：

```text
    当操作没有达到预期的目标时，我们可以不停的重试，而不会对资源产生副作用。从这个意义上说，POST操作往往是有害的，但很多时
候我们还是不得不使用它。还有一点需要注意的就是，创建操作可以使用POST，也可以使用PUT，区别在于POST 是作用在一个集合资
源之上的（/uri），而PUT操作是作用在一个具体资源之上的（/uri/xxx），再通俗点说，如果URL可以在客户端确定，
    那么就使用PUT，如果是在服务端确定，那么就使用POST，比如说很多资源使用数据库自增主键作为标识信息，而创建的资源的标识信息
到底是什么只能由服务端提供，这个时候就必须使用POST。
```

## 关于GET POST 的混淆

1.先说相同点，只有了解了相同点之后才能理解为什么会发生混淆。两者都能向服务器发送数据，提交的“内容”\[注1\]的格式相同，都是

```text
param1=value1&param2=value2&....  
```

get 和 post 区别如字面，一个是get（获取），一个是post（发送）。

```text
    get用来告诉服务器需要获取哪些内容（uri+query），向静态页面（uri）请求则直接返回文件内容给浏览器，向一个动态页面请求时
可以提供查询参数（query）以获得相应内容。

    post用来向服务器提交内容，主要是为了提交，而不是为了请求内容，就是说post的初衷并不要求服务器返回内容[注2]，只是提交内
容让服务器处理（主要是存储或者处理之后再存储）。
```

2.get和post出现混淆是因为对提交的数据处理方法的滥用造成的，数据是无辜的。

混淆之一：

```text
将get提交的用来查询的字段当作是存储数据存入了服务器端文件或者数据库。然后就误以为get是用来提交用于存储的数据的。
```

混淆之二：

```text
编写脚本在服务器端通过处理post提交的数据并返回内容。只要有数据，就能用来进行判断，脚本怎写是程序员的事，而不在乎数据来源的形
式（post、get，或者是自己预设值的常量）。这点功能上确实没问题，只是背离的其初始目的而已。
```

由于都是要传送数据，且数据格式相同（即使数据格式不同，只要能提取出相应数据）。使用的时候难免出现张冠李戴，将get数据用来存储、将post数据用来检索返回数据。

3.二者区别（用途而“人为”造成）：

```text
    get的长度限制在2048字节（由浏览器和服务器限制的，这是目前IE的数据，曾经是1024字节），很大程度上限制了get用来传递“存
储数据”的数据的能力，所以还是老老实实用来做检索吧；
    post则无此限制（只是HTTP协议规范没有进行大小限制，但受限于服务器的处理能力），因此对于大的数据（一般来说需要存储的数据
可能会比较大，比2048字节大）的传递有天然的优势，谁让它是 nature born post 呢。


    get提交的数据是放在url里，目的是灵活的向服务其提交检索请求，可以在地址栏随时修改数据以变更需要获取的内容，比如直接修改
分页的编号就跳到另外一个分页了（当然也可能是 404）。

    post提交的数据放在http请求的正文里，目的在于提交数据并用于服务器端的存储，而不允许用户过多的更改相应数据（主要是相对于
在url 修改要麻烦很多，url的修改只要点击地址栏输入字符就可以了），除非是专门跑来编辑数据的。

    post和get的安全性在传输的层面上区别不大，但是采用url提交数据的get方式容易被人肉眼看到，或者出现在历史纪录里，还是可能
被肉眼看到，都是一些本地的问题。
```

注：get方式主要是为了获得预期内容，即uri+query相同时所得到的内容应该是相同的。而post主要是提交内容，至于是否有必要返回页面可 能只是出于用户体验，比如注册时返回你的注册id，但是如果只是返回一个“您已注册成功”的相同页面（即使你post的数据不一样）也没什么好奇怪的。

## HTTP POST GET 本质区别

1.原理区别

```text
一般在浏览器中输入网址访问资源都是通过GET方式；在FORM提交中，可以通过Method指定提交方式为GET或者POST，默认为GET提交 
Http定义了与服务器交互的不同方法，最基本的方法有4种，分别是GET，POST，PUT，DELETE URL 全称是资源描述符，我们可以这样认
为：一个URL地址，它用于描述一个网络上的资源，而HTTP中的GET，POST，PUT，DELETE就对应着对这个资源的查 ，改 ，增 ，删 4个
操作。到这里，大家应该有个大概的了解了，GET一般用于获取/查询 资源信息，而POST一般用于更新 资源信息(个人认为这是GET和POST
的本质区别，也是协议设计者的本意，其它区别都是具体表现形式的差异 )。 　　
```

2.根据HTTP规范，GET用于信息获取，而且应该是安全的和幂等的 。

```text
1.所谓安全的意味着该操作用于获取信息而非修改信息。换句话说，GET请求一般不应产生副作用。就是说，它仅仅是获取资源信息，就像数
据库查询一样，不会修改，增加数据，不会影响资源的状态。 
　　
* 注意：这里安全的含义仅仅是指是非修改信息。 　　

2.幂等的意味着对同一URL的多个请求应该返回同样的结果。这里我再解释一下幂等 这个概念： 　　

幂等 （idempotent、idempotence）是一个数学或计算机学概念，常见于抽象代数中。 　　

    幂等有以下几种定义：对于单目运算，如果一个运算对于在范围内的所有的一个数多次进行该运算所得的结果和进行一次该运算所得的结
    果是一样的，那么我们就称该运算是幂等的。比如绝对值运算就是一个例子，在实数集中，有abs(a) = abs(abs(a)) 。 　　

    对于双目运算，则要求当参与运算的两个值是等值的情况下，如果满足运算结果与参与运算的两个值相等，则称该运算幂等，如求两个数
    的最大值的函数，有在在实数集中幂等，即max(x,x) = x 。 看完上述解释后，应该可以理解GET幂等的含义了。 　　

    但在实际应用中，以上2条规定并没有这么严格。引用别人文章的例子：比如，新闻站点的头版不断更新。虽然第二次请求会返回不同
    一批新闻，该操作仍然被认为是安全的和幂等的，因为它总是返回当前的新闻。从根本上说，如果目标是当用户打开一个链接时，他可以
    确信从自身的角度来看没有改变资源即可。 

     根据HTTP规范，POST表示可能修改变服务器上的资源的请求 。继续引用上面的例子：还是新闻以网站为例，读者对新闻发表自己的评
     论应该通过POST实现，因为在评论提交后站点的资源已经不同了，或者说资源被修改了。 　　
```

3.上面大概说了一下HTTP规范中，GET和POST的一些原理性的问题。但在实际的做的时候，很多人却没有按照HTTP规范去做，导致这个问题的原因有很多，比如说：

```text
 　　        
    1.很多人贪方便，更新资源时用了GET，因为用POST必须要到FORM（表单），这样会麻烦一点。　　
    2.对资源的增，删，改，查操作，其实都可以通过GET/POST完成，不需要用到PUT和DELETE。 　　
    3.另外一个是，早期的但是Web MVC框架设计者们并没有有意识地将URL当作抽象的资源来看待和设计 。还有一个较为严重的问题是
    传统的Web MVC框架基本上都只支持GET和POST两种HTTP方法，而不支持PUT和DELETE方法。
```
