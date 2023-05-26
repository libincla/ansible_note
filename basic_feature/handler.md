# 介绍最基本的功能 handlers

## 概览

1. 什么是`handlers`以及使用场景
2. `handlers`的执行顺序
3. 如何在一个`task`里执行多个`handlers`

## 什么情况下使用handlers

举个例子，有个应用场景，涉及到两步task操作
1. taskA: 修改配置文件
2. taskB: 当配置文件修改成功后，重启nginx(以nginx为例)
   
以`ansible`正常的执行步骤来看，以上执行结果无非两种结果
1. taskA做修改，返回changed，没有问题，执行taskB，重启应用
2. taskA没有做修改，返回skipping，没有问题，执行taskB，重启应用

但是我们想实现这种效果，只有当`taskA`时做了修改，才会执行`taskB`，重启应用; `taskA`不做修改，不会执行`taskB`

于是`handlers`就是来解决这个问题，`handlers`看起来像一种`tasks`，一种任务列表。
简而言之，只有`tasks`中的任务造成了实际的改变(changed)，`handlers`中被调用的任务才会执行；`tasks`中任务没有做任何改变实际操作，`handlers`也不会执行

## 简单示例1， 通过文件改变触发handlers中的动作


```yaml

---

- hosts: sw
  remote_user: root
  tasks:
  - name: modify configuration
    lineinfile:
      path=/etc/nginx/nginx.conf
      regexp="listen(.*)8080(.*)"
      line="listen\1 18080\2"
      backrefs=yes
      backup=yes
    notify:
      restart nginx

  handlers:
  - name: restart nginx
    systemd:
      name=nginx
      state=restarted

```

注意，这里把"="替换成":"的话，会报错

---

```shell
ansible-playbook -i inventory.ini test1.yaml

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]

TASK [modify configuration] *******************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]

RUNNING HANDLER [restart nginx] ***************************************************************************************************************************************
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

修改监听的端口从8080改成17777，只有当修改有效时，触发`handlers`逻辑，即重启`nginx`
再次执行，只要文件内容不改变，handlers中的动作就不会触发

```shell
ansible-playbook -i inventory.ini test1.yaml
PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]

TASK [modify configuration] *******************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]
ok: [skywalking-ecs-p003.shL.XXXX.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

handlers与tasks属于"平级"的，缩进相同


## 简单示例2， 定义多个handlers

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: mkdir1
    file:
      path=/opt/h1
      state=directory
    notify:
      h1
  - name: mkdir2
    file:
      path=/opt/h2
      state=directory
    notify:
      h2
  handlers:
  - name: h1
    file:
      path=/opt/h1/t1
      state=touch
  - name: h2
    file:
      path=/opt/h2/t2
      state=touch

```

结果输出如下

```shell

ansible-playbook -i inventory.ini test2.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p001.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [mkdir1] *********************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]

TASK [mkdir2] *********************************************************************************************************************************************************
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]
changed: [skywalking-ecs-p003.shL.XXXX.net]

RUNNING HANDLER [h1] **************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]

RUNNING HANDLER [h2] **************************************************************************************************************************************************
changed: [skywalking-ecs-p001.shL.XXXX.net]
changed: [skywalking-ecs-p003.shL.XXXX.net]
changed: [skywalking-ecs-p002.shL.XXXX.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.XXXX.net : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.XXXX.net : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

会发现它的执行顺序竟然是 mkdir1 --> mkdir2 --> h1 --> h2
下面引发出它的执行顺序问题

## handlers的执行顺序

> 默认情况下，所有定义的`tasks`执行完毕后，才会执行各个`handler`，并非执行完某个`task`后，立即执行`handlers`
> 如果需要在执行完某个`task`后立即执行对应的`handler`，需要使用`meta`模块


### 简单示例1， 通过meta模块来改变执行顺序

```yaml

---
- hosts: sw
  remote_user: root
  tasks:
  - name: mkdir d1
    file:
      path=/opt/d1
      state=directory
    notify: d1

  - meta: flush_handlers

  - name: mkdir d2
    file:
      path=/opt/d2
      state=directory
    notify: d2

  handlers:
  - name: d1
    file:
      path=/opt/d1/h1
      state=touch
  - name: d2
    file:
      path=/opt/d2/h2
      state=touch
      
```

执行顺序变成了 mkdir d1 --> d1 --> mkdir d2 --> d2

通过在加入一行` - meta: flush_handlers`，影响了这个执行顺序

`meta`任务是一种特殊的任务，它会影响`ansible`内部运行的方式


### 简单示例2，触发多个handler的动作

首先，我们要知道，多个`handler`的`name`相同时，只有一个`handler`会被执行

但是 我们可以通过将 多个handler合成一个类似“组”的概念，通过`listen`关键字，这个“组”内的所有`handler`都会被`notify`

通过一个示例来演示

```yaml

---

- hosts: sw
  remote_user: root
  tasks:
  - name: t1
    file:
      path=/opt/t1
      state=directory

    notify: handler group

  - name: trigger handlers
    meta: flush_handlers
  
  - name: t2
    shell: echo "hiiii"

  handlers:
  - name: h1
    listen: handler group
    file:
      path=/opt/t1/h1
      state=touch

  - name: h2
    listen: handler group
    file:
      path=/opt/t1/h2
      state=touch
```

执行结果

```shell

PLAY [sw] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [skywalking-ecs-p001.shL.vevor.net]
ok: [skywalking-ecs-p003.shL.vevor.net]
ok: [skywalking-ecs-p002.shL.vevor.net]

TASK [t1] *************************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.vevor.net]
changed: [skywalking-ecs-p001.shL.vevor.net]
changed: [skywalking-ecs-p002.shL.vevor.net]

RUNNING HANDLER [h1] **************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.vevor.net]
changed: [skywalking-ecs-p002.shL.vevor.net]
changed: [skywalking-ecs-p001.shL.vevor.net]

RUNNING HANDLER [h2] **************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.vevor.net]
changed: [skywalking-ecs-p001.shL.vevor.net]
changed: [skywalking-ecs-p002.shL.vevor.net]

TASK [t2] *************************************************************************************************************************************************************
changed: [skywalking-ecs-p002.shL.vevor.net]
changed: [skywalking-ecs-p001.shL.vevor.net]
changed: [skywalking-ecs-p003.shL.vevor.net]

PLAY RECAP ************************************************************************************************************************************************************
skywalking-ecs-p001.shL.vevor.net : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p002.shL.vevor.net : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.vevor.net : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

如示例所示，handler的`h1`和`h2`都属于 `handler group`这个组，当`notify`触发这个“组”，“组”里面的任务会按照编排顺序执行