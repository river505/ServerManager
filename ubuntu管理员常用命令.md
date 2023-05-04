# ubuntu管理员常用的命令

------



# 用户管理

1. 增加用户
   1. ```undefined
      sudo adduser username
      ```
2. 删除用户
   1. ```undefined
      sudo userdel -r username
      ```

   2.  其中-r表示同时删除该用户的主目录和邮箱等其他相关文件
3. 查看本设备共有哪些用户
   1. ```undefined
      cat /etc/passwd | grep bash$
      ```

   2.  查看所有用户的的用户名和家目录

   3. ```undefined
      cat /etc/passwd | awk -F: '{print $1, $6}'
      ```

# 权限管理

1. 给予用户sudo权限
   1. ```Shell
      sudo usermod -aG sudo username
      ```
2. 收回用户sudo权限
   1. ```Shell
      sudo deluser username sudo
      ```
3. 查看拥有sudo权限用户有哪些
   1. ```Bash
      grep '^sudo:' /etc/group | cut -d: -f4
      ```

# 查看剩余空间大小

1. 查看所有磁盘剩余空间大小：
   1. ```undefined
      df -hl
      ```
2. 当前目录下还剩余多少空间
   1. ```undefined
      df -h .
      ```
3. 查看当前目录已经使用总大小及当前目录下一级文件或文件夹各自使用的总空间大小：
   1. ```undefined
      du -h --max-depth=1
      ```

# 修改目录的拥有者

1. 将文件夹filename的所有者改为用户<user>

```
sudo chown ``-``R  <``user>``  <filename>
```

-R ：递归文件夹内部的所有文件及文件夹

使用时，将<user>替换为自己的用户名，<filename>换成要修改权限的文件夹名字。

- # 防火墙管理

-  查看防火墙状态`ufw status`

-  启动防火墙 `ufw enable`

-  关闭防火墙 `ufw disable`

-  重启防火墙 `ufw reload`

-  放行一个端口 `ufw allow 7000`

-  添加放行一个协议端口策略 `ufw allow 7000/tcp`

# 设置定时任务

crontab命令用法：

1. `crontab -e`：编辑当前用户的计划任务列表（如果不存在，则将创建一个新的）。
2. `crontab -l`：列出当前用户的计划任务列表。
3. `crontab -r`：删除当前用户的计划任务列表。
4. `crontab -u username -e：`编辑指定用户的计划任务列表。
5. `crontab -u username -l`：列出指定用户的计划任务列表。
6. `crontab -u username -r`：删除指定用户的计划任务列表。

在编辑计划任务时，crontab命令使用的时间格式为：

```Plain
复制代码
*     *     *   *    *        command to be executed
-     -     -   -    -
|     |     |   |    |
|     |     |   |    +----- day of the week (0 - 6) (Sunday=0)
|     |     |   +------- month (1 - 12)
|     |     +--------- day of the month (1 - 31)
|     +----------- hour (0 - 23)
+------------- min (0 - 59)
```

例如，要在每天午夜运0点行一个脚本，可以在crontab文件中添加以下行：

```Plain
0 0 * * * /path/to/script.sh
```

这表示在每天的0点0分运行脚本`/path/to/script.sh`。

需要注意的是，在使用crontab命令时，命令和时间域之间空格分隔，每一行只能有一条计划任务，且时间域支持的最小单位为1分钟。

# 进程管理

1. 查看进程 `ps` ：ps 直接执行不带任何选项，只显示当前用户会话中打开的进程。
   1. a：显示当前终端下的所有进程信息，包括其他用户的进程。
   2. u：使用以用户为主的格式输出进程信息。
   3. x：显示当前用户在所有终端下的进程。
   4. -e：显示系统内的所有进程信息。
   5. -l：使用长（long）格式显示进程信息。
   6. -f：使用完整的（full）格式显示进程信息
2. 将以简单列表的形式显示**所有用户**进程信息 `ps aux` 
3. 将以简单列表的形式显示**当前用户**进程信息 `ps ux`
4. 列出指定用户的进程信息 `ps -u <user>`
5. 以长格式显示系统中的进程信息，包含更丰富的内容。PPID为父进程的PID `ps -elf`
6. 查看这个进程是什么： `ps aux | grep PIDNAME`
7. 杀死这个进程：`kill -9 PIDNAME`