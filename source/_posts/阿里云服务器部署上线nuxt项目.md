---
title: 阿里云服务器部署上线nuxt项目
tags: ['nuxt']
categories: ['nuxt']
date: 2021-11-17 16:21:59
---


最近公司重新开发的企业官网，考虑企业官网对SEO有一定要求，所以考虑使用服务端渲染（ssr），采用了vue.js官网推荐的nuxt.js框架进行开发。nuxt 项目和单页应用（spa）项目除了开发上有一些区别之外，部署上线也相对复杂一些，本文对部署上线流程做一下简单的记录。


#### 系统环境

服务器系统版本：CentOS Linux release 7.6.1810 (Core)


## 安装nodejs (由于yum可以安装的nodejs版本较老，这里我们自己下载安装)

### 1. 下载

```bash
wget https://cdn.npm.taobao.org/dist/node/v14.16.0/node-v14.16.0-linux-x64.tar.xz -P /opt/
```

### 2. 解压tar包

进入第一步下载到的文件夹，

```bash
cd /opt
```

解压

```bash
tar xf node-v14.16.0-linux-x64.tar.xz
```

### 3. 配置环境变量

#### 3.1 配置软连接（方便使用）

```bash
ln -s node-v14.16.0-linux-x64 node
```

#### 3.2 配置环境变量

```bash
echo "PATH=/opt/node/bin/:$PATH" >> /etc/profile
```

### 4. 使配置文件生效

```bash
source /etc/profile
```

### 5. 测试，在任意目录下执行 `node -v`，能打印出版本号就说明安装配置成功

```bash
node -v
```


## 安装yarn

由于我的项目使用yarn管理项目依赖，所以需要安装yarn，如果你使用npm或者其他包管理工具，可以跳过此步骤不必安装，**注意：如果使用npm，后面打包，安装依赖和运行项目要替换成对应的命令**。

### 1. 下载

```bash
curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
```

### 2. 安装（yum安装会自动配置好环境变量）

```bash
sudo yum install yarn
```

### 3. 测试（同样打印出版本号就说明安装成功）

```bash
yarn -v
```


## 打包部署

### 1. 打包

```bash
yarn build
```

### 2. 部署，打包完成之后把下列文件和文件夹上传到服务器(我是放在/var/www/)

- .nuxt/ 文件夹
- static/ 文件夹
- package.json
- nuxt.config.js

### 3. 安装项目依赖

首先，进入到上传到的服务器目录，然后执行安装命令

```bash
yarn
```

### 4. 运行项目

```bash
yarn start
```

项目默认运行在`127.0.0.1:3000`（如果开发过程中修改了端口配置，访问时也要修改），此时在服务器本地可以访问项目页面了。但是外部还不能通过网络访问，下面我们通过nginx做一个反向代理让我们的网页可以通过网络访问。


## 使用nginx做反向代理

### 1. 安装

```bash
yum install nginx
```

### 2. 测试

```bash
nginx -v
```

### 3. 启动nginx

```bash
nginx
```

启动之后就可以使用服务器ip访问nginx服务器欢迎页面了，默认启动在80端口，如果无法访问欢迎页面，请检查服务安全规则或者防火墙配置，看相应端口是否开放。

### 4. 修改nginx配置，反向代理到nuxt项目页面

修改nginx.conf文件(按上述安装方式，该文件在/etc/nginx/目录下)。在 server 中添加如下配置：


```
location / {
    expires 1h;

    proxy_redirect                      off;
    proxy_set_header Host               $host;
    proxy_set_header X-Real-IP          $remote_addr;
    proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto  $scheme;
    proxy_read_timeout          1m;
    proxy_connect_timeout       1m;
    proxy_pass                          http://127.0.0.1:3000; # nuxt项目运行的地址
}
```

### 5. 测试配置是否正确

```bash
nginx -t
```

### 6. 重启nginx

```bash
nginx -s reload
```

重启之后就可以通过服务器IP访问到nuxt项目网页了。例如：http://56.121.30.89，默认是80端口。如果修改了端口号，需要加上端口号访问。例如：http://56.121.30.89:81。
**注意：此时nuxt项目要在运行中才能访问，如果上面打包部署完运行项目把nuxt项目停止运行了，要重新运行起来。**

到现在虽然我们可以访问nuxt开发的项目了，但是如果我们关掉执行运行nuxt项目命令的窗口，那么nuxt项目就会停止运行，这样我们就没法访问了。所以下面我们要安装一个守护进程管理器——pm2（PM2 是一个守护进程管理器，它将帮助您管理和保持您的应用程序在线）。


## 使用PM2守护程序

### 安装pm2

```bash
yarn global add pm2
```

### 使用PM2守护nuxt项目程序

进入nuxt项目部署目录，执行以下命令（project-name请替换成具体项目名称，意为指定进程名称）：

```bash
pm2 start npm --name 'project-name' -- run start
```

查看进程，输入一下命令输出如下：

```bash
pm2 list
```

┌─────┬────────────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name               │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼────────────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ project-name       │ default     │ N/A     │ fork    │ 6007     │ 54m    │ 0    │ online    │ 0%       │ 29.9mb   │ root     │ disabled │
└─────┴────────────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘

此时我们可以看到，我们的项目已经在运行了。更过pm2命令，请参考[pm2文档](https://pm2.keymetrics.io/docs/usage/quick-start/)。

这样我们就完成了整个项目部署上线。


## 参考资料：

- [nuxt项目上线](https://www.cnblogs.com/zezhou/p/14432439.html)
- [yarn安装](https://yarn.bootcss.com/docs/install/#centos-stable)
- [nuxt中文官网常见问题——nginx 代理](https://www.nuxtjs.cn/faq/nginx-proxy)
- [pm2文档](https://pm2.keymetrics.io/docs/usage/quick-start/)
- [next.js、nuxt.js等服务端渲染框架构建的项目部署到服务器，并用PM2守护程序](https://segmentfault.com/a/1190000012774650)