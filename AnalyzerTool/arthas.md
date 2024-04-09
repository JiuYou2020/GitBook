---
description: 与Arthas相关的一些使用技巧
---

# Arthas

```bash
PS D:\JAVA\Arthas> java -jar .\arthas-boot.jar -h
[INFO] JAVA_HOME: D:\JAVA\jdk-1.8\jre
[INFO] arthas-boot version: 3.7.2
Usage: arthas-boot [--session-timeout <value>] [--disabled-commands <value>]
       [--password <value>] [--select <value>] [--stat-url <value>] [--http-port
       <value>] [--use-version <value>] [--versions] [--use-http]
       [--attach-only] [--agent-id <value>] [--app-name <value>]
       [--tunnel-server <value>] [-f <value>] [--username <value>]
       [--telnet-port <value>] [--repo-mirror <value>] [-h] [--arthas-home
       <value>] [--target-ip <value>] [-c <value>] [--height <value>] [--width
       <value>] [-v] [pid]

Bootstrap Arthas

EXAMPLES:
  java -jar arthas-boot.jar <pid>
  java -jar arthas-boot.jar --telnet-port 9999 --http-port -1
  java -jar arthas-boot.jar --username admin --password <password>
  java -jar arthas-boot.jar --tunnel-server 'ws://192.168.10.11:7777/ws'
--app-name demoapp
  java -jar arthas-boot.jar --tunnel-server 'ws://192.168.10.11:7777/ws'
--agent-id bvDOe8XbTM2pQWjF4cfw
  java -jar arthas-boot.jar --stat-url 'http://192.168.10.11:8080/api/stat'
  java -jar arthas-boot.jar -c 'sysprop; thread' <pid>
  java -jar arthas-boot.jar -f batch.as <pid>
  java -jar arthas-boot.jar --use-version 3.7.2
  java -jar arthas-boot.jar --versions
  java -jar arthas-boot.jar --select math-game
  java -jar arthas-boot.jar --session-timeout 3600
  java -jar arthas-boot.jar --attach-only
  java -jar arthas-boot.jar --disabled-commands stop,dump
  java -jar arthas-boot.jar --repo-mirror aliyun --use-http
WIKI:
  https://arthas.aliyun.com/doc

Options and Arguments:
    --session-timeout <value>     The session timeout seconds, default 1800
                                  (30min)
    --disabled-commands <value>   disable some commands
    --password <value>            The password
    --select <value>              select target process by classname or
                                  JARfilename
    --stat-url <value>            The report stat url
    --http-port <value>           The target jvm listen http port, default 8563
    --use-version <value>         Use special version arthas
    --versions                    List local and remote arthas versions
    --use-http                    Enforce use http to download, default use
                                  https
    --attach-only                 Attach target process only, do not connect
    --agent-id <value>            The agent id register to tunnel server
    --app-name <value>            The app name
    --tunnel-server <value>       The tunnel server url
 -f,--batch-file <value>          The batch file to execute
    --username <value>            The username
    --telnet-port <value>         The target jvm listen telnet port, default
                                  3658
    --repo-mirror <value>         Use special remote repository mirror, value is
                                  center/aliyun or http repo url.
 -h,--help                        Print usage
    --arthas-home <value>         The arthas home
    --target-ip <value>           The target jvm listen ip, default 127.0.0.1
 -c,--command <value>             Command to execute, multiple commands
                                  separated by ;
    --height <value>              arthas-client terminal height
    --width <value>               arthas-client terminal width
 -v,--verbose                     Verbose, print debug info.
 <pid>                            Target pid
```
