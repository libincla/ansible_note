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