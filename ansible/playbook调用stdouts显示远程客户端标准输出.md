# playbook调用stdouts显示远程客户端标准输出

如果采用ansible中的playbook方法进行远程客户端服务器的操作，需要对于远程客户端终端的返回数据进行抓取和进一步分析，就需要得到playbook方法的返回信息，其实也就是stdout的值。

如果采用Runner方法，他会自动返回对应的数据，下面是一个标准的输出结果

```bash
ok: [192.168.101.110] => (item=demoStop.results) => {
    "item": "demoStop.results", 
    "var": {
        "demoStop": {
            "changed": true, 
            "cmd": "/data/server/eric/demo.sh stop", 
            "delta": "0:00:10.062006", 
            "end": "2015-10-13 16:40:13.367927", 
            "invocation": {
                "module_args": "/data/server/eric/demo.sh stop", 
                "module_name": "shell"
            }, 
            "rc": 0, 
            "start": "2015-10-13 16:40:03.305921", 
            "stderr": "", 
            "stdout": "stop finished", 
            "stdout_lines": [
                "stop finished"
            ], 
            "warnings": []
        }
    }
}
```

对于以上的结果，如果我们只是想得到部分我们需要的内容，就需要自定义的输出

输出定义
定义输出的格式通过yml中的debug来完成。以下是我只是定义的只是获取stdout的内容

修改playbook

```bash
- name: Demo Proc Stop
  shell: /data/server/eric/demo.sh stop
  register: demoStop

- name: Process Status
  debug: var=demoStop.stdout
  with_items: demoStop.results
```

上面的方法是通过将demoStop.stdout赋予var变量，后面再由demoStop.results输出var的值来完成

以下为输出的结果

```bash
ok: [192.168.101.110] => (item=demoStop.results) => {
    "item": "demoStop.results", 
    "var": {
        "demoStop.stdout": "stop finished"
    }
}
```