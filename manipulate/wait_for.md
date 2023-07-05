# wait_for

> `wait_for`是`ansible-core`核心模块提供的模块
>


## 梗概

- 可以设置`timeout`，如果未设置任何内容或者仅指定`timeout`，不会产生错误
- 常用于，当一个`java`应用在初始化脚本后无法立即返回可用时，`wait_for`就变得非常有用
- 也常用于，当通过`community.libvirt.virt`模块启动一个`guest`来需要暂停一直到它准备好
- 也可以用于，等待文件中的内容被正则表达式规则所匹配 
- 在`Ansible 1.6`以及后面的版本，这个模块被用于判断文件系统的文件是否存在
- 在`Ansible 1.8`以及后面的版本，这个模块也被用于在继续之前等待活动连接关闭，以及节点在负载均衡池中轮换


## 参数
- `active_connection_states`: 被统计为活动连接(active connections)的`TCP`连接的列表， 接受参数类型为`string`或者`list`，默认值是["ESTABLISHED", "FIN_WAIT1", "FIN_WAIT2", "SYN_RECV", "SYN_SENT", "TIME_WAIT"]
- `connect_timeout`: 在关闭和重试之前等待连接发生的最大秒数，接受参数类型为`integer`，默认值是5秒
- `delay`: 在开始`poll`之前的描述，接受参数类型为`integer`，默认值是0秒(其实是类似delay+隔多少秒检测一次)
- `exclude_hosts`: 当查找`drained`状态的活动的`TCP`连接时，要忽略的主机或者`IP`列表，接受参数类型为`string`或者`list`
- `host`: 等待的可解析的主机名或者`IP`地址，接受参数类型为`string`，默认值是"127.0.0.1"
- `msg`: 当遇到需求条件时，这里的参数会覆盖常规的错误消息
- `path`: 文件系统中文件的路径
- `port`: 开始`poll`的端口号，接受参数类型为`integer`; `path`和`port`是互斥的参数
- `search_regex`: 用于在一个文件或者一个`socket`连接中匹配字符串，接受参数类型为`string`
- `sleep`: 检测之间的`sleep`的秒数，`Ansible 2.3`之前被硬核编码成1秒
- `state`: 检测的状态，可选参数为`present`, `started`, `stopped`,`absent`或者`drained`。当检查的是一个端口，`started`要确保这个端口是`open`的，`stopped`检查它是关闭的。`drained`检查活动连接数； 当检查的是一个文件或者搜索的是一个字符串，`present`或者`started`确保文件或者字符串在检测期间是存在的，`absent`检查文件不存在或者是被删除掉
- `timeout`: 等待的最大秒数，接受参数类型为`integer`。当与其他条件一起使用时，它会强制发生错误；当使用时没有其他选项，它等同于`sleep`，默认值是`300`秒


## 返回值

- `elapsed`: 等待共消耗了多少秒


## 实验

> 我们的`java`应用检测使用到了这个模块，但是参数一直用的很不规范，以下示例为了理解参数

### 场景1

> 使用`wait_for`给`java`应用设置一个通用的超时时间，比如120秒， 如果超时时间内应用启动了立即返回成功并退出

以下场景，用一个不存在的端口号(23)模拟长时间的java应用上线过程

**参数设置1: 端口不存在**
port=23  state=started

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 300,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 300,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
```

在此情况下，由于没有设置`timeout`，这里触发了`timeout`的默认值，300秒


**参数设置2: 端口存在，设置了`timeout`**
port=22  state=started timeout=2

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 0,
    "match_groupdict": {},
    "match_groups": [],
    "path": null,
    "port": 22,
    "search_regex": null,
    "state": "started"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 0,
    "match_groupdict": {},
    "match_groups": [],
    "path": null,
    "port": 22,
    "search_regex": null,
    "state": "started"
}
```

在此情况下，由于22号端口存在，虽然设置了`timeout`的时间为2秒，但是当端口号存在还是直接返回，没有等待时间

**参数设置3: 端口不存在，设置`timeout`**
port=23  timeout=2 state=started

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 2,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 2,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
```

在此情况下，由于23号端口不存在，设置了`timeout`的时间为2秒，2秒过程中，检测端口号一直不存在，直接返回错误

**参数设置4:  端口不存在，设置了`delay`，设置`timeout`小于`delay`**
delay=30 port=23 timeout=2 state=started

```shell
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 30,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 30,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
```
在此情况下，由于23号端口不存在，设置了`delay`检测为30秒，`timeout`为2秒，实际上`timeout`没有生效，`delay`了30秒后，发现端口没有监听，报告异常退出

**参数设置5: 端口不存在，设置了`delay`，并未设置`timeout`**
delay=30 port=23  state=started

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 300,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 300,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
```
在此情况下，由于23号端口不存在，设置了`delay`检测为30秒，但并未设置`timeout`的值，实际上应用还是等了300秒后，发现`delay`的值似乎并未生效，抑或是包含到`timeout`默认的`300`秒里面了


**参数设置6: 端口存在，设置了`delay`，设置`timeout`大于`delay`**
port=22  state=started timeout=200 delay=100

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 100,
    "match_groupdict": {},
    "match_groups": [],
    "path": null,
    "port": 22,
    "search_regex": null,
    "state": "started"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 100,
    "match_groupdict": {},
    "match_groups": [],
    "path": null,
    "port": 22,
    "search_regex": null,
    "state": "started"
}
```
在此情况下，由于22号端口存在，设置了`delay`检测为100秒, `timeout`的值大于`delay`，应用在延迟100秒后，返回成功

**参数设置7: 端口存在，设置了`delay`，设置`timeout`小于`delay`**
port=22  state=started timeout=100 delay=150

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 150,
    "msg": "Timeout when waiting for 127.0.0.1:22"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 150,
    "msg": "Timeout when waiting for 127.0.0.1:22"
}
```
在此情况下，发生了一个**bug**，22端口是通的，由于`delay`大于`timeout`，发生了错误


**参数设置8: 端口不存在，设置了`delay`，设置`timeout`等于`delay`**
port=23  state=started timeout=20 delay=20

```shell
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 20,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 20,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
```

在此情况下，由于23号端口不存在，分不出到底是`delay`还是`timeout`生效了


**参数设置9: 端口不存在，仅设置`timeout`**
port=23  state=started timeout=400

```shell
xxxx-xxxx-xxxx-xx1.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 400,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
xxxx-xxxx-xxxx-xx2.shL.xxxx.net | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "elapsed": 400,
    "msg": "Timeout when waiting for 127.0.0.1:23"
}
```
在此情况下，由于23号端口不存在，设置了`timeout`的值为400秒，它等待了400秒后发现端口不存在而退出
