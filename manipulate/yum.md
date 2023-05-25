# yum 常用包

> 介绍一下包管理模块

## yum 
在远程主机上通过`yum`源管理软件包

**参数**
1. 
2. b
3. c

**使用场景**


## yum_repository

`yum_repository`帮我们管理远程主机的`yum`仓库

**参数**
1.  name: 必须的参数，指定要操作的唯一的仓库`ID`，也是`.repo`配置文件中每个仓库的对应中括号的仓库`ID`
2.  baseurl: 参数设置`yum`仓库的`baseurl`
3.  description: (必须提供) 参数设置仓库的注释信息，也就是`.repo`配置文件中每个仓库对应的`name`对应的内容
4.  file: 参数用于设置仓库的配置文件名称，设置`.repo`配置文件的文件名前缀，在不使用此参数的情况下，默认以`name`参数的仓库`ID`作为`.repo`配置文件的文件名前缀，同一个`.repo`配置文件中可以存在多个`yum`源
5.  enabled: 参数用于设置是否激活对应的`yum`源
    1.  yes
    2.  no
6.  gpgcheck: 参数用于设置是否开启`rpm`包验证功能
    1.  no
    2.  yes
7.  gpgcakey: 当`gpgcheck`设置为`yes`后，需要使用此参数指定验证包所需的公钥
8.  state: 默认是`present`，当值设置为`absent`时，表示删除对应的源

**使用场景**

1. 在远程主机上添加`id`为`aliEpel`的`yum`源，仓库的配置文件路径在`/etc/yum.repo.d/aliEpel.repo`

```shell
ansible sw -i inventory.ini -m yum_repository -a 'name=aliEpel description="ali EPEL" baseurl="https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/" '

查看是否生成aliEpel.repo

ansible sw -i inventory.ini -m shell -a 'ls -l /etc/yum.repos.d/aliEpel.repo'

skywalking-ecs-p003.shL.vevor.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 99 May 25 15:46 /etc/yum.repos.d/aliEpel.repo
skywalking-ecs-p001.shL.vevor.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 99 May 25 15:46 /etc/yum.repos.d/aliEpel.repo
skywalking-ecs-p002.shL.vevor.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 99 May 25 15:46 /etc/yum.repos.d/aliEpel.repo


ansible sw -i inventory.ini -m shell -a 'cat  /etc/yum.repos.d/aliEpel.repo
[aliEpel]
baseurl = https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/
name = ali EPEL

```

2. 在远程主机上添加`id`为`aliEpel`的`yum`源，仓库的配置文件路径在`/etc/yum.repo.d/alibaba.repo`

```shell
ansible sw -i inventory.ini -m yum_repository -a 'name=aliEpel baseurl="https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/" file=alibaba description="alibaba epel repo" '

查看生成结果

ansible sw -i inventory.ini -m shell -a 'ls -l /etc/yum.repos.d/alibaba.repo'

skywalking-ecs-p002.shL.vevor.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 108 May 25 15:52 /etc/yum.repos.d/alibaba.repo


ansible sw -i inventory.ini -m shell -a 'cat /etc/yum.repos.d/alibaba.repo'
skywalking-ecs-p003.shL.vevor.net | CHANGED | rc=0 >>
[aliEpel]
baseurl = https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/
name = alibaba epel repo

```

3. 在远程主机上设置ID为`local`的`yum`源，但是不启动它(local源使用系统光盘作为本地`yum`源，baseurl以`file:///`开头)

```shell
ansible sw -i inventory.ini -m yum_repository -a 'name=local baseurl=file:///media description="local cd yum" enabled=no'
```
