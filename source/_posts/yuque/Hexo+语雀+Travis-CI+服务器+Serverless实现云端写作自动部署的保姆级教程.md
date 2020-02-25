
---

title: Hexo+语雀+Travis-CI+服务器+Serverless实现云端写作自动部署的保姆级教程

urlname: lhbp8s

date: 2020-02-19 18:02:27 +0800

tags: []

---
<a name="JjSNN"></a>
### 前序
你需要准备的：nodejs安装、云服务器、语雀账号、github账号
<a name="bhiw7"></a>
### 步骤

1. hexo博客部署到云服务器
1. Travis-CI监听github仓库变动部署到云服务器
1. 语雀发布文章触发serverless发起请求trigger一个build任务

<!--more-->

<a name="RYsaB"></a>
### 参考链接
[手把手教从零开始在GitHub上使用Hexo搭建博客教程(三)-使用Travis自动部署Hexo(1)](https://www.cnblogs.com/seayxu/archive/2016/06/02/5553229.html)<br />[Travis-CI自动化测试并部署至自己的CentOS服务器](https://blog.csdn.net/weixin_34088838/article/details/91367043)<br />[利用 Travis CI 自动部署 Hexo 博客](https://blog.yusanshi.com/2019-11-24-travis-deploy/)<br />[Hexo 博客终极玩法：云端写作，自动部署](https://segmentfault.com/a/1190000017797561?utm_source=tag-newest)

<a name="Jfo3J"></a>
### 我的博客

[一、hexo安装部署到云服务器](http://www.daiailing.cn/2020/02/19/yuque/Hexo+%E8%AF%AD%E9%9B%80+Travis-CI+%E6%9C%8D%E5%8A%A1%E5%99%A8+Serverless%E5%AE%9E%E7%8E%B0%E4%BA%91%E7%AB%AF%E5%86%99%E4%BD%9C%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E7%9A%84%E4%BF%9D%E5%A7%86%E7%BA%A7%E6%95%99%E7%A8%8B/)<br />[二、Travis-CI监听github仓库变动部署到云服务器](http://www.daiailing.cn/2020/02/22/yuque/Travis-CI%E7%9B%91%E5%90%ACgithub%E4%BB%93%E5%BA%93%E5%8F%98%E5%8A%A8%E9%83%A8%E7%BD%B2%E5%88%B0%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8/)<br />[三、语雀写作，自动部署服务器](http://www.daiailing.cn/2020/02/22/yuque/%E8%AF%AD%E9%9B%80%E5%86%99%E4%BD%9C%EF%BC%8C%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E6%9C%8D%E5%8A%A1%E5%99%A8/)

<a name="Yhbe4"></a>
### 一、hexo安装部署到云服务器
<a name="RvqCX"></a>
#### 1.配置node环境
去[node官网](https://nodejs.org/zh-cn/download/)下载符合你本地操作系统的安装包，安装到<目录,例如我安装到E:/Nodejs/>后打开cmd(win+R输入cmd)

```javascript
node -v
npm -v
```
能查看到说明安装成功
<a name="PUGx6"></a>
#### 2.配置全局环境
进入安装目录，创建文件夹node_global和node_cache并执行

```javascript
npm config set prefix "E:\Nodejs\node_global"
npm config set cache "E:\Nodejs\node_cache"
```

环境配置：新增系统变量 `Path` 和环境变量 `Path`，值都为：`E:\Nodejs\node_global`

环境变量：<br />![环境变量.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582357310201-af7ba9d1-43dc-4854-ad3e-feecc30e9c8e.png#align=left&display=inline&height=263&name=%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F.png&originHeight=263&originWidth=532&size=18520&status=done&style=none&width=532)

系统变量：<br />![系统变量.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582357358933-3ee9369f-eb71-4d79-b7d4-49fd1d9cd1c8.png#align=left&display=inline&height=555&name=%E7%B3%BB%E7%BB%9F%E5%8F%98%E9%87%8F.png&originHeight=555&originWidth=1080&size=123695&status=done&style=none&width=1080)

<a name="GclS1"></a>
#### 3.安装`hexo-cli`

```javascript
npm i hexo-cli -g
hexo
```

![hexo.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/933518/1582357507609-832e046c-2fe7-4c95-8d7e-30026d374ca9.jpeg#align=left&display=inline&height=429&name=hexo.jpg&originHeight=429&originWidth=651&size=16834&status=done&style=none&width=651)

出现这样的提示说明hexo安装成功。如果提示命令未找到，或不是可执行程序，请重复步骤2操作，说明环境配置有问题。

<a name="5aKnR"></a>
#### 4.初始化hexo
新建一个文件夹，这里叫hexoBlog-auto，在该文件夹目录下输入

```javascript
hexo init
```

hexo init会去clone github上的hexo项目，初始化成功之后可以去下载你喜欢的主题，我用的是yilia主题，大部分人用的比较多的是next主题，或者去官网搜索更多主题

```javascript
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

如果下载了主题，在`_config.yml`中`theme`字段修改成你安装的主题，`_config.yml`中具体的参数会在其他文章里讲。<br />![主题.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582358014630-e6cd6f3d-dd28-45d2-8093-177aa6a37849.png#align=left&display=inline&height=137&name=%E4%B8%BB%E9%A2%98.png&originHeight=137&originWidth=447&size=12935&status=done&style=none&width=447)
<a name="Mvc1w"></a>
#### 5.运行hexo
在hexo目录cmd中输入

```javascript
hexo clean
hexo g
hexo s
```
hexo clean 是清除之前生成过的静态资源文件<br />hexo g是hexo generate的缩写，将source文件下的md生成静态资源html<br />hexo s是hexo serve的缩写，启动hexo服务器

![运行.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582358264451-264fce62-5bbc-46f7-ba2e-21dd7842cc77.png#align=left&display=inline&height=72&name=%E8%BF%90%E8%A1%8C.png&originHeight=72&originWidth=594&size=9146&status=done&style=none&width=594)

运行成功后可以打开 [http://localhost:4000](http://localhost:4000) 查看效果。
<a name="RMxIK"></a>
#### 6.git环境搭建
git安装：[git官网](https://git-scm.com/downloads)<br />生成`ssh`认证

```javascript
git config --global user.name "yourname" //你的github名称
git config --global user.email youremail@example.com //你的github邮箱
ssh-keygen -t rsa -C "youremail@example.com"   //你的github邮箱,生成ssh公钥私钥
git config --global core.autocrlf false  // 禁用自动转换，这个不设置后面上传时会出现警告，如下
```

生成的ssh在`C:\Users\Administrator\.ssh`中

<a name="GFAFv"></a>
#### 7.云服务器配置——搭建远程git仓库
登录到远程服务器，推荐使用Xshell，顺便一起下载Xftp。<br />以Xshell为例，在主机栏里填写你的`公网ip`地址<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582358997286-10796e36-e2f3-415f-8c05-b04e9172b50d.png#align=left&display=inline&height=677&name=image.png&originHeight=677&originWidth=671&size=53069&status=done&style=none&width=671)


![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582359067834-f9aa0927-dbb9-468c-81a5-2ddaba233bbd.png#align=left&display=inline&height=247&name=image.png&originHeight=247&originWidth=1683&size=43526&status=done&style=none&width=1683)

点击连接，然后输入账号密码，购买过云服务器，账号密码会在你的邮箱里，密码忘记了可以重置一次密码

1.安装git

```javascript
git --version // 如无，则安装
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
yum install -y git
```

```javascript
useradd git  //添加另一个叫git的用户，在../home/中会多一个git文件夹
passwd ****** // 设置密码
su git // 这步很重要，不切换用户后面会很麻烦
cd /home/git/      //或者 cd ~ ，~是跳到 ../home/git，当前用户的文件夹下
git init --bare hexoBlog.git // 创建一个裸露的仓库
cd hexoBlog.git/hooks
vi post-receive // 创建 hook 钩子函数，输入了内容如下
```

`post-receive` 内容
```javascript
#!/bin/bash
git --work-tree=/home/hexoBlog --git-dir=/home/git/hexoBlog.git checkout -f
```
然后esc，输入`:wq` 保存并退出编辑模式<br />2.修改权限

```javascript
chmod +x post-receive
exit // 退出到 root 登录，或者 su #
chown -R git:git /home/git/hexoBlog.git // 添加权限
```
3.测试git仓库是否可用，本地找一个空文件夹

```javascript
git clone git@你的服务器ip:/home/git/hexoBlog.git
```
能成功clone说明云服务器git仓库搭建成功

4.建立ssh免密登录服务器

```javascript
ssh-copy-id -i C:/Users/Administrator/.ssh/id_rsa.pub git@你的服务器ip
ssh git@你的服务器ip // 测试能否登录
```

ssh-copy-id把本地的公钥提交到了服务器上，本地ssh到服务器时，用本地的id_rsa私钥去连接公钥，能成功是不需要密码的，如果需要密码说明失败了，请仔细对照步骤。
<a name="yaio2"></a>
#### 8.云服务器配置——搭建nginx服务器
1.下载并安装nginx

```javascript
cd /usr/local/src
wget http://nginx.org/download/nginx-1.15.2.tar.gz
tar xzvf nginx-1.15.2.tar.gz
cd nginx-1.15.2
./configure // 如果后面还想要配置 SSL 协议，就执行后面一句！
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module
make && make install
alias nginx='/usr/local/nginx/sbin/nginx' // 为 nginx 取别名，后面可直接用
```

2.配置nginx文件

先启动是否安装成功

```javascript
nginx // 直接来！浏览器查看 server_ip，默认是 80 端口
```

修改配置文件

```javascript
nginx -s stop // 先停止nginx
cd /usr/local/nginx/conf
vi nginx.conf
修改 root 解析路径，如下图,修改完esc :wq保存退出
nginx -s reload //重启
```

在`conf`文件中找到`server_name`，设置成自己的公网ip ，`root` 设置成 `/home/hexoBlog`<br />nginx将根目录指定到/home/hexoBlog下，如果你不知道为什么是/home/hexoBlog，请重新看在上面`post-receive`中配置过，`git --work-tree=/home/hexoBlog`，我们把生成的静态文件放到了/home/hexoBlog下<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582361586390-7858121d-82e0-4baf-af58-896c10dfc193.png#align=left&display=inline&height=237&name=image.png&originHeight=237&originWidth=468&size=16401&status=done&style=none&width=468)
<a name="y4Xmb"></a>
#### 9.发布hexo
在本地hexo文件中`_config.yml`文件修改`deploy`：

```javascript
deploy:
  type: git
  repo: git@你的服务器ip:/home/git/hexoBlog
  branch: master
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582363017316-9632f6cd-916e-4be4-9a4e-cb0665dbeef2.png#align=left&display=inline&height=113&name=image.png&originHeight=113&originWidth=546&size=12663&status=done&style=none&width=546)

在cmd中输入

```javascript
hexo deploy
```
输入你的服务器ip就能查看刚刚部署的页面啦！如果你有域名，在云服务器控制台配置域名，域名需要备案<br />打开云服务器控制台，找云解析<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582363504298-99af5b74-612a-4860-879d-ae484205264a.png#align=left&display=inline&height=139&name=image.png&originHeight=139&originWidth=1230&size=17030&status=done&style=none&width=1230)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/933518/1582363537546-8abe7431-847b-4975-a32e-9430f7b1468e.png#align=left&display=inline&height=223&name=image.png&originHeight=223&originWidth=973&size=18429&status=done&style=none&width=973)<br />记录值为你的公网ip，然后你就可以输入你的域名查看页面了


<a name="VBZwX"></a>
### Travis-CI监听github仓库变动部署到云服务器放在下一章博客中

