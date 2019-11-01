## Git 踩坑记录：从零开始搭建Git（一）
本文旨在搭建一个独立的，私有的，可供一个小型团队协作开发的 Git 仓库。
话不多说，但是希望阅读前你已经安装了 Git 并且会初步的使用 Git 基本命令。  

```shell
#添加一个新的用户
adduser git
#使用 git 用户进行操作
su git
```
```shell
#配置 git 用户名及邮箱
git config --global user.name 'your name'
git config --global user.email 'your email address'
 
#初始化空的 Git 仓库
mkdir -p /home/git/MyProject
cd /home/git/MyProject
git init
#克隆一个裸仓库
cd ..
git clone --bare MyProject MyProject.git
```
git init 之后，对该服务器有 SSH 访问权限的用户已经可以通过地址来访问 git 并拉取代码。我们克隆一个裸仓库的必要性在于创建一个 git 目录数据的副本，我们对外提供的是这个副本，这将在我们使用 hook 时为我们提供便利。

```shell
#添加远程仓库
git remote add origin MyProject.git
```
至此已经能够实现代码的拉取与推送，接下来为项目成员开放服务器的 ssh 权限。

---
首先每位成员都要在自己的本地安装 git ，然后同上配置 git 用户名及邮箱。  

服务器端：
```shell
#创建文件存放授权用户的公钥
mkdir -p /home/git/.ssh
touch /home/git/.ssh/authorized_keys
#设置权限控制
chmod 700 /home/git/.ssh/authoized_keys
```
客户端：
```shell
#生成 ssh key 公钥
ssh-keygen -t rsa -C "your email address" -b 2048
```
-t 指定密钥类型，默认是 rsa(SSH-2)  
-C 指定一个注释，比如邮箱  
-b 指定密钥长度，RSA密钥最小要求768位，默认是2048位
>Tips: 我最初使用的是没有 -b 参数的，后来项目上有个小伙伴使用同样的命令生成的 ssh key 比其他人的长了好多，而且认证不通过，猜测是位数的原因，后来指定了密钥长度为 2048 后就正常了。

生成公钥的命令会提示三次  
Enter file in which to save the key 输入你的保存文件名  
Enter passphrase 输入你的密码  
Enter same passphrase again 再次输入你的密码  
现在基本上大家都会忽略这些了，我们也选择连按三次 Enter 回车跳过。  
随后会提示  
Your identification has been saved in /c/Users/xx/.ssh/id_rsa  
Your public key has been saved in /c/Users/xx/.ssh/id_rsa.pub  
那就分别是你的私钥和公钥啦，私钥保存好，把公钥 id_rsa.pub 文件里的内容全部复制，写入服务器端 /home/git/.ssh/authorized_keys 文件就可以了。

```shell
#客户端拉取代码
git clone 192.168.xxx.xxx:/home/git/MyProject.git
```
至此就可以开发了，下次记录一下我利用 hook 实现一个简单的自动发布的过程。