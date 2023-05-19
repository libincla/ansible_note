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
