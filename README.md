# telegraf启动程序

> 用于管理启动  https://github.com/influxdata/telegraf

## 一、项目结构
```shell
$ tree telegraf-ctl
telegraf-ctl
├── README.md                     ## 说明文档
├── agent.conf                    ## 配置agent实例的配置文件
├── telegraf-ctl                      ## agent管理指令
├── configs                       ## 不同的agent实例的配置文件
│   ├── monitor.example.1.conf    ## agent配置文件
│   └── monitor.example.2.conf
├── lib
│   └── agent                     ## agent二进制客户端(需要把二进制agent存放与此,agent即是telegraf重命名)
├── logs                          ## 存放agent的log
│   ├── example1.log
│   └── example2.log
└── pids                          ## 存放agent的pid
    ├── example1.pid
    └── example2.pid
``` 

编译说明：
```shell
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build  -o telegraf-ctl
```

## 二、agent启动配置
> 配置文件示例：agent.conf
```vim
## agent.conf
[worker:example1]                               ## 名称为example1的agent
config_path = monitor.example.1.conf            ## 启动example1使用的配置文件(存放在configs目录下)
input_filter = disk:diskio                      ## example1的指标搜集插件
output_filter = kafka                           ## example1输出插件
log_path = example1.log                         ## example1的日志文件地址（存放在logs目录下）
pid_file = example1.pid                         ## example1的pid文件地址（存放在pids目录下）

[worker:example2]                               ## 名称为example2的agent
config_path = monitor.example.2.conf
input_filter = disk
output_filter = kafka
log_path = example2.log
pid_file = example2.pid
```

说明：
- 多个agent请配置多个`[worker:agent名称]`

## 三、帮助说明
```shell
$ ./telegraf-ctl help
Usage:
  agentctl [command]

Available Commands:
  help        Help about any command
  list        显示所有Agent的启动配置项
  start       启动Agent.
  status      检查Agent的运行状态.
  stop        关闭所有Agent， 或某一个Agent.

Flags:
  -h, --help   help for agentctl

Use "agentctl [command] --help" for more information about a command.
```

## 四、显示Agent启动配置项
> 示例如下：
```shell
$ ./telegraf-ctl list
Agent启动配置列表如下:
+----------+---------------------------------+--------------------------------+-------------+----------+-------------------+-------------------+
|   名称   |            项目路径             |            配置文件            |  输入插件   | 输出插件 |     日志文件      |      PID文件      |
+----------+---------------------------------+--------------------------------+-------------+----------+-------------------+-------------------+
| example1 | /Users/dong/Go/src/agent/ | configs/monitor.example.1.conf | disk:diskio | kafka    | logs/example1.log | pids/example1.pid |
| example2 | /Users/dong/Go/src/agent/ | configs/monitor.example.2.conf | disk        | kafka    | logs/example2.log | pids/example2.pid |
+----------+---------------------------------+--------------------------------+-------------+----------+-------------------+-------------------+
``` 

## 五、启动agent
> 示例(启动所有Agent)：
```shell
## ./telegraf-ctl start   或者 ./agentctl start all
$ ./agentctl start all
[2019-11-30 22:50:29]   info    Agent [example1] 启动成功！
[2019-11-30 22:50:29]   info    Agent [example2] 启动成功！
```

> 示例(启动某一个Agent)
```shell
$ ./telegraf-ctl start example1
[2019-11-30 22:52:14]   info    Agent [example1] 启动成功！
```

## 六、关闭agent
> 示例（关闭所有Agent）
```shell
## ./telegraf-ctl stop   或者 ./agentctl stop all
$ ./telegraf-ctl stop all
[2019-11-30 22:54:03]   info    Agent [example1] 已停止！
[2019-11-30 22:54:03]   info    Agent [example2] 已停止！ 
```

> 示例(关闭某一个Agent)
```shell
$ ./telegraf-ctl start example2
[2019-11-30 22:54:36]   info    Agent [example2] 已停止！
```

## 七、重启Agent
 > 示例（重启所有Agent）
```shell
## ./telegraf-ctl restart   或者 ./agentctl stop all
$ ./telegraf-ctl restart
[2019-12-02 09:29:11]   info    Agent [example1] 已停止！
[2019-12-02 09:29:11]   info    Agent [example2] 已停止！
[2019-12-02 09:29:12]   info    Agent [example1] 启动成功！
[2019-12-02 09:29:12]   info    Agent [example2] 启动成功！
```

> 示例(重启某一个Agent)
```shell
$ ./telegraf-ctl restart example2
[2019-12-02 09:36:12]   info    Agent [example2] 已停止！
[2019-12-02 09:36:13]   info    Agent [example2] 启动成功！
```
## 八、查看Agent状态
> 示例：
```shell
$ ./telegraf-ctl status
Agent运行状态如下:
+----------+------+----------+
|   名称   | PID  | 运行状态 |
+----------+------+----------+
| example2 | 9269 | 正常     |
| example1 | 9267 | 正常     |
+----------+------+----------+
```

```shell
$ ./telegraf-ctl status
Agent运行状态如下:
+----------+-----+----------+
|   名称   | PID | 运行状态 |
+----------+-----+----------+
| example1 | -   | 掉线     |
| example2 | -   | 掉线     |
+----------+-----+----------+
```
