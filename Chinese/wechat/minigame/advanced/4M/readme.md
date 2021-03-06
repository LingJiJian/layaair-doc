# 微信小游戏4M本地包的使用

通常我们开发项目的时候，会直接使用本地路径，比如示例中引用的就是本地路径，

```javascript
Laya.Texture2D.load("res/layabox.png");
```

如果项目的目录中，全部大小加起来不超过4M的话，只要能找到本地的资源，怎么写也没问题。

但是，

微信小游戏的本地包有4M的限制，一旦超过这个限制，那就不允许上传，不允许真机预览。

所以，我们的**项目要是大于4M后，怎么处理呢？**

要进行资源目录的规划，分为本地加载与网络动态加载，两种模式结合使用。

本地加载的规划里，我们除了入口文件和必用的配置文件外，只放一些预加载必用的素材，比如加载进度（Loading）页用到的背景与图形等。总之，就是不能超过4M。

> Tips：需要提醒注意的是微信小游戏不允许动态加载创建JS，所以，JS必须要放在4M包里，也就是说JS加上基础配置文件必须要小于4M，项目适配时如果超过4M，要进行优化控制。



**网络动态加载的路径怎么处理呢**。在本地加载的`load()`方法之后使用`URL.basePath`方法。

例如：

```javascript
material.diffuseTexture = Laya.Texture2D.load("res/layabox.png");
box.meshRender.material = material;
Laya.URL.basePath = "https://XXXX.com";//请把XXXX换成自己的真实网址；
//在此之下，再使用load加载资源，都会自动加入URL网址。从网络上动态加载。
```

使用`URL.basePath`方法后，再使用load加载本地路径，都会自动加上URL.basePath里的网址。这样就实现了本地与网络加载的结合。



**这样就结束了吗？并没有！**

按刚刚的写法，`res/layabox.png`明明已经上传到微信小游戏的本地目录，但是如果在使用`URL.basePath`之后，再次加载`res/layabox.png`并不会从本地加载使用，而是从网络动态加载使用。这并不是我们要的结果。



所以，引擎针对使用`URL.basePath`之后，如何再次使用本地加载，进行了**特殊目录和文件的处理**，也就是本地包白名单机制。如下例所示：

```javascript
MiniAdpter.nativefiles =  [
    "wxlocal",
    "res/atlas/houzi.atlas",
    "res/atlas/houzi.png",
    "common/tishi.png",
    "common/bg.png",
    "ui.json",
    "newLb/bg031.png"
];
```

**只要是MiniAdpter.nativefiles里存在的目录名或文件，引擎会自动将该目录视为本地目录**，即便使用了URL.basePath，对于包含在nativefiles白名单内的目录名或文件，都不会从网络动态加载，只会从本地加载。