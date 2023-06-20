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