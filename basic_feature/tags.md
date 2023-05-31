# 介绍最基本的功能 tags

## 概览
1. 什么是tags，以及它的应用场景

## 什么场景下使用tags

正常情况下，运行这个`PLAY`剧本是自上而下执行的，如果**仅想执行其中一部分任务**，可以借助`tags`实现。

`tags`有以下特点
1. 当指定标签后，**只有标签对应的任务**会被执行，其他任务都不会被执行
2. 还可以通过指定哪些标签任务不执行(`--skip-tags`)

## 怎么使用tags

1. 语法格式

```yaml
语法1

tags:
- t1
- t2

语法2
tags: t1,t2

语法3
tags: ['t1', 't2']
```

2. 简单示例

```yaml

---

- hosts:  sw
  remote_user: root
  tasks:
  - name: execute t1
    file:
      path: /opt/tags1
      state: touch
    tags:
    - t1
  - name: execute t2
    file:
      path: /opt/tags2
      state: touch
    tags:
    - t2
```
可以通过`-t TAG_NAME` 来选择执行对应`tags`

```shell

ansible-playbook -i inventory.ini -t t1 test6.yaml

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXX.net]
ok: [skywalking-ecs-p001.shL.XXX.net]
ok: [skywalking-ecs-p002.shL.XXX.net]

TASK [execute t1] *****************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

以及可以通过`--skip-tags`来排除不执行的标签

```shell
ansible-playbook -i inventory.ini --skip-tags t1 test6.yaml

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXX.net]
ok: [skywalking-ecs-p001.shL.XXX.net]
ok: [skywalking-ecs-p002.shL.XXX.net]

TASK [execute t2] *****************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]
changed: [skywalking-ecs-p001.shL.XXX.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

### 多标签的使用

1. 指定多标签（只需在`-t`后面用逗号分隔多个标签名称即可）

```shell
ansible-playbook -i inventory.ini -t t1,t2 test6.yaml

---
TASK [execute t1] *****************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]

TASK [execute t2] *****************************************************************************************************************************************************
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]
changed: [skywalking-ecs-p003.shL.XXX.net]

```

### 特殊内置标签

`Ansible`中有5个特殊的`tag`，分别为

1. `Always`: 任务就会总被执行
2. `Never`
3. `tagged`
4. `untagged`
5. `all`

**`always`**

当我们把任务的`tags`值设置为`always`时，这个任务就会总被执行，除非使用`--skip-tags`明确指定不执行

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: execute job1
    file:
      path: /opt/job1
      state: touch

    tags:
    - job1

  - name: execute job2 always
    file:
      path: /opt/job2
      state: touch
    tags:
    - job2
    - always   # 注意，这里有个always， 我们除非使用`--skip-tags`
```

通过`-t job1`来执行这个任务，发现被标注`tag`的任务还会执行

```shell
ansible-playbook -i inventory.ini -t job1 test7.yaml

...
TASK [execute job1] ***************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]
changed: [skywalking-ecs-p001.shL.XXX.net]

TASK [execute job2 always] ********************************************************************************************************************************************
changed: [skywalking-ecs-p001.shL.XXX.net]
changed: [skywalking-ecs-p003.shL.XXX.net]
changed: [skywalking-ecs-p002.shL.XXX.net]
...

```

**never**


```yaml

---
- hosts: sw
  remote_user: root
  tasks:
  - name: job3 execute
    file:
      path: /opt/job3
      state: touch
    tags: job3

  - name: job4 never execute
    file:
      path: /opt/job4
      state: touch
    tags:
    - job4
    - never
```

测试一下
1. 当我们不指定标签的时候，它的执行流程是怎么样的
2. 我们通过标签选择后， 它的执行流程

**不加标签**的时候，它不会执行带有`never`标签的task

```shell

ansible-playbook -i inventory.ini test8.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.vevor.net]
ok: [skywalking-ecs-p002.shL.vevor.net]
ok: [skywalking-ecs-p001.shL.vevor.net]

TASK [job3 execute] ***************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.vevor.net]
changed: [skywalking-ecs-p002.shL.vevor.net]
changed: [skywalking-ecs-p001.shL.vevor.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.vevor.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.vevor.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.vevor.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

当我们**手动执行标签执行**的时候, 只要符合`tag`筛选条件还是会执行对应的逻辑

```shell
ansible-playbook -i inventory.ini -t job4 test8.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.vevor.net]
ok: [skywalking-ecs-p001.shL.vevor.net]
ok: [skywalking-ecs-p002.shL.vevor.net]

TASK [job4 never execute] *********************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.vevor.net]
changed: [skywalking-ecs-p001.shL.vevor.net]
changed: [skywalking-ecs-p002.shL.vevor.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.vevor.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.vevor.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.vevor.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

