## Git Hook
上文已经介绍了如何初始化一个 Git 仓库   
[Git 踩坑记录（一）：从零开始搭建Git](https://wwyx778.github.io/#/posts/6)  

本文将介绍如何使用 hook 实现自动发布以及对代码管理做一些定制化的配置。
在编写 hook 时使用的是 shell 语言，但是大都是语义化的操作，笔者也是边百度学习边写出来的，如果想要获得流畅的阅读体验，请先熟悉 shell 语言的 **if（判断）** 和  **for（循环）** 语法。
>初始化 git 仓库时会自带一些 hook 样例，在 ./git/hook 目录下，把后缀名 .sample 去掉即可使用。  
[官方例子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E4%BD%BF%E7%94%A8%E5%BC%BA%E5%88%B6%E7%AD%96%E7%95%A5%E7%9A%84%E4%B8%80%E4%B8%AA%E4%BE%8B%E5%AD%90)

### 使用 update 控制更新分支的人员权限
```shell
#新建一个文件存放可更新的人员
cd /home/git
touch GIT_MASTER_ALLOW
vim GIT_MASTER_ALLOW
```
输入用户名:邮箱
zhangsan:zhangsan@xxx.com
```shell
#打开 git 仓库的 update.sample
vim /home/git/MyProject.git/hooks/update.sample
```
可以看到这里 ---Command line之后传入了三个参数，他们分别是  
- refname="$1" 被推送的引用的名字
- oldrev="$2" 推送前分支的修订版本（revision）
- newrev="$3" 用户准备推送的修订版本（revision）

我们在其后增加两个常量
```shell
#git仓库地址
GIT_DIR="/home/git/MyProject.git"
#允许人员列表地址
USERLIST="/home/git/GIT_MASTER_ALLOW"
```
在 Safety check 后添加我们的校验用户的代码
```shell
# check user and email
# 如果被推送的分支为 master 
if [ "$refname" = "refs/heads/master" ]; then
    # 取到即将推送版本的用户名
	name=$(git log -1 --pretty=format:%aN $newrev)
    # 取到即将推送版本的用户邮箱
	email=$(git log -1 --pretty=format:%ae $newrev)
    # 拿取到的用户名和邮箱在允许列表中查询
	check_user=$(cat $USERLIST|grep -c $name:$email)
	if [ "$check_user" = "0" ]; then
        # 查询不到则打印报错信息并 exit 1 中断推送
		echo "$name:$email Permission denied, please check and set user.name&user.email !"
		exit 1
	fi
fi
```
保存后修改文件名 update.sample 为 update 即可。

### 使用 post-receive 实现自动发布
post-receive 这个hook由远程资源库的'git-receive-pack'触发，此时，本地资源库的'git push'已经完成，且所有ref已经更新。  
本文实现自动发布的思路是推送成功后，取到当前的修订版本，然后执行 git pull 拉取最新的代码，取到拉取最新代码后的修订版本，然后将两个修订版本的差异代码复制到工程目录的相同路径下。
```shell
#新建 hook 文件
cd /home/git/MyProject.git/hooks
touch post-receive
chmod 755 post-receive
```
编写脚本
```shell
#!/bin/sh
#cpfun 实现目录层级的复制文件
funCopyFile(){
    # 源目录
    source=$1
    # 目标目录
    des=$2
	if [ ! -e "$source" ]; then
        # git删除操作
		rm -f $des
	else
        # 计算/分隔之后总共有多少列
        fields=`echo ${des}|awk -F '/' '{print NF}'`
        dir_fields=$((${fields}-1))
        # 从第一列截取到倒数第二列
        des_dir=`echo $des|cut -d '/' -f 1-${dir_fields}`
        # 目标目录不存在则新建目录
        if [ ! -d $des ]; then
            mkdir -p $des_dir
        fi
		#echo "cp $source $des"
        cp $source $des
    fi
}

# 指定我的代码检出目录
CODE_DIR=/home/git/MyProject
# 指定我的WEB目录
WEB_DIR=/u01/webapp
# 忽略文件列表
# 此处意义为我希望开发人员可以从版本库中获取一些配置文件，但是我不希望开发人员随意的通过推送代码的方式来修改这些配置文件
GIT_IGNORE="/home/git/GIT_IGNORE"

# --- Command line
unset GIT_DIR
# 进入代码检出目录先获取旧版本
cd ${CODE_DIR}
oldrev=$(git log -1 --pretty=format:%h)
# 防止文件污染，先 clean 仓库
git add . -A
git stash
# 拉取代码并获取新版本
git pull origin master
newrev=$(git log -1 --pretty=format:%h)
# 取到两次版本中所有的 commit
new_commits=$(git rev-list $oldrev..$newrev --reverse)

for rev in ${new_commits[@]}
do
    # 循环每次提交的所修改文件列表
	files_modified=$(git log -1 --name-only --pretty=format:'' $rev)
	for path in ${files_modified[@]}
	do
		# 循环这些文件判断是否在忽略列表中
		update_flag=$(cat ${GIT_IGNORE}|grep -c ${path})
		if [ "$update_flag" = "0" ]; then
			# 不在忽略列表中则把文件覆盖到 WEB 目录
			funCopyFile ${CODE_DIR}/${path} ${WEB_DIR}/${path}
		fi
	done
done
```
至此便实现了一个可以控制人员权限、自动发布时忽略文件的简单自动发布的过程。