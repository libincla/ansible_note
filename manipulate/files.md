# ansible针对文件的操作

## 常用模块
1. copy模块
2. file模块
3. blockinfile模块
4. lineinfile模块
5. find模块
6. replace模块


### copy

> 拷贝文件，copy的作用是将ansible主机的文件拷贝到远程主机

**参数**

1. src  用于指定需要拷贝的文件或者目录
2. dest 指定文件要拷贝到远程主机的哪个目录中， *必须参数*
3. content 当不指定src拷贝文件时，可以直接使用`content`直接指定文件内容，`src`和`content`两个参数必备其一
4. force 是否强制覆盖，可选指为yes或者no, 默认是yes，会强制执行覆盖拷贝操作
5. backup 当远程主机目标路径中已经存在同名文件，而且与`ansible`主机的文件内容不同时，是否对远程主机的文件进行备份，可选值有yes或者no，当设置为yes时，会先备份远程主机的文件，然后再将`ansible`主机的文件拷贝到远程主机
6. owner 文件拷贝到远程主机后的属主 
7. group 文件拷贝到远程主机后的属组
8. mode 文件拷贝到远程主机后的权限，（若要文件权限为rw-r--r--, 可以设置为mode=0644;或者通过权限位，比如mode=u+x)


**使用场景**

1. 将jdk1.8.0_102.tar.gz从ansible的主机拷贝到远程主机上的/opt目录里，并设置权限为0755
`ansible sw -i inventory.ini -m copy -a "src=jdk1.8.0_102.tar.gz dest=/opt/ mode=0755"`
可以看到远程的文件已经拷贝了 `-rwxr-xr-x 1 root root 181468986 May 19 10:37 jdk1.8.0_102.tar.gz`

2.  将jdk.tar.gz从ansible的主机拷贝到远程主机上的/opt目录里，并设置权限为0755，并备份
`ansible sw -i inventory.ini -m copy -a 'src=jdk.tar.gz dest=/opt mode=0644 backup=yes'`
当使用`backup`参数后，当目录文件的内容发生变化后，会在同目录位置生成一个`jdk.tar.gz.1551.2023-05-19@10:58:34~`的备份文件，格式为文件名+时间格式

3. 在远程目标主机生成一个文件，内容为111\n222
`ansible sw -i inventory.ini -m copy -a 'content="111\n222\n" dest=/opt/1.log mode=0755'`
当使用`content`参数后，远程目标主机的`dest`类型必须为一个文件

4.  将jdk.tar.gz从ansible拷贝到远程主机上，如果远程主机上存在了jdk.tar.gz这个文件，并且远程主机上该文件的内容发生了变化，**不执行拷贝操作**
`ansible sw -i inventory.ini -m copy -a 'src=jdk.tar.gz dest=/opt mode=0755 force=no'`
5.  将jdk.tar.gz从ansible拷贝到远程主机，并修改属主属组为nobody
`ansible sw -i inventory.ini -m copy -a 'src=jdk.tar.gz dest=/opt owner=nobody group=nobody backup=yes'`
如果用户或者用户组不存在，就会报错


### file

> file模块可以帮助我们对文件进行基本操作，创建文件、目录；删除文件、目录；修改文件权限

**参数**
1. path 必须参数，指定要操作的文件或者目录在远程主机的位置，为了兼容之前版本，使用`dest`或者`name`也可以
2. state 灵活的参数
   1. 如果要创建的`path`是一个目录时，要将`state`设置为`directory`
   2. 如果要创建的`path`是一个文件时，要将`state`设置为`file`/`touch`(touch当path文件不存在时，会创建，否则会**更新时间戳**，而file则不会，当path不存在时，报错)
   3. 如果要创建的`path`是一个软链接，要将`state`设置为`link`
   4. 如果要创建的`path`是一个硬链接，要将`state`设置为`hard`
   5. 如果要删除`path`， 要将`state`设置为`absent`（删除时不会区分文件、目录或者是链接）
3. src 当state设置为link或者hard后，表示我们要创建一个软链接或者硬链接。我们必须要指明软链接、硬链接的是哪个文件，即通过这个`src`参数指定
4. force 当state为link时，可配置此参数强制创建链接文件
   1. 当force为yes时，表示强制创建链接文件，不过强制创建链接文件也分两种情况
      1. 当你要创建的链接文件指向的源文件不存在时，使用`force=yes`时，可以先强制创建出链接文件
      2. 当你要创建的链接文件中的目录已经存在与链接文件同名的文件时，使用`force=yes`时，会将同名文件覆盖为链接文件，相当于删除了同名的文件，并创建链接文件
      3. 当你要创建的链接文件中的目录中已经存在与链接文件同名的文件，并且链接文件指向的源文件也不存在，这时会强制替换同名文件为链接文件
5. owner 修改`path`文件、目录的属主
6. group 修改`path`文件、目录的属组
7. mode 指定被操作文件的权限，（若要文件权限为rw-r--r--, 可以设置为mode=0644;或者通过权限位，比如mode=u+x)
8. recurse 当操作的`path`为目录，将`recurse`设置为yes，会递归的修改目录中文件的属性

**使用场景**

1. 在目标服务器上创建一个文件，当文件不存在时创建
```shell
ansible sw -i inventory.ini -m file -a 'path=/opt/testmyfile state=touch
```
2. 在目标服务器上创建一个文件，并更新文件属组属主为nobody，加上执行权限
```shell
ansible sw -i inventory.ini -m file -a 'path=/opt/testmyfile owner=nobody group=nobody mode=u+x state=file'
```
3. 在目标服务器上创建一个链接文件，软链接文件名为softlinkfile，执行命令时候，/opt/testmyfile是存在的
```shell
ansible sw -i inventory.ini -m file -a 'src=/opt/testmyfile path=/opt/softlinkfile state=link'

可以看到文件生成
-rwxr--r-- 1 nobody nobody    0 May 19 15:05 testmyfile
lrwxrwxrwx 1 root   root     15 May 19 15:18 softlinkfile -> /opt/testmyfile
```
4. 
5. b
6. c