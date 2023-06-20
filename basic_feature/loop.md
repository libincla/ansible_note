## 介绍一下`ansible`的循环


### with_items

> 通过`with_items`关键字，将返回的列表。通过`item`循环获取列表中每条信息

```yaml
  yum:
    name: "{{ item }}"
    state: present
  with_items:
  - zip
  - unzip
  - tar
```

**循环自定义列表项**

也支持自定义列表项，并循环迭代它

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: debug var
    debug:
      msg: "{{ item }}"
    with_items:
    - 1
    - 2
    - 3
```

**循环kv**

```yaml
---
- hosts: sw
  remote_user: root
  tasks:
  - name: debug var
    debug:
      msg: "{{ item.k1 }}"
    with_items:
    - {"k1":"v1", "k2":"v2"} #支持这种格式
    - { k1: a, k2: b }  #同样也支持这种格式
```

执行

```shell
# ansible-playbook -i example.ini test39.yaml

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [debug var] ***************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item={u'k2': u'v2', u'k1': u'v1'}) => {
    "msg": "v1"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item={u'k2': u'b', u'k1': u'a'}) => {
    "msg": "a"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => (item={u'k2': u'v2', u'k1': u'v1'}) => {
    "msg": "v1"
}
ok: [skywalking-ecs-p003.shL.XXXX.net] => (item={u'k2': u'b', u'k1': u'a'}) => {
    "msg": "a"
}

PLAY RECAP *********************************************************************************************************************************************************************************
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

**在`tasks`文件中直接使用循环**

> 通过循环列表一次性创建三个文件

```yaml
---
- hosts: sw
  remote_user: root
  vars:
    file_list:
    - "/opt/m1"
    - "/opt/m2"
    - "/opt/m3"

  tasks:
  - name: create files via loop
    file:
      path: "{{ item }}"
      state: touch
    with_items: "{{ file_list }}"
```

通过执行

```shell
# ansible-playbook -i example.ini test40.yaml

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p003.shL.XXXX.net]
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [create files via loop] ***************************************************************************************************************************************************************
changed: [skywalking-ecs-p003.shL.XXXX.net] => (item=/opt/m1)
changed: [skywalking-ecs-p002.shL.XXXX.net] => (item=/opt/m1)
changed: [skywalking-ecs-p003.shL.XXXX.net] => (item=/opt/m2)
changed: [skywalking-ecs-p002.shL.XXXX.net] => (item=/opt/m2)
changed: [skywalking-ecs-p002.shL.XXXX.net] => (item=/opt/m3)
changed: [skywalking-ecs-p003.shL.XXXX.net] => (item=/opt/m3)

PLAY RECAP *********************************************************************************************************************************************************************************
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
skywalking-ecs-p003.shL.XXXX.net : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

每次`shell`模块执行后的返回值都会放入一个名叫`result`的序列中，这个`result`返回值，当模块中使用了循环，模块每次执行的返回值都会追加存放到`result`这个返回值中



此外，`ansible`还支持 `jinja2`语法，比如，`for`循环可以写成如下

```yaml
---
- hosts: sw
  remote_user: root 
  tasks:
  - name: execute shell 
    shell: "{{ item }}"
    loop:
    - "ls -l /tmp"
    - "ls -l /root"
    register:  shellreturnvalue
  - name: debug it
    debug:
      msg: 
        "{% for x in shellreturnvalue.results %}
            {{ x.stdout }}
        {% endfor %}"
```


## with_items、with_list以及loop

以一个例子来展示`with_items`、`with_list`以及`loop`的不同

**with_items**

```yaml
---
- hosts: sw
  remote_user: root
  tasks:
  - name: execute for
    debug:
      msg: "{{ item }}"
    with_items:
    - [ 1, 2, 3 ]
    - [ "a", "b" ]
```

执行结果(这里为了展示篇幅，通过`-l`只让一个主机生效)

```shell
# ansible-playbook -i example.ini test44.yaml -l sw1   

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [execute for] *************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=1) => {
    "msg": 1
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=2) => {
    "msg": 2
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=3) => {
    "msg": 3
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=a) => {
    "msg": "a"
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=b) => {
    "msg": "b"
}

PLAY RECAP *********************************************************************************************************************************************************************************
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

可以看出来，`with_items`将列表项每一项都拆出来了


**with_list**

```yaml

---
- hosts: sw
  remote_user: root
  tasks:
  - name: execute for
    debug:
      msg: "{{ item }}"
    with_list:
    - [ 1, 2, 3 ]
    - [ "a", "b" ]

```
执行结果

```shell
# ansible-playbook -i example.ini test44.yaml -l sw1

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [execute for] *************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[1, 2, 3]) => {
    "msg": [
        1,
        2,
        3
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[u'a', u'b']) => {
    "msg": [
        "a",
        "b"
    ]
}
```

with_list将item的最小范围缩小成一个[],因此可以看到执行时，出现item=[1, 2, 3] 以及 item=[u'a', u'b']


**loop**

```yaml
---
- hosts: sw
  remote_user: root
  tasks:
  - name: execute for
    debug:
      msg: "{{ item }}"
    loop:
    - [ 1, 2, 3 ]
    - [ "a", "b" ]
```

执行结果

```shell
# ansible-playbook -i example.ini test44.yaml -l sw1

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [execute for] *************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[1, 2, 3]) => {
    "msg": [
        1,
        2,
        3
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[u'a', u'b']) => {
    "msg": [
        "a",
        "b"
    ]
}

```

可以看出来，`loop`的效果和`with_list`相似


> 当处理简单的列表项时，`with_items`和`with_list`以及`loop`没有任何区别
> 但当出现了嵌套列表时，`with_items`会将这个嵌套列表展开，后循环处理所有元素，而`with_list`则不会展开列表，循环处理最外层的列表的每项


## with_flattened

> `with_flattened`可以实现将嵌套列表展开（拉平），逐项展示

直接看示例

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: execute for
    debug:
      msg: "{{ item }}"
    with_flattened:
    - [ [1, 2], 3, [ 4,5,6 ] ]
```]

执行结果

```shell
# ansible-playbook -i example.ini test45.yaml  -l sw1

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [execute for] *************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=1) => {
    "msg": 1
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=2) => {
    "msg": 2
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=3) => {
    "msg": 3
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=4) => {
    "msg": 4
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=5) => {
    "msg": 5
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=6) => {
    "msg": 6
}

PLAY RECAP *********************************************************************************************************************************************************************************
skywalking-ecs-p002.shL.XXXX.net : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```


## with_together

> `with_together`将多个列表的元素**对齐合并**

查看示例

```yaml
---

- hosts: sw
  remote_user: root
  tasks:
  - name: debug item
    debug:
      msg: "{{ item }}"
    with_together:
    - [ 1, 2, 3, 4]
    - [ "a", "b", "c", "d"]
```

查看执行示例

```shell
# ansible-playbook -i example.ini test46.yaml -l sw1

PLAY [sw] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net]

TASK [debug item] **************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[1, u'a']) => {
    "msg": [
        1,
        "a"
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[2, u'b']) => {
    "msg": [
        2,
        "b"
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[3, u'c']) => {
    "msg": [
        3,
        "c"
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[4, u'd']) => {
    "msg": [
        4,
        "d"
    ]
}
```

将上下两个列表相同项合并在一起了, 如果所合并项在某个列表中缺失，`ansible`会补了一个`null`进去，如下面的例子

```yaml

---
- hosts: sw
  remote_user: root
  tasks:
  - name: debug item
    debug:
      msg: "{{ item }}"
    with_together:
    - [ 1, 2, 3, 4]
    - [ "a", "b", "c", "d"]
    - [ "x", "y", "z"]   #可以看到 第三个列表，缺失一项，只有三个
```
这时候的执行结果就是

```shell

TASK [debug item] **************************************************************************************************************************************************************************
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[1, u'a', u'x']) => {
    "msg": [
        1,
        "a",
        "x"
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[2, u'b', u'y']) => {
    "msg": [
        2,
        "b",
        "y"
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[3, u'c', u'z']) => {
    "msg": [
        3,
        "c",
        "z"
    ]
}
ok: [skywalking-ecs-p002.shL.XXXX.net] => (item=[4, u'd', None]) => {
    "msg": [
        4,
        "d",
        null
    ]
}

```
