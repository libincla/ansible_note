# 变量 

> `ansible`可以使用变量，让我们使用更加灵活

## 变量定义

变量由字母、数字、下划线组成，要以字母为开头，内置的关键字不能作为变量名使用

**特点**
1. 灵活
2. 有返回值

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

### 通过set_fact定义变量

> `task`支持通过`set_fact`模块定义变量，如例

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: set variable
    set_fact:
      myfact: "wow"
  - name: print variable
    debug:
      msg: "{{ myfact }}"
```

执行一下，看下效果

```shell
# ansible-playbook -i example.ini test23.yaml

PLAY [sw] ********************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]

TASK [set variable] **********************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]

TASK [print variable] ********************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "wow"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "wow"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "wow"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


**通过`set_fact`将变量传递给另一个变量**

查看例子

```yaml
---

- hosts: sw
  remote_user: root
  vars:
  - outsect: "null"
  tasks:
  - name: A
    shell: >
      echo "haha"
    register: haha
  - name: B
    set_fact:
      t1: "{{ outsect }}"
      t2: "{{ haha.stdout }}"
  - name: C
    debug:
      msg: "{{ t1 }} {{ t2 }}"
```
查看执行结果

```shell
# ansible-playbook -i example.ini  test24.yaml

PLAY [sw] ********************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [A] *********************************************************************************************************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]
changed: [skywalking-ecs-p001.shL.XXXX.net]

TASK [B] *********************************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]

TASK [C] *********************************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "null haha"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "null haha"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "null haha"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**set_fact在相同主机的play中使用**

直接看例子，下面的例子两个`play`的主机是同一个

```yaml
---

- hosts: sw
  remote_user: root
  vars:
  - z1: "clar"
  tasks:
  - name: set var
    set_fact:
      o1: "tv"
  - name: print var
    debug:
      msg: "variable is {{ z1 }} ;fact is {{ o1 }}"


- hosts: sw
  remote_user: root
  tasks:
  - name: print z1
    debug:
      msg: "variable is {{ z1 }}"
  - name: print o1
    debug:
      msg: "fact is {{ o1 }}"
```
在第一个`play`中分别定义了两种变量，一个通过`vars`定义的，另一个通过`set_fact`定义的，我们执行一下看下结果

```shell

TASK [print z1] **************************************************************************************************************************************************************************************************************************************
fatal: [skywalking-ecs-p001.shL.XXXX.net]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'z1' is undefined\n\nThe error appears to be in '/app/ansible/test25.yaml': line 19, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: print z1\n    ^ here\n"}
fatal: [skywalking-ecs-p002.shL.XXXX.net]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'z1' is undefined\n\nThe error appears to be in '/app/ansible/test25.yaml': line 19, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: print z1\n    ^ here\n"}
fatal: [skywalking-ecs-p003.shL.XXXX.net]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'z1' is undefined\n\nThe error appears to be in '/app/ansible/test25.yaml': line 19, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: print z1\n    ^ here\n"}

```
在执行到`play2`的`print z1`就报错了，发现`play1`中通过`vars`定义的变量并未生效，我们看下`set_fact`


```shell
TASK [print o1] **************************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "fact is tv"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "fact is tv"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "fact is tv"
}
```
`set_fact`的变量可以被正常输出的，说明`set_fact`中定义的变量是围绕主机组`sw`的
**即便是同一个`role`的作用域也支持**



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


## 如何注册变量

> `ansible`在模块运行后，都会产生一些“返回值“

默认，返回值不会显示，不过我们可以将返回值**写入**变量中，即称为**注册变量**，进而我们可以根据这些返回值进行一些比如 条件判断等等, 通过关键字`register`来实现

一个简单示例

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: test my shell
    shell: >
      echo abc123 > /opt/test123
    register: testvar
  - name: shell module return value
    debug:
      msg: "variable is {{ testvar }}"
```

执行结果

```shell

# ansible-playbook -i example.ini test15.yaml

PLAY [sw] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXX.net]
ok: [skywalking-ecs-p001.shL.XXX.net]
ok: [skywalking-ecs-p002.shL.XXX.net]

TASK [test my shell] ************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]

TASK [shell module return value] ************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXX.net] => {
    "msg": "variable is {'stderr_lines': [], u'changed': True, u'end': u'2023-06-06 19:26:48.235691', 'failed': False, u'stdout': u'', u'cmd': u'echo abc123 > /opt/test123\\n', u'rc': 0, u'start': u'2023-06-06 19:26:48.224598', u'stderr': u'', u'delta': u'0:00:00.011093', 'stdout_lines': []}"
}
ok: [skywalking-ecs-p002.shL.XXX.net] => {
    "msg": "variable is {'stderr_lines': [], u'changed': True, u'end': u'2023-06-06 19:26:48.242078', 'failed': False, u'stdout': u'', u'cmd': u'echo abc123 > /opt/test123\\n', u'rc': 0, u'start': u'2023-06-06 19:26:48.228603', u'stderr': u'', u'delta': u'0:00:00.013475', 'stdout_lines': []}"
}
ok: [skywalking-ecs-p003.shL.XXX.net] => {
    "msg": "variable is {'stderr_lines': [], u'changed': True, u'end': u'2023-06-06 19:26:48.230660', 'failed': False, u'stdout': u'', u'cmd': u'echo abc123 > /opt/test123\\n', u'rc': 0, u'start': u'2023-06-06 19:26:48.221631', u'stderr': u'', u'delta': u'0:00:00.009029', 'stdout_lines': []}"
}

PLAY RECAP **********************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXX.net : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXX.net : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXX.net : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---
例子2

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: curl website
    uri:
      url: https://www.baidu.com
      method: GET
      return_content: true  #这个参数会将返回的内容注册一个叫content的key添加到变量中
    register: getresult

  - name: get result
    debug:
      msg: "get baidu.com result is {{ getresult.content }}"
```
执行

```shell
# ansible-playbook -i inventory.ini test21.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]

TASK [curl website] ***************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]

TASK [get result] *****************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "get baidu.com result is <html>\r\n<head>\r\n\t<script>\r\n\t\tlocation.replace(location.href.replace(\"https://\",\"http://\"));\r\n\t</script>\r\n</head>\r\n<body>\r\n\t<noscript><meta http-equiv=\"refresh\" content=\"0;url=http://www.baidu.com/\"></noscript>\r\n</body>\r\n</html>"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "get baidu.com result is <html>\r\n<head>\r\n\t<script>\r\n\t\tlocation.replace(location.href.replace(\"https://\",\"http://\"));\r\n\t</script>\r\n</head>\r\n<body>\r\n\t<noscript><meta http-equiv=\"refresh\" content=\"0;url=http://www.baidu.com/\"></noscript>\r\n</body>\r\n</html>"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "get baidu.com result is <html>\r\n<head>\r\n\t<script>\r\n\t\tlocation.replace(location.href.replace(\"https://\",\"http://\"));\r\n\t</script>\r\n</head>\r\n<body>\r\n\t<noscript><meta http-equiv=\"refresh\" content=\"0;url=http://www.baidu.com/\"></noscript>\r\n</body>\r\n</html>"
}

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



## 交互式输入变量
实在是不常用，跳过


## 通过命令行传入变量

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: pass variable via command line
    debug:
      msg: "{{ pass_var }}"

```

可以通过命令行选项 -e 或者是 --extra-vars 来传递变量，如下所示； 
此外，-e还支持一次性传入多个变量，每个变量之间使用**空格**隔开，比如`-e ' A="ABC" B="BCD" '`
变量的优先级，命令行的优先级最高


```shell
ansible-playbook -i inventory.ini -e "pass_var=haihaihai" test22.yaml

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]

TASK [pass variable via command line] *********************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "haihaihai"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "haihaihai"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "haihaihai"
}

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## 传入变量的多种形式

1. 普通的`kv`形式
```shell
-e ' A="ABC" B="BCD" '
```
2. 通过`json`形式
```shell
-e '{"pass_var":"sunny", "pass_var1":"ll"}'
```
3. 通过数组模式

```shell
-e '{"pass_var": ["one", "two", "three"], "pass_var1": "null"}'
```
4. 通过变量文件形式，引用的时候通过"@"符号引用

变量文件
```yaml
pass_var:
- one
- two
- three
- four
pass_var1: 1234
```
引用变量 (通过"@"符号引用)
```shell
 ansible-playbook -i example.ini  -e "@passvar" test22.yaml

PLAY [sw] ********************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [pass variable via command line] ****************************************************************************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "[u'one', u'two', u'three', u'four'] 1234"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "[u'one', u'two', u'three', u'four'] 1234"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "[u'one', u'two', u'three', u'four'] 1234"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```


### 主机变量

如何配置主机变量，在编写资源清单文件时候，可以直接使用k=v形式，例如

```ini
[sw]
skywalking-ecs-p001.shL.XXXX.net ansible_host=172.21.0.115 testhostvar=1.1.1.1
skywalking-ecs-p002.shL.XXXX.net ansible_host=172.21.0.117 testhostvar=2.2.2.2
skywalking-ecs-p003.shL.XXXX.net ansible_host=172.21.0.116 testhostvar=3.3.3.3
```

分别为每个主机组的主机定义了一个`testhostvar`变量

直接通过执行来查看变量是否生效

```shell

# ansible sw -i example.ini -m debug -a 'msg="{{ testhostvar }}"'
skywalking-ecs-p003.shL.XXXX.net | SUCCESS => {
    "msg": "3.3.3.3"
}
skywalking-ecs-p001.shL.XXXX.net | SUCCESS => {
    "msg": "1.1.1.1"
}
skywalking-ecs-p002.shL.XXXX.net | SUCCESS => {
    "msg": "2.2.2.2"
}
```

### 主机组变量

直接通过在主机组定义后加`:vars`方式来定义

```ini
[sw]
skywalking-ecs-p001.shL.XXXX.net ansible_host=172.21.0.115 testhostvar=1.1.1.1
skywalking-ecs-p002.shL.XXXX.net ansible_host=172.21.0.117 testhostvar=2.2.2.2
skywalking-ecs-p003.shL.XXXX.net ansible_host=172.21.0.116 testhostvar=3.3.3.3

[sw:vars]
test_group_var="kafka"
```
通过`shell`模块执行一下

```shell
#ansible sw -i example.ini -m shell -a " echo {{ test_group_var }}"
skywalking-ecs-p003.shL.XXXX.net | CHANGED | rc=0 >>
kafka
skywalking-ecs-p001.shL.XXXX.net | CHANGED | rc=0 >>
kafka
skywalking-ecs-p002.shL.XXXX.net | CHANGED | rc=0 >>
kafka
```


### 内置变量

1. 获取`ansible`的版本信息变量
```yaml

---
- hosts: sw
  remote_user: root
  tasks:
  - name: print ansible version
    debug:
      msg: "ansible version is {{ ansible_version }}"

```
执行
```shell
# ansible-playbook -i example.ini test31.yaml

PLAY [sw] ********************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]

TASK [print ansible version] *************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "ansible version is {'major': 2, 'full': '2.9.27', 'string': '2.9.27', 'minor': 9, 'revision': 27}"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "ansible version is {'major': 2, 'full': '2.9.27', 'string': '2.9.27', 'minor': 9, 'revision': 27}"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "ansible version is {'major': 2, 'full': '2.9.27', 'string': '2.9.27', 'minor': 9, 'revision': 27}"
}

PLAY RECAP *******************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

2. 获取`ansible`控制的主机组所有的主机名(主机变量属于hostvars,因此要将gather_facts设置为yes)

```yaml
---
- hosts: sw
  remote_user: root
  # gather_facts: no
  tasks:
  - name: print ansible remote machine ipaddress
    debug:
      msg: "ansible hostname is {{ ansible_fqdn }}"
```
执行

```shell
PLAY [sw] ********************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]

TASK [print ansible remote machine ipaddress] ********************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net] => {
    "msg": "ansible hostname is skywalking-ecs-p001.shL.XXXX.net"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": "ansible hostname is skywalking-ecs-p002.shL.XXXX.net"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": "ansible hostname is skywalking-ecs-p003.shL.XXXX.net"
}

PLAY RECAP *******************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

3. inventory_hostname变量

先看一下主机资源清单文件
```ini
[sw:children]
sw1
sw2

[sw:vars]
test_group_var="kafka"

[sw1]
skywalking-ecs-p002.shL.XXXX.net ansible_host=172.21.0.117
[sw2]
skywalking-ecs-p003.shL.XXXX.net ansible_host=172.21.0.116
```


```shell
# ansible sw -i example.ini -m debug -a "msg={{ inventory_hostname }}"
skywalking-ecs-p002.shL.XXXX.net | SUCCESS => {
    "msg": "skywalking-ecs-p002.shL.XXXX.net"
}
skywalking-ecs-p003.shL.XXXX.net | SUCCESS => {
    "msg": "skywalking-ecs-p003.shL.XXXX.net"
}
```

当我们用不同风格的主机清单文件，它的`inventory_hostname`显示的是不同的样式

```ini
[sw:children]
sw1
sw2

[sw:vars]
test_group_var="kafka"

[sw1]
172.21.0.117
[sw2]
172.21.0.116
```

执行

```shell
# ansible sw -i example1.ini -m debug -a "msg={{ inventory_hostname }}"
172.21.0.117 | SUCCESS => {
    "msg": "172.21.0.117"
}
172.21.0.116 | SUCCESS => {
    "msg": "172.21.0.116"
}
```

---

4. `play_hosts`
  
> 该内置变量可以获取到当前`play`所操作的所有主机的主机名列表

```yaml
---
- hosts: sw1,sw2
  remote_user: root
  gather_facts: true
  tasks:
  - name: debug hosts
    debug:
      msg: "{{ play_hosts }}"
```
执行结果

```shell
PLAY [sw1,sw2] ***************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [debug hosts] ***********************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => {
    "msg": [
        "skywalking-ecs-p002.shL.XXXX.net",
        "skywalking-ecs-p003.shL.XXXX.net"
    ]
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => {
    "msg": [
        "skywalking-ecs-p002.shL.XXXX.net",
        "skywalking-ecs-p003.shL.XXXX.net"
    ]
}

PLAY RECAP *******************************************************************************************************************************************************
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

5. 