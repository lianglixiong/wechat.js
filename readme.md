# 重要提示：

目前，新版的微信 js 出来了，你可能暂时不需要这个版本了。详情见：[微信 JSSDK 说明](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html#.E5.9B.BE.E5.83.8F.E6.8E.A5.E5.8F.A3)。这个版本非常强大，可以传图图片、语音，同时也有更高的要求，需要签名。


----------

# wechat.js

微信打开 DEMO 地址：[http://sofish.github.io/wechat.js](http://sofish.github.io/wechat.js/)，或者扫一扫下面的二维码进行分享：

![sofish/wechat.js](http://ww4.sinaimg.cn/large/61b90cbegw1eknqgwosn6j203p03pglk.jpg)

### 安装

使用 bower 管理 `wechat.js` 的依赖：
```bash
$ bower install wechat.js --save
```
更新 `wechat.js` 依赖版本：
```bash
$ bower update
```

> 下面是 API 详解，使用可参考上面 DEMO 的源代码。微信的 API 是有点恶心的，也不断在变，如果发现问题请给提 issue 或者 pull-request 吧。

### 1、使用指南

**一、唯一接口：`wechat`，有「分享」+ 「操作」两种类型**

```js
// 分享
wechat('friend', data, callback);           // 朋友
wechat('timeline', data, callback);         // 朋友圈
wechat('weibo', data, callback);            // 微博
wechat('email', data, callback);            // 邮件分享

// 操作
wechat('hideToolbar', callback);            // 隐藏底部菜单
wechat('hideOptionMenu', callback);         // 隐藏右上角分享按钮
wechat('showOptionMenu', callback);         // 显示右上角分享按钮
wechat('closeWebView');                     // 关闭webview
wechat('scanQRCode');                       // 跳转到扫描二维码页面
wechat('imagePreview', imgData, callback);  // 图片预览/查看大图
// imgData = {
//   current: 'picture1.jpg',               // 要预览的当前张url
//   urls: ['picture1.jpg', 'picture2.jpg'] // 所有图片的url列表
// }

wechat('network', callback);                // 查看用户当前网络
// 1. wifi
// 2. edge 非 wifi,包含 3G/2G
// 3. fail 网络断开连接
// 4. wwan 2g/3g
```

**二、`data` 「属性」支持函数**

因为有些数据是需要拼接，或者在点击分享按钮的时候可能才存在的，但是又不想写很麻烦时机判断，这里 `data` 中支持传入函数，比如：

```js
// 一般的数据
var data = {
  'app': 'APP ID',    // 选填，默认为空
  'img': '图片 URL',   // 选填，默认为空或者当前页面第一张图片
  'link': '链接',
  'desc': '描述',
  'title': '标题'
};

// 假设我们在一个单页应用，title 可能是 js 在数据载入后才有的，那么可以这样来：
var getTitle = function() {
  return document.title;
};

// 这个数据 ，最终 wechat.js 会自动转换
var data = {
  // 这里需要特别说明的是，建议不要用新浪微博的图片地址，要么你试试，哈哈
  'img': '图片 URL',
  
  'link': '链接',
  'desc': '描述',
  'title': getTitle()
};

// 发送邮件
var data = {
    title: "邮件标题",
    content: "邮件内容"
};
```

**三、回调**

```js
var callback = function() {
  // 返回的数据并不统一，接口已经尽量统一，我觉得微信公司现在缺 js 程序员
  // 也有一些是很恶心的
  console && console.log(argument);
};

wechat('timeline', data, callback);
```

**四、封装**

```js
(function(){
    var wxShare = function(opts){
        var defaults = {
            getWxConfigUrl: '',         // 获取微信接口权限url
            orDebug: false,             // 是否开启调试模式
            title: '',                  // 分享标题
            desc: '',                   // 分享描述
            wxUrl: '',                  // 参与签名的url
            link: '',                   // 分享链接
            imgUrl: '',                 // 分享图标
            success: function(){},      // 用户确认分享后执行的回调函数
            cancel: function(){}        // 用户取消分享后执行的回调函数
        };

        var _opts = $.extend(defaults, opts);

        // 获取微信权限
        $.ajax({
            url: _opts['getWxConfigUrl'],
            type: 'POST',
            dataType: 'json',
            data: {share_url: _opts['wxUrl']},
        })
        .done(function(res) {
            if(typeof res == 'object'){
                // 通过config接口注入权限验证配置
                wx.config({
                    // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                    debug: _opts['orDebug'],
                    // 必填，公众号的唯一标识
                    appId: res.appId,
                    // 必填，生成签名的时间戳
                    timestamp: res.timestamp,
                    // 必填，生成签名的随机串
                    nonceStr: res.nonceStr,
                    // 必填，签名，见附录1
                    signature: res.signature,
                    // 必填，需要使用的JS接口列表
                    jsApiList: ['onMenuShareTimeline', 'onMenuShareAppMessage', 'onMenuShareQQ']
                });
                // 处理config验证成功或失败
                _wxReady(res.url);
            }else{
                console.log('获取微信接口权限失败！');
            }
        })
        .fail(function() {
            console.log("获取微信接口权限失败！");
        });

        // 处理config验证成功或失败
        var _wxReady = function(urls){
            // wxConfig验证成功处理
            wx.ready(function(){

                // 分享到朋友圈
                wx.onMenuShareTimeline({
                    title: _opts['title'],
                    link: _opts['link'],
                    imgUrl: _opts['imgUrl'],
                    success: _opts['success'],
                    cancel: _opts['cancel']
                });

                // 分享给朋友
                wx.onMenuShareAppMessage({
                    title: _opts['title'],
                    desc: _opts['desc'],
                    link: _opts['link'],
                    imgUrl: _opts['imgUrl'],
                    // type: '', // 分享类型,music、video或link，不填默认为link
                    // dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
                    success: _opts['success'],
                    cancel: _opts['cancel']
                });

                // 分享到QQ
                wx.onMenuShareQQ({
                    title: _opts['title'],
                    desc: _opts['desc'],
                    link: _opts['link'],
                    imgUrl: _opts['imgUrl'],
                    success: _opts['success'],
                    cancel: _opts['cancel']
                });
            });

            // wxConfig验证失败返回函数
            wx.error(function(res){
                console.log(res,'wxConfig验证失败')
            });
        };
    };

    window.wxShare = wxShare;

})();
```
调用：
```js
//是否微信内置浏览器
if(ua.match(/MicroMessenger/i) == 'micromessenger'){
    // 微信分享
    new wxShare({
        getWxConfigUrl: "../advice/getWxConfig",
        title: '【八百方】送你68元红包，助攻你的618',
        desc: '会员神券日，来抢61.8元无门槛！',
        wxUrl: thisUrl,
        link: thisUrl,
        imgUrl: "http://www.800pharm.com/shop/js/sales/activity618/images/20190606154315.png",
        success:function(){
            alert('分享成功')
        }
    });
}
```

### 2、授权

[MIT License](license.txt)
