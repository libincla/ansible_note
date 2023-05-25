# 初识`playbook`

## 一点点YAML的语法知识

1. 当使用`YAML`语法中冒号作为键值映射时，":"后面必须要有空格
2. 在`YAML`语法进行缩进时，不能使用`tab`键进行缩进，必须使用空格

## 背景

实际环境中，我们有很多需要重复做多次相同命令的操作.
为什么我们不给它写成一个`playbook`，写成一个脚本，定期/按需执行
这样`ansible`会按照`playbook`一步步的执行，最终达到我们的预期目的
这个剧本遵循`YAML`语法，`playbook`以`.yaml`或者`.yml`文件名当作后缀
让我们写一个最简单的`playbook`做示例，在远程主机上创建某个目录


### 简单配置和运行

```yaml

---
- hosts: sw
  remote_user: root
  tasks:
  - name: ping host
    ping:
  - name: create dir on /opt
    file:
      path: /opt/test_playbook
      state: directory

```

---

```ini
1. 使用三个"-"作为开始；在`YAML`语法中，"---"表示文档的开始
2. 使用"-"作为开头 (在`YAML`使用"-"中表示一个块序列的节点); hosts表示我们在要 inventory文本中的 sw主机组上执行操作
3. remote_user 表示我们在远程操作时，使用哪个用户来进行操作
4. tasks 表示我们要进行操作的任务列表，之后的行都属于`tasks`键值对儿中的值，之后的行都属于`tasks`任务列表中的任务
    整个列表一共有两个任务组成，每个任务以"-"开头，每个任务都有自己的名称，使用`name`关键字进行指定, 第一个任务使用`ping`模块，第二个任务使用`file`模块

```

执行`playbook`

```shell
ansible-playbook -i inventory.ini test1.yaml

PLAY [sw] ************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]

TASK [ping host] *****************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [create dir on /opt] ********************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]

PLAY RECAP ***********************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


```

如图所示，`playbook`返回来一些信息，这些信息是剧本运行的情况

1. PLAY [sw] 表示此次运行的`playbook`是针对`sw`这个主机组运行
2. 此次`PLAY`一共包含三个任务
   1. [Gathering Facts]  每次`PLAY`在执行时，都先执行这个默认任务，收集当前`PLAY`对应目标主机的相关信息
   2. [ping host] 
   3. [create dir on /opt] 
3. 任务返回的颜色
   1. 红色， 任务失败，如果没有设置`ignore_errors`为`true`，当前`PLAY`退出
   2. 黄色， 任务发生过改变
   3. 绿色， 任务并没有发生变化，一般有幂等性不会有变化发生
4. PLAY RECAP 当所有的`PLAY`执行完毕后，会出现`PLAY RECAP`报告信息


### 如何检查`playbook`中的语法错误以及模拟执行

1. 语法检测

```shell

ansible-playbook --syntax-check test1.yaml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
[WARNING]: Could not match supplied host pattern, ignoring: sw

playbook: test1.yaml  

```
若只返回了`playbook`的名称，表示没有语法错误

2. 模拟执行(并不会真正执行)

```shell
ansible-playbook -i inventory.ini --check test1.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [sw] ************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.vevor.net]
ok: [skywalking-ecs-p002.shL.vevor.net]
ok: [skywalking-ecs-p001.shL.vevor.net]

TASK [ping host] *****************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.vevor.net]
ok: [skywalking-ecs-p001.shL.vevor.net]
ok: [skywalking-ecs-p002.shL.vevor.net]

TASK [create dir on /opt] ********************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.vevor.net]
ok: [skywalking-ecs-p001.shL.vevor.net]
ok: [skywalking-ecs-p002.shL.vevor.net]

PLAY RECAP ***********************************************************************************************************************************************************
skywalking-ecs-p001.shL.vevor.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.vevor.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.vevor.net : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

模拟执行，之前的任务并不会真正执行，不会产生对应的结果。
