# 变量 

> `ansible`可以使用变量，让我们使用更加灵活

## 变量定义

变量由字母、数字、下划线组成，要以字母为开头，内置的关键字不能作为变量名使用

## 在playbook中使用变量

使用`vars`关键字，可以在`playbook`中使用变量

```yaml
---

- hosts: sw
  remote_user: root
  vars:
    testvar: ttt
  tasks:
  - name: task1
    file:
      path: /opt/{{ testvar }}.log
      state: touch

```

这里我们在`vars`里定义了一个变量， 变量名是`testvar`，变量值是`ttt`； 当我们想引用变量名`testvar`时，使用`{{ }}`引用对应的变量

### 当需要定义多个变量

1. 可以多种方式定义变量

**方式1**

```yaml
testvars:
  a1: abcd
  a2: bcde
```

**方式2**

```yaml
testvars:
- a1: aaaa
- b1: bbbb
```

2. 使用类似**属性**方式定义变量

```yaml

---
- hosts: sw
  remote_user: root
  vars:
    nginx:
      conf80: /etc/nginx/conf.d/80.conf
      conf8080: /etc/nginx/conf.d/8080.conf
  tasks:
  - name: task1
    file:
      path: "{{ nginx.conf80 }}"
      state: touch
  - name: task2
      path: "{{ nginx.conf8080 }}"
      state: touch
```

```shell
# ansible-playbook -i inventory.ini test11.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [sw] ******************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [skywalking-ecs-p003.shL.XXX.net]
ok: [skywalking-ecs-p002.shL.XXX.net]
ok: [skywalking-ecs-p001.shL.XXX.net]

TASK [task1] ***************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]

TASK [task2] ***************************************************************************************************
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]

PLAY RECAP *****************************************************************************************************
skywalking-ecs-p001.shL.XXX.net : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXX.net : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXX.net : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

当使用这种**属性**时，可以使用两种方式来引用
引用1: {{ nginx.conf80 }}
引用2: {{ nginx['conf80']}}


3. 变量中的双引号
   1. 当变量被引用时，处于**顶头的位置**，要**使用双引号**，例如 `path: "{{ nginx.conf80 }}"`
   2. 当变量使用"="为变量赋值的时候，不用考虑双引号的问题，例如 `path={{ nginx.conf80 }}`

### 使用变量文件产生变量与剧本分离

> 这样有了`role`的雏形了, 在`playbook`里使用`vars_files`关键字即可, `vars_files`可以接收多个变量文件定义

将变量写在`nginx_vars.yml`里

```yaml
---

- hosts: sw
  remote_user: root
  vars_files:
  - nginx_vars.yml
  tasks:
  - name: touch file1
    file:
      path: "{{ nginx.conf60 }}"
      state: touch
  - name: touch file2
    file:
      path: "{{ nginx.conf6060 }}"
      state: touch

```

- `nginx_vars.yml`内容 

```yaml
---
nginx:
  conf60: /etc/nginx/conf.d/60.conf
  conf6060: /etc/nginx/conf.d/6060.conf
```

## 跳过变量收集

在`playbook`中，使用`gather_facts: false`，关闭默认收集远程主机的相关信息，包括远程主机的`IP`地址、主机名、系统版本、硬件配置等等信息


## setup模块

> `setup`模块可以查看到`gathering facts`任务收集到的信息, 例如

```shell
# ansible sw -i inventory.ini -m setup | more
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
skywalking-ecs-p003.shL.XXX.net | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.21.0.116",
            "172.17.0.1"
        ],
        "ansible_all_ipv6_addresses": [
            "::AAAA:BBBB:CCCC:DDDD"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "04/01/2014",
        "ansible_bios_version": "9e9f1cc",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/boot/vmlinuz-3.10.0-1160.76.1.el7.x86_64",
            "console": "ttyS0,115200n8",
            "crashkernel": "auto",
            "net.ifnames": "0",
            "noibrs": true,
            "nvme_core.admin_timeout": "4294967295",
            "nvme_core.io_timeout": "4294967295",
            "quiet": true,
            "rhgb": true,
            "ro": true,
            "root": "UUID=4f4de78b-e926-4e41-9ab6-0643a8eed46f",
            "spectre_v2": "retpoline"
        },
        "ansible_date_time": {
            "date": "2023-06-06",
            "day": "06",
            "epoch": "1686017969",
            "hour": "10",
            "iso8601": "2023-06-06T02:19:29Z",
            "iso8601_basic": "20230606T101929981760",
            "iso8601_basic_short": "20230606T101929",
            "iso8601_micro": "2023-06-06T02:19:29.981760Z",
            "minute": "19",
            "month": "06",
            "second": "29",
            "time": "10:19:29",
            "tz": "CST",
            "tz_offset": "+0800",
            "weekday": "Tuesday",
            "weekday_number": "2",

.... 
```

返回的是一个很长的`json`字符串，里面的信息非常全面，例如

1. `ansible_all_ipv4_addresses`: 远程主机的所有`ipv4`地址
2. `ansible_architecture`: 远程主机的系统架构
3. `ansible_bios_date`: 远程主机的`bios`日期
等等非常之多

### setup模块过滤信息

使用`filter=`可以过滤`setup`收集的信息，我们来获取我们想要的信息

```shell

1. 仅仅展示`setup`获取的远程主机的内存信息

ansible sw -i inventory.ini -m setup -a 'filter=ansible_memory_mb' -l skywalking-ecs-p001.shL.XXX.net
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
skywalking-ecs-p001.shL.XXX.net | SUCCESS => {
    "ansible_facts": {
        "ansible_memory_mb": {
            "nocache": {
                "free": 2318,
                "used": 5360
            },
            "real": {
                "free": 155,
                "total": 7678,
                "used": 7523
            },
            "swap": {
                "cached": 0,
                "free": 0,
                "total": 0,
                "used": 0
            }
        },
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
2. 仅仅展示`setup`获取的远程主机的所有ipv4地址

ansible sw -i inventory.ini -m setup -a 'filter=ansible_all_ipv4_addresses' -l skywalking-ecs-p001.shL.XXX.net

skywalking-ecs-p001.shL.XXX.net | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.21.0.115",
            "172.17.0.1"
        ],
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}

```

此外，`filter`还支持通配符模式，比如我想只找到带有`mb`的字符的收集的信息，可以这样

```shell

ansible sw -i example.ini -m setup -a 'filter=*mb*' -l skywalking-ecs-p002.shL.XXX.net
skywalking-ecs-p002.shL.XXX.net | SUCCESS => {
    "ansible_facts": {
        "ansible_memfree_mb": 202,
        "ansible_memory_mb": {
            "nocache": {
                "free": 2869,
                "used": 4809
            },
            "real": {
                "free": 202,
                "total": 7678,
                "used": 7476
            },
            "swap": {
                "cached": 0,
                "free": 0,
                "total": 0,
                "used": 0
            }
        },
        "ansible_memtotal_mb": 7678,
        "ansible_swapfree_mb": 0,
        "ansible_swaptotal_mb": 0,
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```


### 自定义fact信息

**位置**
`ansible`默认会去远程主机的`/etc/ansible/facts.d`目录下查找主机中的**自定义信息**，规定自定义信息要写在`.fact`后缀的文件中。这些`.fact`文件格式内容要以`INI`或者`JSON`格式的

**如何引用**
我们在远程主机自定义的信息，被称为**local facts**，当我们在使用`setup`模块收集信息时，远程主机的`local facts`也会被收集， 它被称为**ansible_local**

为了展示，在`skywalking-ecs-p001.shL.XXX.net`主机内创建示例文件`/etc/ansible/facts.d/testinfo.ini`

```ini
[testmsg]
msg1=Hello
msg2=World
```

下面展示

```shell

ansible sw -i example.ini -m setup -a 'filter=ansible_local' -l skywalking-ecs-p001.shL.XXX.net
skywalking-ecs-p001.shL.XXX.net | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "testinfo": {
                "testmsg": {
                    "msg1": "Hello",
                    "msg2": "World"
                }
            }
        },
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}

```

通过上述的方式，我们可以在目标主机上自定义信息。


### 使用debug模块调试变量

`debug`模块有两个作用
1. 通过`msg`参数在控制台上输出信息
2. **直接输出变量中的信息**

下面我们看示例

```yaml
---

- hosts: sw
  remote_user: root
  vars:
    testvar: liusha
  tasks:
  - name: debug variable
    debug:
      var: testvar
```

执行结果

```shell

ansible-playbook -i example.ini test14.yaml

PLAY [sw] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXX.net]
ok: [skywalking-ecs-p002.shL.XXX.net]
ok: [skywalking-ecs-p001.shL.XXX.net]

TASK [debug variable] ***********************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXX.net] => {
    "testvar": "liusha"
}
ok: [skywalking-ecs-p003.shL.XXX.net] => {
    "testvar": "liusha"
}
ok: [skywalking-ecs-p002.shL.XXX.net] => {
    "testvar": "liusha"
}

PLAY RECAP **********************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

`ansible`通过`debug`模块将`var`参数输出到终端

或者在`msg`参数中输出变量

```yaml

---
- hosts: sw
  remote_user: root
  vars:
    testvar: liusha
  tasks:
  - name: debug variable
    debug:
      msg: "{{ testvar }}"
```

举个例子，我们通过`debug`模块展示一下远程主机的内存使用情况

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: debug variable
    debug:
      msg: "remote {{ ansible_host }} memory information: {{ ansible_memory_mb['real'] }}"
```

查看执行结果

```shell

# ansible-playbook -i example.ini test14.yaml

PLAY [sw] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXX.net]
ok: [skywalking-ecs-p001.shL.XXX.net]
ok: [skywalking-ecs-p002.shL.XXX.net]

TASK [debug variable] ***********************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXX.net] => {
    "msg": "remote 172.21.0.115 memory information: 211"
}
ok: [skywalking-ecs-p002.shL.XXX.net] => {
    "msg": "remote 172.21.0.117 memory information: 317"
}
ok: [skywalking-ecs-p003.shL.XXX.net] => {
    "msg": "remote 172.21.0.116 memory information: 266"
}

PLAY RECAP **********************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```