# 命令类型模块

## command

>  在远程主机执行命令
>  不支持 管道、重定向等功能, 因此这些符号会失效 <, >, |, ;
>  如果要支持windows，要使用`win_command`

**参数**

1. `chdir`: 指定要执行命令的目录，`ansible`在执行命令前，会先进入到`chdir`参数指定的目录
2. `creates`: 参数指定的文件存在时，就不执行对应命令（比如，/test/1这个文件存在，就不执行我们指定的命令）
3. `removes`: 参数指定的文件不存在时，就不执行对应命令


**使用场景**

1. 在远程主机上执行命令

```shell
ansible sw -i inventory.ini -m command -a 'ls '
```

2. 在远程主机上执行命令，在/opt目录

```shell
ansible sw  -i inventory.ini  -m command -a 'chdir=/opt ls -lrth' 
```
3. 在远程主机上执行命令，如果/opt/1.log存在在远程主机，不执行命令

```shell
ansible sw -i inventory.ini -m command -a 'creates=/etc/hosts ls -lrth /opt'
```


## shell

> 帮助我们在远程主机上执行命令，`shell`模块在远程主机上执行命令时，会经过远程主机上的`/bin/sh`程序处理

**参数**

1. `chdir`: 指定要执行命令的目录，`ansible`在执行命令前，会先进入到`chdir`参数指定的目录
2. `creates`: 参数指定的文件存在时，就不执行对应命令（比如，/test/1这个文件存在，就不执行我们指定的命令）
3. `removes`: 参数指定的文件不存在时，就不执行对应命令
4. `executable`： 执行`shell`命令的shell是哪个


## script

> 该模块帮助我们在远程主机上执行`ansible`主机上的脚本（即，要执行的脚本存在于`ansible`主机上，不需要先拷贝到远程主机上执行）

**参数**

1. `chdir`: 指定要执行命令的目录，`ansible`在执行命令前，会先进入到`chdir`参数指定的目录
2. `creates`: 参数指定的文件存在时，就不执行对应命令（比如，/test/1这个文件存在，就不执行我们指定的命令）
3. `removes`: 参数指定的文件不存在时，就不执行对应命令
