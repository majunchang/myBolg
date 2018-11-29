---
title: cookie和web存储的比较
abbrlink: 5140aa99
date: 2017-12-19 20:11:37
tags:
---
## cookie


### 基本概念
1. cookie非常小，限制在4kb左右，很多浏览器都限制一个站点最多保存20个cookie。
2. 如果没有设置时间，则表示cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就会消失，这种被称为会话cookie，它会被保存在内存中。
3. 当设置了过期时间，浏览器会把cookie保存在硬盘中，关闭浏览器之后任然有效，直到超过设定的过期时间。
### 设置和获取cookie的方法

> 原生
```
// 使用js创建cookie
document.cookie="username=John Doe";
// 添加一个过期时间
document.cookie="username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 GMT";
// 使用path 告诉浏览器cookie的路径
document.cookie="username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 GMT; path=/";


//读取
var x = document.cookie;
// 修改  旧的值将会被覆盖
document.cookie="username=John Smith; expires=Thu, 18 Dec 2013 12:00:00 GMT; path=/";

// 删除 删除 cookie 非常简单。您只需要设置 expires 参数为以前的时间即可
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 GMT";
```
> 封装

```
function setCookie(cname,cvalue,exdays)
{
  var d = new Date();
  d.setTime(d.getTime()+(exdays*24*60*60*1000));
  var expires = "expires="+d.toGMTString();
  document.cookie = cname + "=" + cvalue + "; " + expires;
}

function getCookie(cname)
{
  var name = cname + "=";
  var ca = document.cookie.split(';');
  for(var i=0; i<ca.length; i++) 
  {
    var c = ca[i].trim();
    if (c.indexOf(name)==0) return c.substring(name.length,c.length);
  }
  return "";
}
```
## localStorage和sessionStorage 

#### 优势

- 扩展了cookie的4k限制，为了更大的容量存储而设计的，是在浏览器端存储的数据
- 减少网络流量，快速的读取数据，性能较好，可以作为临时存储
- localStorage是永久性存储，而sessionStorage属于当会话结束的时候，就会被清空

#### 劣势 

- 本质上是对字符串的读取，内容较多的时候 会消耗内存，导致页面变卡，
- 不能被爬虫抓取到


## 三者的异同




特性名称|cookie | localStorage | sessionStorage |
---|---|---|---|
数据的声明周期 | 可设置失效时间，默认是关闭浏览器后失效 | 除非被清除，否则永久保存 | 仅仅在当前会话下有效，关闭页面或者浏览器后会被清除 |
存放的数据大小 | 4k左右 | 一般为5M | 一般为5M |
与服务端通信 | 会在http头中携带，如果使用cookie保存过多数据会带来性能问题 | 仅在浏览器端保存不参与服务器的通信 | 仅在浏览器端保存不参与服务器的通信 |
易用性  | 需要自己封装 | 有现成的api接口可以使用 | 有现成的api接口可以使用 |


