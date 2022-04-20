# 记一次Docsify装修gitalk 403问题

## 搭建博客

最近也是经常写点心得给自己，所以跟着小傅哥一起在github上搭建了一个属于自己的个人博客。搭建博客的教程可以参考[教小白使用 docsify，搭建一个贼简单的所见即所得博客！ (qq.com)](https://mp.weixin.qq.com/s/aK9Z9RkqWMUpcNzUREEx4Q)

如果你不想搭建docsify也可以购买云服务器使用linux+hexo或者wordpress的方式搭建，网络上方法很多，我就不一一教授了！~~（实话实说，我也不懂。）~~

## 装修gitalk

以下是自己踩坑过程，觉得啰嗦可以直接看总结部分，有代理地址可以直接用。

### 1.安装gitalk

博客什么的都搭建好了，但只放博客的话似乎有些不够看，有些笔记总要和人交流交流获得反馈才能知道自己还有哪些方面的不足。于是评论功能也是博客重要的一环。

不过小傅哥的博客似乎已经帮我们安装好了gitalk，试试能不能用？

![image-20220420154455803](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201544962.png)

OK，可以用，但不是完全能用，名字都不是我的，估计要修改配置才行。

如果你的博客没有安装gitalk。可以参照一下github上的官方教程。[Gitalk中文安装教程](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)

### 2.修改相关配置

很显然，这是因为我们的gitalk用的是别人的配置。

找到项目里有关`gitalkConfig`的地方。

修改为如下配置：

```js
    var gitalkConfig = {
      clientID: '4a657aa4ae6c0f1862a1',
      clientSecret: '94793d5bac0c884a8798ea941f7c5bc9fb004720',
      repo: 'Tegasus9.github.io',
      owner: 'Tegasus9',
      admin: ["Tegasus9"],
      distractionFreeMode: true
    };
```

**注意！这时候我只修改了repo，owner，admin相关配置**

我们提交到github仓库里，再打开网页然后登陆评论试试看？

![image-20220420155419024](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201554045.png)

？？？我们会发现，登陆完成之后报错并直接重定向到了原作者的博客。是哪里出了问题吗？

### 3.申请github application

**这时候我才登陆gitalk官网，查看教程，发现原来还需要申请github application**

于是按照教程，申请了application

![image-20220420155737182](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201557214.png)

然后将`clientId`和`clientSecret`填上自己的。

这次总该行了吧。

于是再次打开博客，登陆github。

![img](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201559757.png)

事情仍旧没有结束。

### 4.增加proxy代理

将`gitalk 403`问题放到百度去查找一番。发现gitalk其实默认用的是`https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token`这个网址进行跨域请求。而这个请求在`2021.1.31`开始就不再作为开放的代理服务了。那么我们请求这个地址就自然会报403错误。我们得自行更换代理地址。第一次碰见这类问题，手头上也没什么好的代理服务器，于是照搬了博客里的这个代理地址请求：`https://netnr-proxy.cloudno.de/https://github.com/login/oauth/access_token`

再次尝试。

![img](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201603974.png)

OK。很痛苦。

### 5.自行申请在线代理服务器

在百度上查询类似的`gitalk Network Error`无果，在github中的issues中也没有发现相关问题，似乎只剩下询问大佬这一条路，不过我还是相信自己的动手能力。于是按F12查看相关请求。

![img](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201605236.png)

429.表示一段时间内过多请求。果不其然，这个地址也被滥用了。看来博客的威力无处不在。

于是只好选择自行申请代理服务器这一条路。

根据这篇：[在cloudflare上创建一个免费的在线代理来解决gitalk授权登录报403问题 | 代码视界 (chenhanpeng.com)](https://www.chenhanpeng.com/create-own-cors-anywhere-to-resolve-the-request-with-403/#利用cloudflare-worker搭建在线代理)大佬文章我们可以知道。可以走利用cloudflare worker搭建在线代理这一条路来解决我们遇到的问题。

在cloudflare上建立一个账号之后，就可以申请我们的worker了。

**不过当时这个教程里修改github仓库里`index.js`这部分我却没有看懂。**比如黑白名单这一块，我只是访问github网站而已，那这黑白名单分别是什么呢？

所以误以为需要将worker里的代码复制到仓库里的哪个地方所以作罢。不过worker已经申请好了，可以先试试。

于是再次打开博客，登陆github，一气呵成。看到的却依旧是`Network Error`

？？能不能解决了，能不能解决了。

稳住，一定是有哪里掉了链子。

再次按F12打开开发者工具，看看后台返回了什么。**一共有两条发送到代理服务器的请求，其中一条提示CORS错误，另一条提示200**

![image-20220420161300645](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201613671.png)

OK。看来worker应该是搭建好了，但是却提示跨域错误。

于是又是一番百度查询资料。期间我询问了一个公司里的前端同事。以下是相关聊天记录。

![image-20220420161520594](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201615638.png)

虽然我并不是很懂前端，但理解它的回答并不难，综合一下百度相关资料。可以得出解决办法。就是在后端响应头上加一些跨域相关参数

```java
response.setHeader("Access-Control-Allow-Origin",'*');
```

但是就和我上图15:16分的问题一样。我没有后端哇。也没有nginx。不是用linux服务器搭建的。而是直接在github构建。难道这问题解决不了了？？

等等……等等……难道这个后端……就是worker？！

### 6.修改worker

这时候我好像突然明白这个worker是用来干什么的了。

果然，参考博客里让修改的index.js内容，其实就是worker中的代码。这个worker，就类似于一个小型应用，接受请求，然后处理请求，再响应请求。所以我们可以用它来做代理服务器。因为我们是用这个worker来访问github，而不是在博客中直接访问。

这样一来，黑白名单也可以顺利的理解了。

我们将index.js中的代码复制一下黏贴进worker里。如下，然后修改白名单。

```js
/*
CORS Anywhere as a Cloudflare Worker!
(c) 2019 by Zibri (www.zibri.org)
email: zibri AT zibri DOT org
https://github.com/Zibri/cloudflare-cors-anywhere
*/

/*
whitelist = [ "^http.?://www.chenhanpeng.com$", "chenhanpeng.com$" ];  // regexp for whitelisted urls
*/

blacklist = [ ];           // regexp for blacklisted urls
// whitelist = [ ".*" ];     // regexp for whitelisted origins
whitelist = [ "tegasus9.github.io" ]

function isListed(uri,listing) {
    var ret=false;
    if (typeof uri == "string") {
        listing.forEach((m)=>{
	          if (uri.match(m)!=null) ret=true;
        });
    } else {            //   decide what to do when Origin is null
    	  ret=true;    // true accepts null origins false rejects them.
    }
    return ret;
}

addEventListener("fetch", async event=>{
    event.respondWith((async function() {
        isOPTIONS = (event.request.method == "OPTIONS");
        var origin_url = new URL(event.request.url);

        function fix(myHeaders) {
            //            myHeaders.set("Access-Control-Allow-Origin", "*");
            myHeaders.set("Access-Control-Allow-Origin", event.request.headers.get("Origin"));
            if (isOPTIONS) {
                myHeaders.set("Access-Control-Allow-Methods", event.request.headers.get("access-control-request-method"));
                acrh = event.request.headers.get("access-control-request-headers");
                //myHeaders.set("Access-Control-Allow-Credentials", "true");

                if (acrh) {
                    myHeaders.set("Access-Control-Allow-Headers", acrh);
                }

                myHeaders.delete("X-Content-Type-Options");
            }
            return myHeaders;
        }
        var fetch_url = unescape(unescape(origin_url.search.substr(1)));

        var orig = event.request.headers.get("Origin");
        
        var remIp = event.request.headers.get("CF-Connecting-IP");

        if ((!isListed(fetch_url, blacklist)) && (isListed(orig, whitelist))) {

            xheaders = event.request.headers.get("x-cors-headers");

            if (xheaders != null) {
                try {
                    xheaders = JSON.parse(xheaders);
                } catch (e) {}
            }

            if (origin_url.search.startsWith("?")) {
                recv_headers = {};
                for (var pair of event.request.headers.entries()) {
                    if ((pair[0].match("^origin") == null) && 
			(pair[0].match("eferer") == null) && 
			(pair[0].match("^cf-") == null) && 
			(pair[0].match("^x-forw") == null) && 
			(pair[0].match("^x-cors-headers") == null)
		    ) recv_headers[pair[0]] = pair[1];
                }
		    
                if (xheaders != null) {
                    Object.entries(xheaders).forEach((c)=>recv_headers[c[0]] = c[1]);
                }

                newreq = new Request(event.request,{
                    "headers": recv_headers
                });

                var response = await fetch(fetch_url,newreq);
                var myHeaders = new Headers(response.headers);
                cors_headers = [];
                allh = {};
                for (var pair of response.headers.entries()) {
                    cors_headers.push(pair[0]);
                    allh[pair[0]] = pair[1];
                }
                cors_headers.push("cors-received-headers");
                myHeaders = fix(myHeaders);

                myHeaders.set("Access-Control-Expose-Headers", cors_headers.join(","));

                myHeaders.set("cors-received-headers", JSON.stringify(allh));

                if (isOPTIONS) {
                    var body = null;
                } else {
                    var body = await response.arrayBuffer();
                }

                var init = {
                    headers: myHeaders,
                    status: (isOPTIONS ? 200 : response.status),
                    statusText: (isOPTIONS ? "OK" : response.statusText)
                };
                return new Response(body,init);

            } else {
                var myHeaders = new Headers();
                myHeaders = fix(myHeaders);

                if (typeof event.request.cf != "undefined") {
                    if (typeof event.request.cf.country != "undefined") {
                        country = event.request.cf.country;
                    } else
                        country = false;

                    if (typeof event.request.cf.colo != "undefined") {
                        colo = event.request.cf.colo;
                    } else
                        colo = false;
                } else {
                    country = false;
                    colo = false;
                }

                return new Response(
                	"CLOUDFLARE-CORS-ANYWHERE\n\n" + 
                	"Source:\nhttps://github.com/Zibri/cloudflare-cors-anywhere\n\n" + 
                	"Usage:\n" + origin_url.origin + "/?uri\n\n" +
                	"Limits: 100,000 requests/day\n" + 
                	"          1,000 requests/10 minutes\n\n" + 
                	(orig != null ? "Origin: " + orig + "\n" : "") + 
                	"Ip: " + remIp + "\n" + 
                	(country ? "Country: " + country + "\n" : "") + 
                	(colo ? "Datacenter: " + colo + "\n" : "") + "\n" + 
                	((xheaders != null) ? "\nx-cors-headers: " + JSON.stringify(xheaders) : ""),
                	{status: 200, headers: myHeaders}
                );
            }
        } else {

            return new Response(
                "Create your own cors proxy</br>\n" + 
                "<a href='https://github.com/Zibri/cloudflare-cors-anywhere'>https://github.com/Zibri/cloudflare-cors-anywhere</a></br>\n" +
                "\nDonate</br>\n" +
                "<a href='https://www.chenhanpeng.com/create-own-cors-anywhere-to-resolve-the-request-with-403'>在cloudflare上创建一个免费的在线代理来解决gitalk授权登录报403问题</a>\n",
                {
                    status: 403,
                    statusText: 'Forbidden',
                    headers: {
                        "Content-Type": "text/html"
                    }
                });
        }
    }
    )());
});
```

也可以不设置，但是设置了白名单之后，就只有白名单之内的网址可以访问你的代理服务器，避免像之前那几个代理服务器一样被滥用然后报错429了。

### 7. 大功告成！

![image-20220420162817244](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201628273.png)

OK，我们再保存部署一下Worker。再次访问我们的博客。登陆。

![image-20220420162904255](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201629291.png)

果然不出所料！至此，我们的gitalk装修到此结束，来试试评论吧！

![image-20220420162945441](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204201629490.png)

YOHO！！！！！完结撒花！

## 总结

来总结一下装修我们自己gitalk的过程

1. 安装博客
2. 安装gitalk
3. 申请githubapplication
4. 申请在线代理服务器（并且部署好
5. 填写gitalk相关配置
6. 大功告成。