昨天晚上有个小伙伴因为不会 linux 被一个文件及文件夹的权限问题困扰了很久，我兴致勃勃的说要帮他解决，然而学艺不精，手放到键盘上突然啥都忘了 :-( ，最后还是百度了一下瞬间就想起来了，于是今天整理一下比较常用的点，强化记忆。

---
查看文件详情：
```
[root@test]# ll
drwxr-xr-x 2 root root 4096 Jun  3 16:49 test
```
`drwxr-xr-x` 包含的信息：
- `d` : 第 1 个字符表示文件类型：
  - `-` ：普通文件
  - `d` ：目录文件
  - `l` ：链接文件
  - `s` ：套接字文件
  - `p` ：管道文件
  - `c` ：字符文件
  - `b` ：块文件
- `rwxr-xr-x` : 为三个三个一组（ rwx 为一组）的权限组控制（详见如下列表），其中 `r` 位为是否有读权限、`w` 位为是否有写权限、`x` 位为是否有执行权限（ `x` 位还有一些特殊权限，因不常用暂时略过）、如果某一位上为 `-` 则表示没有对应的权限。
  - 前 3 位的 rwx 表示该文件的 owner 权限。
  - 中间 3 位的 rwx 表示该文件的 group 的权限。
  - 最后 3 位的 rwx 表示其他人对该文件的权限。

所以 `drwxr-xr-x` 代表的就是 **test** 是一个目录：
- 它的所有者具有读、写和执行权限。
- 和它的所有者同一用户组的用户具有读和执行权限，没有写权限。
- 其他用户具有读和执行权限，没有写权限。

---
通过 `chmod` 命令可以修改权限：
```
[root@test]# ll
drwxr-xr-x 2 root root 4096 Jun  3 16:49 test
[root@test]# chmod -rwx test
[root@test]# ll
d---r-xr-x 2 root root 4096 Jun  3 16:49 test
[root@test]# chmod +rw test
[root@test]# ll
drw-r-xr-x 2 root root 4096 Jun  3 16:49 test
```
可以看到，我们可以很直观的使用 `+` 和 `-` 来增加或取消指定的权限。但是默认情况下我们修改的总是文件 owner 的权限，想要修改文件 group 或者其他用户的权限需要在 `+` 或 `-` 之前加上相应的参数：
  - `u` ：（user）该文件的 owner 权限。
  - `g` ：（group）该文件的 group 的权限。
  - `o` ：（other）其他人对该文件的权限。

举栗🌰 ：取消其他用户对 **test** 目录的读和执行权限：
```
[root@test]# ll
drw-r-xr-x 2 root root 4096 Jun  3 16:49 test
[root@test]# chmod o-rx test
[root@test]# ll
drw-r-x--- 2 root root 4096 Jun  3 16:49 test
```
但是这样每次只能改一种权限，无法同时给不同的权限组分配不同的权限。还有一种更加方便更常用的方法。每一组的 r、w、x 由一位 8 进制数来表示，比如 `r-x` -> `101` -> `5`，那么上述的 `drw-r-x---` 就可以用 `650` 来表示。

另外递归的修改目录下包含的所有文件及目录也经常用到，使用 `-R ` 参数实现，其他参数可以通过 `chmod --help` 查看。

举栗🌰 ：
```
[root@test]# ll
drw-r-x--- 2 root root 4096 Jun  3 16:49 test
[root@test]# ll test/
-rw-r--r-- 1 root root   4 Jun  3 16:49 file_1
-rw-r--r-- 1 root root 139 Jun  3 20:02 file_2
[root@test]# chmod -R 755 test
[root@test]# ll
drwxr-xr-x 2 root root 4096 Jun  3 16:49 test
[root@test]# ll test/
drwxr-xr-x 1 root root   4 Jun  3 16:49 file_1
drwxr-xr-x 1 root root 139 Jun  3 20:02 file_2
```
---
当我们作为 root 用户时，更多的时候要下发文件的权限，这时就要更改文件的所有者或者组来让各个用户或组单独控制它所管控的文件，这非常简单。

通过 `chown` 和 `chgrp` 修改文件的所有者和组：
```
[root@test]# ll
drwxr-xr-x 2 root root 4096 Jun  3 20:15 test
[root@test]# chown jehu test
[root@test]# ll
drwxr-xr-x 2 jehu root 4096 Jun  3 20:15 test
[root@test]# chgrp jehu test
[root@test]# ll
drwxr-xr-x 2 jehu jehu 4096 Jun  3 20:15 test
```
还有简便方法同时修改用户和组：
```
[root@test]# ll
drwxr-xr-x 2 jehu jehu 4096 Jun  3 20:15 test
[root@test]# chown root:root test
[root@test]# ll
drwxr-xr-x 2 root root 4096 Jun  3 20:15 test
```
递归：
```
[root@test]# ll
drwxr-xr-x 2 root root 4096 Jun  3 16:49 test
[root@test]# ll test/
drwxr-xr-x 1 root root   4 Jun  3 16:49 file_1
drwxr-xr-x 1 root root 139 Jun  3 20:02 file_2
[root@test]# chown -R jehu:jehu test
[root@test]# ll
drwxr-xr-x 2 jehu jehu 4096 Jun  3 20:15 test
[root@test]# ll test/
drwxr-xr-x 1 jehu jehu   4 Jun  3 16:49 file_1
drwxr-xr-x 1 jehu jehu 139 Jun  3 20:02 file_2
```