# Supervisor示例

---

参考: http://www.supervisord.org/configuration.html

```
[program:siem_remote]
# 较低的优先级表示先启动后关闭，较高的优先级表示最后启动和先关闭的程序
priority=100
# 执行子程序之前会切换到此目录
directory=/root/siem_remote/
# 该程序启动时将运行的命令
command=python3 manage.py runserver 0:8001 --noreload
# 如果为真，这个程序将在supervisord启动时自动启动
autostart=true
# 程序在启动后，即保持RUNNING状态超过以下的秒数，认为启动成功
startsecs=10
# 在尝试启动程序时，supervisord允许的连续失败次数，然后放弃并将进程置于FATAL状态
startretries=3
# 进程在RUNNING状态下退出，supervisord是否应该自动重启进程
autorestart=true 
# 日志文件配置
stdout_logfile=/var/log/supervisor/siem_remote_stdout.log
stderr_logfile=/var/log/supervisor/siem_remote_stderr.log

```

