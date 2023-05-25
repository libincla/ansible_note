# 系统管理类模块

## cron

> 帮助我们管理远程主机的`crontab`任务

**参数**

1. minute: 计划任务中分钟的值，不设置时，分钟设置的定位的值为"*"
2. hour: 计划任务中的小时的值，不设置时，小时设置的定位的值为"*"
3. day: 计划任务中的天的值，不设置时，天设置的定位的值为"*"
4. month: 计划任务中的月份的值，不设置时，月份设置的定位的值为"*"
5. weekay: 计划任务中的周几的值，不设置时，周几设置的定位的值为"*"
6. special_time: 
   1. annually: 每年
   2. daily: 每天
   3. hourly: 每时
   4. monthly: 每月
   5. reboot: 重启后
   6. weekly: 每周
   7. yearly: 每年
7. user: 设置计划任务属于哪个用户（默认是管理员）
8. job: 设置任务中需要实际执行的命令或者脚本，比如"echo test"这种命令
9. name: 设置任务的名称，任务的名称会在注释中显示
10. state: 根据计划任务的名称，设置修改或者删除计划任务
---
如果上述时间单位未指定，计划任务的时间被设定为"* * * * *"，这样表示每分钟都执行一次计划任务
11. disabled: 根据计划任务的名称，使计划任务失效(注释掉对应的任务)
12. backup: 当修改或者删除对应的计划任务时，先对计划任务进行备份，再对计划任务进行修改或删除

**使用场景**

1. 在远程主机上创建计划任务，任务名称叫test1，任务将于每天2点20分执行，任务内容为输出hello字符

```shell
ansible sw -i inventory.ini -m cron -a 'name="test1" minute=20 hour=2 job="echo 'hello' " '

ansible sw -i inventory.ini -m shell -a 'crontab -l'

skywalking-ecs-p001.shL.vevor.net | CHANGED | rc=0 >>
#Ansible: test1
20 2 * * * echo hello
skywalking-ecs-p003.shL.vevor.net | CHANGED | rc=0 >>
#Ansible: test1
20 2 * * * echo hello
skywalking-ecs-p002.shL.vevor.net | CHANGED | rc=0 >>
#Ansible: test1
20 2 * * * echo hello

```
2.  在远程主机上创建计划任务，任务名称叫test2，任务将于每3天执行一次，执行时间是当日的5点5分

```shell
ansible sw -i inventory.ini -m cron -a 'name="test2" minute=5 hour=5 day=*/3 job="echo test2 &> /dev/null" '

#Ansible: test2
5 5 */3 * * echo test2 &> /dev/null
```

3. 在远程主机上创建计划任务，任务名称叫test3，任务只有在重启后才被执行

```shell
ansible sw -i inventory.ini -m cron -a 'name="test3" special_time="reboot" job="echo test3" '

#Ansible: test3
@reboot echo test3

```

4. 修改远程主机的test3的计划任务，将重启后执行改成每小时执行，并备份计划任务

```shell
ansible sw -i inventory.ini -m cron -a 'name="test3" special_time=hourly job="echo testtest" backup=yes'

#Ansible: test3
@hourly echo testtest

```
备份计划任务可以从返回的信息中的"backup_file": "/tmp/crontabKbtEZd", 看到文件位置


## system

> 可以被管理的服务


**参数**
1. name: 参数用于指定要操作的服务名称，比如`nginx`
2. state: 四种状态，
   1. started(幂等操作)
   2. stopped(幂等操作)
   3. reloaded 将始终重新加载
   4. restarted 会总是触发反弹单位
3. enabled: 选择这个服务单位是否在`boot`阶段会启动
   1. `false`
   2. `true`
4. daemon_reload: 在做任何操作更改之前运行`daemon-reload`，当设置为`true`，不管模块的服务是否启动/停止都运行`daemon-reload`
   1. false (默认是这个)
   2. true
5. masked: 参数表示服务单位是否应该被隐藏，一个`masked`的服务单元是不会启动的
   1. false
   2. true

**使用场景**

1. 安装`httpd`服务，确保这个`httpd`服务是运行的

```shell
ansible sw -i inventory.ini -m yum -a 'name=httpd state=present'
ansible sw -i inventory.ini -m systemd -a 'name=httpd state=started'

```
2. 确保`httpd`服务是停止的

```shell
ansible sw -i inventory.ini -m systemd -a 'name=httpd state=stopped'
```
3. 重启一下`httpd`服务，还使用`daemon-reload`方式获取配置

```shell
ansible sw -i inventory.ini -m systemd -a 'name=httpd state=restarted daemon-reload=yes'
```

4. 确保`httpd`服务，并且确保这个服务单元不是`masked`的

```shell
ansible sw -i inventory.ini -m systemd -a 'name=httpd enabled=yes masked=no'
```
5. 强迫`systemd`重读配置文件

```shell
ansible sw -i inventory.ini -m systemd -a 'daemon_reload=yes'
```

## user

> 管理远程主机的用户，创建、删除、修改用户以及为用户创建密钥

**参数**
1. name: 设置用户的名称
2. create_home:  除非设置`false`，否则用户创建时会创建家目录
   1. false
   2. true(默认)
3. group: 设置该用户所在的组
4. groups: 设置用户所在的附加组。（如果用户想要继续添加新的附加组，要结合append参数使用）
5. shell: 该参数设置用户默认的shell
6. uid: 该参数用于指定用户的`uid`
7. expires: 该参数用于指定用户的过期时间，相当于设置`/etc/shadow`文件的第8列  
   1. 比如要设置用户过期时间为2024年12月31日，首先要获取2024年12月31日的`unix`时间戳，可以使用`date -d 2024-12-31 +%s`, 然后设置`expires=刚才命令的返回值`
8. comment: 参数用于指定用户的注释信息
9. state: 参数用于指定用户是否存在于远程主机上，可选的有
   1.  present: 存在
   2.  absent: 不存在
10. remove: 当state值设置为absent时，只是删除用户，并不删除用户价目录，如果设置remove=yes，删除用户的同时还会删除用户的家目录
    1.  false
    2.  true
11. password: 参数用于指定用户的密码，这个要输入的密码是明文密码加密后的字符串
    1.  举个例子，比如我想设置密码为"asd1234", 可以在python上通过明文密码对应的加密字符串
    2. 
    ```python
    import crypt; crypt.crypt('asd1234') --> '8ag80F/Y22NFs'
    ```
12. update_password: 
    1.  always: 如果用户参数设置的值与之前加密的密码字符串不一样，直接更新用户密码(默认)
    2.  on_create: 如果用户参数设置的值与之前加密的密码字符串不一样，**不会更新**用户密码
13. generate_ssh_key: 是否为用户生成`ssh`密钥对（默认会在用户家目录.ssh生成id_rsa和id_rsa.pub; 若同名的密钥已经存在，原同名的密钥并不会覆盖，若执意覆盖，使用force=yes）
    1.  false（默认）
    2.  true
14. ssh_key_file: 当设置`generate_ssh_key=yes`时，使用该参数自定义生成`ssh`私钥的路径和名称，默认值是`.ssh/id_rsa.`
15. ssh_key_comment: 当设置`generate_ssh_key=yes`时，创建证书后，使用此参数设置公钥中的注释信息，默认注释信息为"ansible-generated on $HOSTNAME" (若同名的密钥已经存在，即不做任何操作，若执意，使用force=yes)
16. ssh_key_passphrase: 当设置`generate_ssh_key=yes`时，使用这个参数可以设置私钥的密码 （若同名的密钥已经存在，即不做任何操作，若执意，使用force=yes)
17. ssh_key_type: 当设置`generate_ssh_key=yes`时, 创建证书时，使用此参数设置密钥的类型，默认密钥类型为`rsa` （若同名的密钥已经存在，即不做任何操作，若执意，使用force=yes)

**使用场景**

1. 在远程主机上创建`libincla`用户，若用户存在，不做任何操作, 密码设置为123456

先查找123456加密后的明文字符串

```python
import crypt;
crypt.crypt('123456')
'ZomhEfHJh/00U'
```
然后执行ansible命令

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla password="ZomhEfHJh/00U" '

```

2. 在远程主机上删除`libincla`用户，包括用户的家目录

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla state=absent remove=yes'
```

3. 在远程主机上删除`libincla`用户，仅仅删除用户，不删除用户家目录

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla state=absent'
```
4. 在远程主机上更新`libincla`用户的密码，若用户当前的加密字符串和命令中设置的加密字符串不一样(将密码12346改成abc123)，不**进行密码更新的操作**

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla password="cRshI7kk4NWdw" update_password=on_create'
```
5.  在远程主机上更新`libincla`用户的密码，将密码12346改成abc123 (update_password=always可以不用加，默认就是这个)

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla password="cRshI7kk4NWdw" update_password=always'
```
6. 在远程主机上新建`libincla`，并且生成`id_rsa`和`id_rsa.pub`密钥文件

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla state=present generate_ssh_key=yes'
```
7. 在远程主机上新建`libincla`，并且生成`id_rsa`和`id_rsa.pub`密钥文件（重复执行，若存在同名的密钥文件，不会覆盖原来的密钥文件）

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla state=present generate_ssh_key=yes'
```
8. 在远程主机上新建`libincla`，并且**执意覆盖**原有的`id_rsa`和`id_rsa.pub`密钥文件 (加入force=yes)

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla state=present generate_ssh_key=yes force=yes'
```
9. 在远程主机上新建`libincla`，自定义ssh密钥对的文件名以及位置(在/opt目录下，生成libincla.pub和libincla)

```shell
ansible sw -i inventory.ini -m user -a 'name=libincla generate_ssh_key=yes ssh_key_file=/opt/libincla'
```
10. 在远程主机上新建`libincla`，指定公钥的注释信息为"www.libincla.com"，(参数只在创建密钥时使用才生效，不操作同名的老密钥)

```shell
ansible sw -i inventory.ini -m user -a 'user=libincla generate_ssh_key=yes ssh_key_comment="www.libincla.com" '
```
11. 在远程主机上新建`libincla`，指定密钥对的类型为dsa

```shell
ansible sw -i inventory.ini -m user -a 'name=libincla generate_ssh_key=yes ssh_key_type=dsa'
```


## group

>