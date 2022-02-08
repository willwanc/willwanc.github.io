---
title: 多个git账户配置ssh keys
date: 2020-08-27 20:58:42
tags: ['github', 'gitlib', 'git', 'ssh keys']
categories: ['git']
---

## 背景

最近发现公司的gitlab代码仓库里提交记录是个人github的用户名，知道自己又忘了给公司的gitlab账户配置单独的用户名和邮箱了。使用过git的应该都知道：我们可以在本地生成ssh keys，然后在github/gitlab添加生成公钥，就可以不用git用户名和密码直接提交代码。如果在一台电脑上只配置一个git账户的ssh keys，我们不需要给公、私钥文件命名（直接用默认的就行），但是对大多数开发者来说，都会向gitlab（公司代码服务器）和github、gitee等个人代码仓库等不同账户提交代码，而我们的公司git账户名和个人git账户名往往是不同的，这时候我们就要区分公司git账户和个人git账户，根据不同账户的邮箱生成不同的ssh keys。这么说如果你没有配置多账户的ssh keys的话，可能还是一头雾水。不要着急，往下看。。。

## 第一步：生成ssh keys

以我自己为例，ssh给gitlab(公司)和github（个人）生成不同的ssh keys。

首先，进到`~/.ssh/`目录：
```bash
$ cd ~/.ssh/
$
```

然后，我们就可以在这个目录下生成github的公私钥文件，输入如下命令，然后按回车，

```bash
$ ssh-keygen.exe -t rsa -C "你的邮箱"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/wanchong/.ssh/id_rsa): id_rsa_github
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your public key has been saved in id_rsa_github.pub.
The key fingerprint is:
SHA256:a7bS18Gmmzq0xtOzYw4j/i11AQIga8ixVIhkILdWJvY wilson3cy@126.com
The key's randomart image is:
+---[RSA 2048]----+
|*=B.+...         |
|*=oO    . .      |
|.o= E    . .     |
| o          .    |
|        S  . .   |
|        ... =    |
|      .+*+ = .   |
|     ..+O=B..    |
|      .++O*=     |
+----[SHA256]-----+
```

然后会依次提示你输入文件名，密码和确认密码，文件名可以自己随意命名，密码建议不用设置，如果设置，每次提交代码都需要输入会很麻烦。输入完文件名之后，一直按回车，就会输出如上内容，说明成功生成了公私钥文件。

然后，我们可以用`ls`查看一下

```bash
$ ls
id_rsa_github  id_rsa_github.pub  known_hosts
```

id_rsa_github 和 id_rsa_github.pub 这两个文件就是生成的默认的私钥文件和公钥文件。接下来，我们按照同样的步骤就可以生成gitlab的公私钥文件。最终生成两组公私钥文件如下：

```bash
$ ls
id_rsa_github  id_rsa_github.pub id_rsa_gitlab  id_rsa_gitlab.pub   known_hosts
```

## 第二步：添加ssh keys的配置文件

我们先登上github账户，然后依次点击：个人头像、settings、SSH and GPG keys；然后点击new SSH key 按钮，输入title(自己命名)，打开id_rsa_github.pub文件，复制全部内容，粘贴到key对应的文本输入框里，添加到ssh keys 里，点击 Add SSH key保存即可。gitlab按同样的步骤操作即可。


## 第三步 给不同账户的代码仓库配置不同的用户名和邮箱

现在我们生成了不同的公私钥文件，并且把他添加到了对应账户的ssh key里了。但是我们提交代码的时候，我们的git程序怎么知道提交代码到gitlab和github账户分别用哪个对应的私钥文件呢，显然它是不知道的。为了让它知道，我们需要在`~/.ssh`目录下添加一个config文件。

创建config文件：

```bash
wanchong@DESKTOP-2GKTN2J MINGW64 ~/.ssh
$ touch config
```

然后输入如下内容：
```
Host github
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_github
Host gitlab
    HostName gitlab.com
    IdentityFile ~/.ssh/id_rsa_gitlab
```
在这个文件中，可以设置多个代码服务器主机的配置。每个用户配置可以包含以下几个配置项：

Host：代码仓库服务器主机的别名
Port：端口号。默认为22，可不配置
User：代码仓库网站的用户名
HostName：真正的代码仓库服务器地址
PreferredAuthentications：指定优先使用哪种认证方式，支持密码和秘钥验证方式
IdentityFile：私钥文件的绝对路径

## 第四步：测试连接

配置完成以后，我们测试一下配置是否正确。输出如下结果，就说明配置正确，成功连接到github了。

```
$ ssh -T git@github.com
Hi willwan92! You've successfully authenticated, but GitHub does not provide shell access.
```

gitlab和github测试同样。


## 解决第四步的麻烦
