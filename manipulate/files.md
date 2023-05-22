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
3. 在目标服务器上创建一个软链接文件，软链接文件名为softlinkfile，执行命令时候，/opt/testmyfile是存在的
```shell
ansible sw -i inventory.ini -m file -a 'src=/opt/testmyfile path=/opt/softlinkfile state=link'

可以看到文件生成
-rwxr--r-- 1 nobody nobody    0 May 19 15:05 testmyfile
lrwxrwxrwx 1 root   root     15 May 19 15:18 softlinkfile -> /opt/testmyfile
```
4. 在目标服务器上创建一个硬链接文件，软链接文件名为hardlinkfile，执行命令时候，/opt/testmyfile是存在的
```shell
ansible sw -i inventory.ini -m file -a 'src=/opt/testmyfile path=/opt/hardlinkfile state=hard'
```
5. 在目标服务器上创建链接文件，如果源文件不存在或者是与其他文件同名时，强行覆盖
```shell
ansible sw -i inventory.ini -m file -a 'path=/opt/hardlinkfile state=link force=yes'
```
6. 修改目标服务器的文件权限0755、属组、属主为nobody

```shell
ansible sw -i inventory.ini -m file -a 'path=/opt/touchfile state=touch owner=nobody group=nobody mode=u+x'
```

### blockinfile

> 表示在指定的文件中插入一段标记过的文本， 我们在这段文件中做了一些记号，以便日后通过标记找到文本，可以修改、删除它

**参数**
1. path 要操作的文件路径, *必须参数*
2. block 指定我们要操作的那一段文本，这个参数也称为`content`
3. marker、marker_begin、marker_end 这三个参数用于插入文本时的标记，默认情况下
   1. marker的默认值是 "# {mark} ANSIBLE MANAGED BLOCK"
   2. marker_begin的默认值是 "BEGIN"
   3. marker_end的默认值是"END"
   {mark}会被自动替换成 开始标记和结束标记中的`BEGIN`和`END`，通过插入段落文本，为不同段落添加不同的标记，**下次通过标记可以找到对应的段落**
4. state block的状态，该值有两个参数，`present`和`absent`，默认值是`present`
   1. 当值为`present`时，默认会更新对应的`block`的文本内容
   2. 当值为`absent`时， 从文件中删除`block`中的文本内容

下面两个参数涉及到插入文本的方式，默认会在文件末尾插入文本
5. insertafter  指定将文本插入某一行的后面，使用参数指定对应的行；也可以使用python的正则表达式将文本插入到符合正则的行之后（若多行文本都被正则匹配，则以最后一个满足的为准），该参数还可以设置`EOF`, 表示将文本插入到文档末尾
6. insertbefore 指定将文本插入某一行的前面，使用参数指定对应的行；也可以使用python的正则表达式将文本插入到符合正则的行之前（若多行文本都被正则匹配，则以最后一个满足的为准），该参数还可以设置`BOF`, 表示将文本插入到文档开头
7. backup 是否需要在修改文件之前对其进行备份，默认值是不会备份的
8. create 当操作的文件不存在时，是否创建对应的文件，默认值是不会创建的

**使用场景**

1. 测试环境之前，将测试文件/opt/rc.local准备一下，要在/opt/rc.local文件末尾添加两行

`systemctl start mariadb`
`systemctl start httpd`

```shell
ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local block="systemctl start mariadb\nsystemctl start httpd"'

可以看到插入内容后，文本文件的内容变成了

# BEGIN ANSIBLE MANAGED BLOCK  
systemctl start mariadb
systemctl start httpd
# END ANSIBLE MANAGED BLOCK
```
 BEGIN ANSIBLE MANAGED BLOCK  是开始标记
 END ANSIBLE MANAGED BLOCK    是结束标记

2. 在/opt/rc.local文件末尾添加两行，并修改标注的内容

```shell
ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local block="hahaha\nhehehe" marker="#{mark} myself" '

可以看到插入内容后，文本文件的内容变成了
#BEGIN myself
hahaha
hehehe
#END myself

```
3. 在/opt/rc.local文件末尾更新之前block的内容(这个是会按照marker的位置做修改的)，修改marker="#{mark} myself"的内容

```shell
ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local block="NGINX" marker="#{mark} myself"'

可以看到插入内容后，文本文件的内容变成了

#BEGIN myself
NGINX
#END myself

```

4. 清理block中的文本内容
```shell

ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local state=absent marker="#{mark} MARK1"'

可以看到插入内容后，文本文件的内容连注释的标记和内容一起消失了

```
5. 将文本块插入到文档的开头，使用`insertbefore`参数，将其值设置为`BOF`

```shell
ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local block="FIRST OF ALL" marker="#{mark} mybegin" insertbefore=BOF'

可以看到插入内容后，文本文件的内容变成了

#BEGIN mybegin
FIRST OF ALL
#END mybegin
#!/bin/bash
```

6. 正则匹配touch这个字母，将文本内容叫alpha的插入这个touch字母之上

```shell
ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local block="alpha" marker="#{mark}alphabeta" insertbefore="^touch"'

可以看到插入内容后，文本文件的内容变成了

#BEGINalphabeta
alpha
#ENDalphabeta
touch /var/lock/subsys/local
```

7. 在/opt/rc.local.new里插入文本内容，如果文件不存在就创建文件

```shell
ansible sw -i inventory.ini -m blockinfile -a 'path=/opt/rc.local.new block="new bee" marker="#{mark} SWORD" create=yes backup=yes '

可以看到插入内容后，文本文件的内容变成了

#BEGIN SWORD
new bee
#END SWORD

backup参数会在备份文件后添加时间戳
-rw-r--r-- 1 root   root     32 May 19 19:21 rc.local.new.27142.2023-05-19@19:24:40~
```

### lineinfile

> 确保 某一行文本 存在于指定的文本，或者确保文本从文件中删除， 或者通过正则表达式，替换 某一行文本

**参数**
1. path参数， 指定要操作的文件
2. line参数， 指定文本内容
3. regexp参数，使用**正则匹配**的对应行，若多行文本被匹配，只有最后匹配的文本会被替换；若在删除的场景下，多行文本被匹配，那么多行都会被删除
4. state参数 该行是否应该存在
   1. present 存在（默认是）
   2. absent 删除(
5. backrefs 参数 当正则匹配替换文本时，如果存在分组，可以通过设置backref=yes开启后向引用, 这样line参数就能对regexp参数中的分组进行后项引用(详见实例)
   1. backrefs=yes还有一个功能，如果正则没有匹配到任何行，line对应的内容会被插入到文件的末尾，如果设置了backref=yes，当正则没有匹配到任何行时，不会对文件进行任何操作
6. insertafter 通过insertafter参数将文本插入指定的行后 (如果使用backrefs，此参数忽略)
   1. EOF（默认是文档的末尾）
   2. *regex* 如果设置成正则表达式，表示将文本插入到匹配的正则的行之后，没有正则匹配到，则表示插入文档末尾
7. insertbefore  通过insertafter参数将文本插入指定的行前 (如果使用backrefs，此参数忽略)
   1. BOF（默认是文档的开头）
   2. *regex* 如果设置成正则表达式，表示将文本插入到匹配的正则的行之前，没有正则匹配到，则表示插入文档末尾
8. backup参数 修改文件之前是否对文件进行备份
9. create参数 当操作的文件不存在，是否创建文件


**使用场景**

本次实验的文本内容如下

```ini
# comment
Hello ansible,Hiiii
redundant
11112222
redundant
WTF
```

1. 确保指定的**一行文本**存在于文件中，若文本存在，不做任何操作；若不存在，默认在文件末尾插入这行文本 (若文件不存在，则创建这个文件)

```shell
ansible sw -i inventory.ini -m lineinfile -a 'path=/opt/testfile line="QQQ" '

skywalking-ecs-p001.shL.vevor.net | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "backup": "",
    "changed": true,
    "msg": "line added"
}
skywalking-ecs-p002.shL.vevor.net | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "backup": "",
    "changed": true,
    "msg": "line added"
}
skywalking-ecs-p003.shL.vevor.net | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "backup": "",
    "changed": true,
    "msg": "line added"
}

```
2. 使用正则表达式替换“某一行”，如果匹配不止一行，只有最后匹配的正则会被替换，替换内容会换成line参数指定的内容，如果没匹配上，line参数的内容会被添加到文件的最后一行

```shell
ansible sw -i inventory.ini -m lineinfile -a 'path=/opt/testfile regexp="^[a-z].*" line="Mozart" '

```

3. 使用正则表达式替换“某一行”， 使用正则匹配删除某个文件中以"#"开头的行

```shell
ansible sw -i inventory.ini -m lineinfile -a 'path=/opt/testfile regexp="^#" state=absent'
```

4. 使用正则表达式替换“某一行”， 如果匹配不止一行，只有最后的一个正则匹配的行会被替换，被替换的内容换成line参数后的内容，如果没有被正则匹配到， **不对文件进行任何操作(通过使用backrefs=yes)**

```shell
ansible sw -i inventory.ini -m lineinfile -a 'path=/opt/testfile regexp="^#" line="castlevania" backrefs=yes'
```

5. 使用正则表达式删除匹配的行

```shell
 ansible sw -i inventory.ini -m lineinfile -a 'path=/opt/testfile regexp="^re" state=absent'
```

---
将文本内容替换为以下， 为了演示通过正则表达式匹配的后项引用

```ini
# comment
Hello world Hello
Hiiii world Hiiii
Hello world Hiiii
```


6. 使用后项引用，通过正则表达式，把匹配到的文本内容替换为第一组匹配到的内容

```shell
ansible sw -i inventory.ini -m lineinfile -a 'path=/opt/testfile  regexp="(H.{4}) world (H.{4})" line="\1" backrefs=yes '
```
