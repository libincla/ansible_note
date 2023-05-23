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


---

### find

>  find模块帮我们在远程主机上查找到符合条件的文件

**参数**
1. paths: 指定在哪个目录中查找文件，可以指定多个路径，路径之间使用逗号隔开，此参数有别名，可以使用path或者name来替代paths
2. recurse: 递归查找；默认情况下，只会在指定的目录中查找文件，加上这个参数后，ansible会递归的进入指定目录的子目录查找文件
3. hidden: 是否查找隐藏文件，默认情况下，不会查找隐藏文件
4. file_type: 顾名思义，就是要查找的文件类型
   1. `any`
   2. `directory`
   3. `file` (默认是这个)
   4. `link`
5. pattern(patterns): 此参数指定要查找的文件名称，支持`shell`（通配符）或者正则表达式去匹配文件名称，若要使用`python`的正则表达式匹配文件名，将`use_regex`参数设置为`yes`
6. use_regex: 默认情况下，find模块不会使用正则表达式解析`patterns`参数中对应的内容，当`use_regex`设置为`yes`时，使用`python`正则解析`patterns`参数中的表达式；否则的话，使用**通配符**来解析`patterns`
   1. 设置为`false`，使用通配符查找（默认）
   2. 设置为`true`，使用python的正则表达式
7. contains: 与文件内容匹配的正则表达式或者模式(只有当file_type是文件的时候，生效)
8. age: 根据时间范围查找文件，默认以文件的`mtime`与指定时间做对比
   1. **这里的时间是指从当前时间往前后推三天**
      1. 比如，查找`mtime`三天之前的文件，设置`age=3d`
      2. 比如，查找`mtime`三天之内的文件，设置`age=-3d`
   2. 单位支持 秒(s)、分(m)、时(h)、天(d)、星期(w)
9.  age_stamp: 按照时间种类来查找时，时间种类
    1.  atime
    2.  ctime
    3.  mtime
10. size: 使用此参数可以根据文件大小查找文件 (单位支持，t,g,m,k,b)
    1.  比如，查找大于3M的文件，设置`size=3m`
    2.  比如，查找小于50k的文件，设置`size=-50k`
11. get_checksum： 当符合查找条件的文件被找到时，返回文件的`sha1`校验码;若文件大小很大，生成校验码时间会长


**使用场景**

1. 在目标主机的某个目录下，查找文件内容中包含"Hello"字符串的文件，不包含隐藏文件

```shell
ansible sw -i inventory.ini -m find -a 'paths=/opt contains=".*Hello.*"'

查找结果
...
     {
            "atime": 1684739126.108184,
            "ctime": 1684739123.7081478,
            "dev": 64769,
            "gid": 99,
            ...
            "mode": "0755",
            "mtime": 1684739123.7081478,
            "nlink": 1,
            "path": "/opt/testfile",
            "pw_name": "nobody",
            ...
            "xoth": true,
            "xusr": true
        }
    ],
    "matched": 2,
    "msg": ""
...
```
2. 在目标主机的某个目录下，**递归**查找文件内容中包含"Hello"字符串的文件，不包含隐藏文件

```shell
ansible sw -i inventory.ini -m find -a 'paths=/opt contains=".*Hello.*" recurse=yes '
```

3. 在目标主机上的某个目录下，查找以.log结尾的日志文件，包含隐藏文件，不去递归查找，不包含非file的其他文件格式

```shell
ansible sw -i inventory.ini -m find -a 'paths=/var/log patterns="*.log" hidden=yes'
```
4. 在目标主机上的某个目录下，查找以.conf结尾的日志文件，包含隐藏文件，包含所有文件类型，不去递归查找
   
```shell
ansible sw -i inventory.ini -m find -a 'paths=/etc patterns="*.conf" file_type=any hidden=yes'

```
5. 在目标主机上的某个目录下，查找以.conf结尾的日志文件，包含隐藏文件，包含所有文件类型，不去递归查找(使用**正则表达式方式**查找)

```shell
ansible sw -i inventory.ini -m find -a 'paths=/etc patterns=".*\.conf" use_regex=yes file_type=any hidden=yes'
```
6. 在目标主机上的某个目录下，查找mtime在4天之内的log文件，不包含隐藏文件，不包含目录、软链接等文件类型

```shell
ansible sw -i inventory.ini -m find -a 'paths=/var/log patterns="*.log" age=-4d recurse=yes'
```
7. 在目标主机上的某个目录下，查找atime在2周内的文件，不包含隐藏文件、目录以及软链接

```shell
ansible sw -i inventory.ini -m find -a 'paths=/var/log age=-2w age_stamp=atime recurse=yes'
```

8. 在目标主机上的某个目录下，查找大于100m的文件，不包含隐藏文件、目录以及软链接的文件类型

```shell
ansible sw -i inventory.ini -m find -a 'paths=/var/log size=100m recurse=yes'
```

9. 在目标主机上的某个目录下，查找大于1m的日志，包含隐藏文件、递归查找并且返回符合条件的sha1码

```shell
ansible sw -i inventory.ini -m find -a  'paths=/var/log patterns="*.log" size=1m hidden=yes recurse=yes get_checksum=yes'
```

---

### replace

> 根据指定的正则表达式替换文本的字符串，**所有匹配到的字符串都会被替换**

**参数**
1. path: 必须，指定要操作的文件
2. regexp: 必须，指定一个python正则表达式，匹配到的字符串会被替换
3. replace: 指定要替换的字符串内容
4. backup: 是否要在修改前备份文件
5. after: 如果指定了，匹配之后文档才会被替换或者删除
   1. 使用python的正则表达式
   2. 如果使用启动`DOTALL`, `.`这个字符可以匹配任意的单个字符，包括换行符和回车符
6. before: 如果指定了，匹配之前文档才会被替换或者删除
   1. 使用python的正则表达式
   2. 如果使用启动`DOTALL`, `.`这个字符可以匹配任意的单个字符，包括换行符和回车符
   


**使用场景**

测试文本以`/etc/ssh/ssh_config`为主

1. 将目标主机的文件中所有的`Host`字符串替换为`MyHOST`

```shell
ansible sw -i inventory.ini -m replace -a 'path=/opt/ssh_config regexp=".*Host.*" replace="MyHOST" '
```
2. 将目标主机的文件中所有的`Host`字符串替换为`MyHOST`， 进行备份
   
```shell
ansible sw -i inventory.ini -m replace -a 'path=/opt/ssh_config regexp=".*Host.*" replace="MyHOST" backup=yes'
```
3. 使用正则表达式替换/etc/hosts文件

```shell

1. 首先拷贝文件
ansible sw -i inventory.ini -m copy -a 'src=/etc/hosts dest=/opt owner=nobody group=nobody mode=0755'

2. 做文本替换, 将shL替换为shC

ansible sw -i inventory.ini -m replace -a 'path=/opt/hosts regexp="(\S+)\.shL\.(\S+)" replace="\1.shC.\2"'

```

4. 替换正则表达式之后的文件内容
为了演示这个效果，特意在hosts的文件上加一行，在127.0.0.1之前加一行，测试文本如下

```ini
::1	localhost	localhost.localdomain	localhost6	localhost6.localdomain6
172.21.0.116	skywalking-ecs-p003.shC.vevor.net	skywalking-ecs-p003.shC.vevor.net

127.0.0.1	localhost	localhost.localdomain	localhost4	localhost4.localdomain4

172.21.0.100	jenkins-ecs-p001.shC.vevor.net	jenkins-ecs-p001.shC.vevor.net

172.21.37.85	cop-ecs-testa001.shC.vevor.net	cop-ecs-testa001.shC.vevor.net

172.21.0.116	skywalking-ecs-p003.shC.vevor.net	skywalking-ecs-p003.shC.vevor.net
```
---
这次只修改 127.0.0.1之后匹配到的正则表达式
```shell
ansible sw -i inventory.ini -m replace -a 'path=/opt/hosts regexp="(\S+)\.shC\.(\S+)" replace="\1.shD.\2" after="127.0.0.1" backup=yes'
```

文档中，`after`接收的值类型是string，这里只要写到包含字符的行即可，通过命令可以发现

```ini
::1	localhost	localhost.localdomain	localhost6	localhost6.localdomain6
172.21.0.116 skywalking-ecs-p003.shC.vevor.net skywalking-ecs-p003.shC.vevor.net
127.0.0.1	localhost	localhost.localdomain	localhost4	localhost4.localdomain4 

# 127.0.0.1之后的行才发生了变化，之前的行并没有改变

172.21.0.100	jenkins-ecs-p001.shD.vevor.net	jenkins-ecs-p001.shD.vevor.net

172.21.37.85	cop-ecs-testa001.shD.vevor.net	cop-ecs-testa001.shD.vevor.net

172.21.0.115	skywalking-ecs-p001.shD.vevor.net	skywalking-ecs-p001.shD.vevor.net
```

5. 使用正则表达式替换某个行之前的内容

```shell
ansible sw -i inventory.ini -m replace -a 'path=/opt/hosts regexp="(\S+)\.shC\.(\S+)" replace="\1.shX.\2" before="127.0.0.1" backup=yes'
```
文档中，`before`接收的值类型是string，这里只要写到包含字符的行即可，通过命令可以发现

6. 使用正则表达式将匹配文本前面加上注释

```shell
ansible sw -i inventory.ini -m replace -a 'path=/opt/hosts after="127.0.0.1" regexp="^(.*)$" replace="# \1" '
```


