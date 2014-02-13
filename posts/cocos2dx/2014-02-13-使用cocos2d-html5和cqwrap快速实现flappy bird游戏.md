## 使用 cocos2d-html5 和 cqwrap 快速实现 Flappy Bird 游戏

毫无疑问，最近手机游戏届最大的赢家无疑是 Flappy Bird 的作者。谁也没想到这样一款简单的脑残游戏竟然能够火爆到这种程度。

惊讶归惊讶吧，作为程序猿，我们更多时候还是淡定滴研究技术，因此本文也还是以如何开发这样一款“脑残游戏”为讨论主题。

### 游戏的准备工作

因为我们要做一个一模一样的游戏，所以我们准备与官方扑腾小鸟长得非常相似的素材。由于这款游戏实在太火爆了，因此网络上各种版本的山寨，所以素材并不难找。将这些素材使用 Texture Packer 工具拼接好，那么我们的游戏准备工作就完成了。

### 使用类库

在这里我们使用 cocos2d-html5-2.2.2，在 [cocos2d-html5官方](https://github.com/cocos2d/cocos2d-html5/releases)，可以下载到这个版本。

另外我们使用一个叫做 cqwrap 的库，这是一个我为 cocos2dx-jsb 和 cocos2dx-html5 定制的扩展，我在前一个开源的游戏 HappyGo 中也使用了它，你可以在[这里](http://go.akira-cn.gitpress.org/)看到我为它写的一个简易的使用文档，可以从 github 的这个项目中获得它的代码。

<!--more-->

### 开始编码

素材有了，框架也有了，这样就可以开工了。

首先，这样一个简单的游戏，我们只需要一个场景，我们把它命名为 PlayScene

在 src/view/ 目录下创建一个 play_scene.js 的文件，

```js
var layers = require('cqwrap/layers'),
    BaseScene = require('cqwrap/scenes').BaseScene;
var GameLayer = layers.GameLayer, BgLayer = layers.BgLayer;

var MyScene = BaseScene.extend({
    init:function () {
        this._super();

        //添加静态的背景
        var bg = new BgLayer("res/bg.png");
        this.addChild(bg);
    }
});

module.exports = MyScene;
```

我们首先创建了一个场景，在这个场景里面，我们添加了一个静态的背景层。在cqwrap中，封装了一个BgLayer的类，用来创建静态的背景层。

这样，我们就建立了一个只有一个远背景层的场景。

### 创建前景

我们在背景的基础上创建一个前景层，小鸟、水管和其他元素都在这个层上

```js

var layers = require('cqwrap/layers'),
    BaseScene = require('cqwrap/scenes').BaseScene;
var GameLayer = layers.GameLayer, BgLayer = layers.BgLayer;

var MyScene = BaseScene.extend({
    init:function () {
        this._super();

        //添加静态的背景
        var bg = new BgLayer("res/bg.png");
        this.addChild(bg);

        //添加前景层
        var layer = new MyLayer();
        this.addChild(layer);
    }
});

module.exports = MyScene;
```

我们创建了一个MyLayer实例，作为前景层，我们在下面的代码里展示 MyLayer 的基本结构：

```js

var MyLayer = GameLayer.extend({
    init: function(){
        this._super();
        return true;
    }
});
```

我们仅仅创建了一个游戏层，这个层继承自 GameLayer，GameLayer是cqwrap提供的一个类，用来创建包括精灵和动作事件的层。

### 在游戏场景中加载资源

首先我们得把素材资源加载进来，在 cocos2d-html5 中，我们可以通过 SpriteFrameCache 将之前打包的资源加载进来

```js

var MyLayer = GameLayer.extend({
    init: function(){
        this._super();
        
        //加载游戏素材资源
        var cache = cc.SpriteFrameCache.getInstance();
            cache.addSpriteFrames("res/flappy_packer.plist", "res/flappy_packer.png");

        return true;
    }
});
```

### 实现一个“往前移动”的效果

在 Flappy Bird 中，小鸟看起来在往前飞翔，是因为用地面的图片做了一个动画效果

```js
var MyLayer = GameLayer.extend({
    init: function(){
        this._super();
        
        //加载游戏素材资源
        var cache = cc.SpriteFrameCache.getInstance();
            cache.addSpriteFrames("res/flappy_packer.plist", "res/flappy_packer.png");

        //显示地面
        var ground = cc.createSprite('res/ground.png', {
            anchor: [0, 0],
            xy: [0, 0],
            zOrder: 3
        });

        //让地面运动起来
        ground.moveBy(0.5, cc.p(-120, 0)).moveBy(0, cc.p(120, 0)).repeatAll().act();

        this.addChild(ground);
        
        return true;
    }
});
```

### 飞翔的小鸟

有了一个会“前进”的地面之后，我们只要在空中放置一只小鸟，那么就可以制造出飞翔的效果了：

```js
//我是一只小小小小鸟
var bird = cc.createSprite('bird1.png', {
    anchor: [0.5, 0],
    xy: [220, 650],
    zOrder: 2
});

this.addChild(bird);        
```

cc.createSprite 是一个非常方便的方法，它是由cqwrap库提供的额外方法，可以用来更方便地创建各种 Sprite.

现在我们创建了一只在空中的小鸟，它看起来会不断向前进，但是看起来比较奇怪，因为它的翅膀是不动的。

我们可以执行动画，让小鸟动起来——

```js

//给小鸟增加飞行动画
bird.animate(0.6, 'bird1.png', 'bird2.png', 'bird3.png').repeat().act();
bird.moveBy(0.3, cc.p(0, -20)).reverse().repeatAll().act();
```

现在，我们拥有了一只在空中不断飞翔的小鸟

### 响应 touch/鼠标 事件

在 cqwrap 中，一个 GameLayer 是一个代理，它可以代理精灵们的事件，甚至可以代理自身的事件——

```js
this.delegate(this);    //将Layer的touch事件代理给Layer自身
this.on('touchstart', function(){
    //事件处理函数 
});
```

### 让小鸟往高处飞

我们添加让小鸟往高处飞的事件——

```js
this.on('touchstart', function(){
    bird.stopAllActions();
    bird.animate(0.2, 'bird1.png', 'bird2.png', 'bird3.png').repeat().act();

    var jumpHeight = Math.min(1280 - birdY, 125);
    bird.moveBy(0.2, cc.p(0, jumpHeight)).act();

    bird.rotateTo(0.2, -30).act(); 
});
```

### 让小鸟会掉落下来

```js
this.on('touchstart', function(){
    var birdX = bird.getPositionX();
    var birdY = bird.getPositionY();
    var fallTime = birdY / 1000;

    bird.stopAllActions();
    bird.animate(0.2, 'bird1.png', 'bird2.png', 'bird3.png').repeat().act();

    var jumpHeight = Math.min(1280 - birdY, 125);
    bird.moveBy(0.2, cc.p(0, jumpHeight)).act();

    bird.rotateTo(0.2, -30).act();
    bird.delay(0.2).moveTo(fallTime, cc.p(birdX, 316), cc.EaseIn, 2)
        .then(function(){
            //小鸟掉落下来了

        }).act();
    bird.delay(0.2).rotateTo(fallTime, 90, 0, cc.EaseIn, 2).act();    
});
```
