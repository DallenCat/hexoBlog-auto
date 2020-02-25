
---

title: 二、Travis-CI监听github仓库变动部署到云服务器

urlname: scag83

date: 2020-02-22 17:32:08 +0800

tags: []

---
<a name="r6fjB"></a>
## 前言
上一篇博客中主要讲了本地如何将hexo部署到云服务器上，但是我们并不希望在本地写一个md文件然后部署一次服务器，如何去自动部署？<br />答案是Travis-CI
<a name="e7Jlq"></a>
## 思路
1.在我们github仓库中新建一个公有仓库<br />2.每当github中有文件发生变动，让travis-ci去执行我们的命令<br />3.我们让travis-ci去云服务器上拉取一次github仓库中的文件，然后执行一次hexo deploy<br />4.这样就做到了自动部署<br /><!--more-->

<a name="v40bp"></a>
## 一、配置多个SSH

- **新建一个用户**

```javascript
#新建用户
useradd travis
passwd ***** //自己设置密码
#为用户添加添加权限
vim /etc/sudoers
```

找到`#Allow root to run any commands anywhere`这一段注释，在下面新增一行：

```javascript
travis  ALL=(ALL)   ALL
```

- **在服务器上生成ssh密钥**

一定要切换到travis用户下，由于之前生成过ssh密钥，现在不能覆盖之前的，只需要在执行生成命令时注意一下就好了。

```javascript
su travis  #切换到travis用户
cd ~    #进入/home/travis目录
ssh-keygen -t rsa -C "github邮箱"
```

当出现`Enter file in which to save the key (/c/.ssh/id_rsa):` 时，输入id_rsa_blog，然后一路回车，在**/home/travis/.ssh**下会生成

![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582365829976-a686dd3f-26c5-4e4f-a922-a40470d02a54.png#align=left&display=inline&height=185&name=image.png&originHeight=185&originWidth=612&size=22731&status=done&style=none&width=612)<br />由于Linux权限的控制规则，文件的权限不是越大越好，所有需要设置合适的权限。这里需要把**.ssh目录设置为700权限，给.sshm=目录下面的文件设置为600权限。

```javascript
#设置.ssh目录为700
[travis@VM_156_69_centos ~]$ chmod 700 ~/.ssh/
#设置.ssh目录下的文件为600
[travis@VM_156_69_centos ~]$ chmod 600 ~/.ssh/*
#可以看到下面的所有目录和文件所用者都是travis这个用户
[travis@VM_156_69_centos ~]$ ls -al
total 28
drwx------  3 travis travis 4096 Mar  6 20:12 .
drwxr-xr-x. 5 root   root   4096 Mar  6 20:03 ..
drwx------  2 travis travis 4096 Mar  6 20:12 .ssh
[travis@VM_156_69_centos ~]$ ls ~/.ssh/ -al
total 16
drwx------ 2 travis travis 4096 Mar  6 20:12 .
drwx------ 3 travis travis 4096 Mar  6 20:12 ..
-rw------- 1 travis travis 1675 Mar  6 20:12 id_rsa
-rw------- 1 travis travis  405 Mar  6 20:12 id_rsa.pub
```

- **将生成的公钥添加为受信列表**

```javascript
[travis@VM_156_69_centos ~]$ cd .ssh/
#将公钥内容输出到authorized_keys中
[travis@VM_156_69_centos .ssh]$ cat id_rsa.pub >> authorized_keys
[travis@VM_156_69_centos .ssh]$ cat authorized_keys 
# authorized_keys文件内容类似这样
ssh-rsa  *************centos
```

- **修改config配置**

```javascript
#在.ssh目录下新增配置文件config,修改完esc :wq保存
[travis@VM_156_69_centos .ssh]$ touch config
[travis@VM_156_69_centos .ssh]$ vim config
```

`config` :
```javascript
Host test
HostName 你的服务器ip
#登陆的用户名
User travis
IdentitiesOnly yes
#登陆使用的密钥
IdentityFile ~/.ssh/id_rsa_blog
```

测试ssh连接

```javascript
#测试连接
[travis@VM_156_69_centos .ssh]$ ssh test
Bad owner or permissions on /home/travis/.ssh/config
#注意此时的测试是失败的，因为authorized_keys和config是我们后面添加的文件，文件权限并不是600
#修改文件权限
[travis@VM_156_69_centos .ssh]$ chmod 600 config 
[travis@VM_156_69_centos .ssh]$ chmod 600 authorized_keys
#重新执行测试
[travis@VM_156_69_centos .ssh]$ ssh test 
The authenticity of host '139.199.90.74 (139.199.90.74)' can't be established.
ECDSA key fingerprint is 41:39:50:e1:e7:c2:f5:19:86:dc:70:e5:91:42:bb:56.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '139.199.90.74' (ECDSA) to the list of known hosts.
Last login: Tue Mar  6 20:43:32 2018 from 139.199.90.74
#测试成功，生成了一个known_hosts文件，以后再登陆时就不需要在输入yes确认了，你可以再做一次测试
```

`.ssh`下多了一个 `known_hosts` 文件<br />

<a name="KIYrO"></a>
## 二、将ssh添加到github中

- 登录到github，鼠标放到右上角头像上，点击`setting`，再点击`SSH and GPG keys`，再点击 `New SSH key`，把生成的密钥id_rsa_blog.pub里的内容复制到key文本框中，最后点击**Add SSH key**

![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582364810600-a218cf38-5c03-482e-b6d9-066ed7f4c881.png#align=left&display=inline&height=462&name=image.png&originHeight=462&originWidth=303&size=30611&status=done&style=none&width=303)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582365202573-d4aa9fc9-b1f9-4912-a97f-320f3f6d7b0c.png#align=left&display=inline&height=548&name=image.png&originHeight=548&originWidth=1060&size=60738&status=done&style=none&width=1060)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582365316145-2cc2d239-b836-406f-8700-2a121bc2fc5e.png#align=left&display=inline&height=424&name=image.png&originHeight=424&originWidth=986&size=35105&status=done&style=none&width=986)<br />

- 在仓库中新建一个`hexoBlog-auto`的公有仓库，因为travis-ci.org是对开源库免费的，私有库需要收费
- 在本地拉取`hexoBlog-auto`仓库新建一个文件`.travis.yml`，然后提交。

`.travis.yml`:

```javascript
language: node_js
node_js:
- stable
branchs:
  only:
  - master
addons:
  ssh_known_hosts:
  - 134.175.240.20
```

<a name="tOPQu"></a>
## 三、配置Travis

- 打开[Travis-CI](https://travis-ci.org/)官网，登录你的github账号
- 找到`hexoBlog-auto`，打勾后点击`Settings`，按照下图打勾

![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582366969875-93607bd8-fc13-4305-a7bb-aae26e1e510c.png#align=left&display=inline&height=56&name=image.png&originHeight=56&originWidth=667&size=4088&status=done&style=none&width=667)<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582367057072-0b0d19ce-0460-4e4b-a152-1b8dafbb8c3f.png#align=left&display=inline&height=374&name=image.png&originHeight=374&originWidth=987&size=30245&status=done&style=none&width=987)
<a name="J21qm"></a>
## 四、在云服务器安装Travis客户端工具

- Travis客户端需要用gem安装，gem是ruby的管理工具，所以要先安装ruby，这里用ruby版本管理工具**rvm**（类似于nodejs与npm关系）

- **安装rvm**

```javascript

[travis@VM_156_69_centos ~]$ curl -sSL https://get.rvm.io | bash -s stable
#安装完成后测试是否安装成功
[root@VM_156_69_centos ~]$ rvm version
rvm 1.29.3 (master) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```

- **安装ruby**

```javascript
#我用travis用户安装时好像有网络错误，所以就用root用户安装
[travis@VM_156_69_centos root] su #
[root@VM_156_69_centos ~]# rvm install ruby
#测试ruby安装
[root@VM_156_69_centos ~]$ su travis
[travis@VM_156_69_centos root]$ ruby --version
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-linux]
```

- **修改镜像源**

安装完ruby之后就可以使用gem包管理工具了，但是官方镜像源被墙了，所以需要更换gem的镜像源

```javascript

[travis@VM_156_69_centos ~]$ gem sources -l
*** CURRENT SOURCES ***
 
https://rubygems.org/
[travis@VM_156_69_centos ~]$ gem -v
2.6.14
#更换镜像源
[travis@VM_156_69_centos ~]$ $ gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
https://gems.ruby-china.org/ added to sources
https://rubygems.org/ removed from sources
[travis@VM_156_69_centos ~]$ gem sources -l
*** CURRENT SOURCES ***
 
https://gems.ruby-china.com/
```

- **安装travis命令行工具**

```javascript
#在travis下面提示没有权限，我切到root用户去安装
[travis@VM_156_69_centos ~]$ gem install travis
Fetching: multipart-post-2.0.0.gem (100%)
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You dont have write permissions for the /usr/local/rvm/gems/ruby-2.4.1 directory.
#安装travis
[root@VM_156_69_centos ~]# gem install travis
Successfully installed travis-1.8.8
Parsing documentation for travis-1.8.8
Done installing documentation for travis after 1 seconds
1 gem installed
#切回travis用户，执行travis命令有以下输出说明安装成功
[travis@VM_156_69_centos root]$ travis
Shell completion not installed. Would you like to install it now? |y| y
Usage: travis COMMAND ...
```

<a name="nfBWp"></a>
## 五、加密ssh私钥至.travis.yml中

- **从github下拉取代码仓库**

```javascript
[travis@VM_156_69_centos ~]$ git clone git@github.com:你的github名称/hexoBlog-auto.git
```

- **在****hexoBlog-auto下****新建一个.travis文件夹，复制ssh私钥到.travis文件夹下**

```javascript
[travis@VM_156_69_centos hexoBlog-auto]$ mkdir .travis
[travis@VM_156_69_centos hexoBlog-auto]$ cp ~/.ssh/id_rsa_blog .travis/
```

- **ssh配置文件**

这个ssh配置文件是用于travis-ci部署使用的，不是本地ssh配置文件，后面travis配置文件会用到<br />

```javascript
[travis@VM_156_69_centos ~]$ touch .travis/ssh_config
[travis@VM_156_69_centos ~]$ vim .travis/ssh_config

Host test
HostName 134.175.240.20
#登陆的用户名
User travis
IdentitiesOnly yes
#登陆使用的密钥
IdentityFile ~/.travis/id_rsa_blog

esc :wq保存
```

- **登录travis**

**
```javascript

[travis@VM_156_69_centos ~]$ cd hexoBlog-auto
[travis@VM_156_69_centos hexoBlog-auto]$ travis login
We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.
 
Try running with --github-token or --auto if you dont want to enter your password anyway.
 
Username: 你的github账号
Password for github账号: ******
Successfully logged in as github账号!
  
[travis@VM_156_69_centos hexoBlog-auto]$ travis encrypt-file .travis/id_rsa_blog --add
```

在**hexoBlog-auto**目录下会生成  **id_rsa_blog.enc** 文件，把 **id_rsa_blog.enc **移动到 **.travis **文件夹下，删除**id_rsa_blog密钥文件（id_rsa_blog.enc是我们加密后的密钥文件，给travis服务器解析用的）**<br />同时你的.travis.yml文件会出现这样的代码：
```javascript
before_install:
- openssl aes-256-cbc -K $encrypted_f61dd4bb83d2_key -iv $encrypted_f61dd4bb83d2_iv
  -in .travis/id_rsa_blog.enc -out ~/.ssh/id_rsa -d
```
说明我们添加成功
<a name="fd6we"></a>
## 六、编辑配置文件.travis.yml

- `.travis.yml`:

```javascript
language: node_js
node_js:
- stable
branchs:
  only:
  - master
install:
- npm install
- npm install --save hexo-deployer-git #hexo提交依赖
- npm install --save hexo-generator-json-content  #yilia主题的依赖，不需要可以不用下载
addons:
  ssh_known_hosts:
  - 99.99.99.99 #你的服务器ip
before_install:
- openssl aes-256-cbc -K $encrypted_f61dd4bb83d2_key -iv $encrypted_f61dd4bb83d2_iv
  -in .travis/id_rsa_blog.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
- cp .travis/ssh_config ~/.ssh/config
- git config --global user.name "xxxxx" #设置github用户名
- git config --global user.email xxxxx@xxx.com #设置github用户邮箱
# 执行的命令
after_success:
- ssh travis@99.99.99.99 -o StrictHostKeyChecking=no 'cd ~/hexoBlog-auto  && git pull && npm run deploy'

script:
- npm run deploy

```

- 编辑 **hexoBlog-auto** `package.json`

```javascript
"scripts": {
        "build": "hexo generate",
        "clean": "hexo clean",
        "server": "hexo server",
        "deploy": "hexo clean && hexo g -d"
},
```

- 保存并提交新增、修改文件

```javascript
[travis@VM_156_69_centos hexoBlog-auto]$ git add .travis package.json .travis.yml
[travis@VM_156_69_centos hexoBlog-auto]$ git commit -m 'update'
[travis@VM_156_69_centos hexoBlog-auto]$ git push
```

<a name="AoRGR"></a>
## 七、在travis官网中查看部署动态
你可能会遇到的坑：

- No such file or directory:bss_file.c:398:fopen(‘.ssh/id_rsa’,’w’)

这是密钥解析出现问题，重新返回步骤五

正常这一步会提示没有权限，因为在上一篇文章中，我们发布到服务器用的**git**用户，**travis**用户无权操作 `/home/git` 下的文件

修改`/home/git` 文件所有者

```javascript
[travis@VM_156_69_centos hexoBlog-auto]$ su git             //切换到git下
[git@VM_156_69_centos hexoBlog-auto]$ cd ~
[git@VM_156_69_centos ~]$ cd ..
[git@VM_156_69_centos home]$ chown -R travis git/              //-R递归修改git下的所有文件
[git@VM_156_69_centos home]$ su #                        //切换到root
[root@VM_156_69_centos home]$ chown -R travis hexoBlog/     //-R递归修改hexoBlog下的所有文件
```

然后restart一次就可以了

![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582374870108-22856d6f-be8f-4a5e-adb6-8d896fcdbeda.png#align=left&display=inline&height=226&name=image.png&originHeight=226&originWidth=1516&size=26027&status=done&style=none&width=1516)


![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582374913569-8a1eb7db-ead0-4d97-b255-cb25fa730fb8.png#align=left&display=inline&height=104&name=image.png&originHeight=104&originWidth=886&size=8340&status=done&style=none&width=886)


<a name="CnoQH"></a>
## hexo同步语雀，语雀发布自动部署在下一章博客中

