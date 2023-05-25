# yum 常用包

> 介绍一下包管理模块

## yum 
在远程主机上通过`yum`源管理软件包

**参数**
1. name: 必须参数，指定要管理的软件包，比如`nginx`
2. state: 指定软件包的状态，默认是`present`
   1. absent 确保软件包已删除
   2. installed 确保软件包已经安装
   3. latest 安装`yum`仓库中最新的版本
   4. present 确保软件包已经安装
   5. removed 确保软件包已删除
3. disable_gpg_check: 用于禁对rpm包的公钥`gpg`验证，
   1. yes 禁用验证，不验证包
   2. no (默认) 不会禁用验证，表示会对安装的包进行验证
4. enablerepo:  用于指定安装软件包时临时启动的`yum`源，（假设想从A源中安装软件，不确定是否启用了A源，可以在安装软件包的时候，将此参数设置为yes）
5. disablerepo: 用于指定安装软件包时临时禁用的`yum`源， 某些场景下需要此参数，当多个`yum`源同时存在要安装的软件包时，可以使用此参数临时禁用某个源

**使用场景**

1. 在远程主机上安装/更新`nginx`，禁用验证

```shell
ansible sw -i inventory.ini -m yum -a 'name=nginx state=present disable_gpg_check=yes'
ansible sw -i inventory.ini -m yum -a 'name=nginx state=latest disable_gpg_check=yes'
```

2. 在远程主机上卸载`nginx`

```shell
ansible sw -i inventory.ini -m yum -a 'name=nginx state=absent'
```
3. 在远程主机上安装`tree`时不确定`local`源是否被启用，使用`enablerepo=local`临时启用源

```shell
ansible sw -i inventory.ini -m yum -a 'name=tree disable_gpg_check=yes enablerepo=local
```
4. 在远程主机上安装`tree`时，确定多个源都有，不想从`local`源中安装，在安装时临时禁用`local`源

```shell
ansible sw -i inventory.ini  -m yum -a 'name=tree disable_gpg_check=yes disablerepo=local'
```


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

skywalking-ecs-p003.shL.XXXX.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 99 May 25 15:46 /etc/yum.repos.d/aliEpel.repo
skywalking-ecs-p001.shL.XXXX.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 99 May 25 15:46 /etc/yum.repos.d/aliEpel.repo
skywalking-ecs-p002.shL.XXXX.net | CHANGED | rc=0 >>
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

skywalking-ecs-p002.shL.XXXX.net | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 108 May 25 15:52 /etc/yum.repos.d/alibaba.repo


ansible sw -i inventory.ini -m shell -a 'cat /etc/yum.repos.d/alibaba.repo'
skywalking-ecs-p003.shL.XXXX.net | CHANGED | rc=0 >>
[aliEpel]
baseurl = https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/
name = alibaba epel repo

```

3. 在远程主机上设置ID为`local`的`yum`源，但是不启动它(local源使用系统光盘作为本地`yum`源，baseurl以`file:///`开头)

```shell
ansible sw -i inventory.ini -m yum_repository -a 'name=local baseurl=file:///media description="local cd yum" enabled=no'
```
