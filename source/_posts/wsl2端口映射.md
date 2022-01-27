---
title: wsl2端口映射
date: 2022-01-05 17:41:19
tags:
---

最近在和同事协调工作的时候需要让同事通过局域网访问我的后端服务, 然而我的服务是放在wsl2里面的, 从局域网内除了本机之外的机器都没办法直接访问到wsl2中的服务. 所以想到做一下端口映射或者转发.

## wsl2端口映射
打开带有管理员权限的powershell, 我们需要先获取本机局域网内ip和wsl2的ip. 
```powershell
# 添加
netsh interface portproxy add v4tov4 listenport=${pc-port} listenaddress=0.0.0.0 connectport=${wsl2-port} connectaddress=${wsl2-ip}
netsh interface portproxy add v4tov4 listenport=9000 listenaddress=0.0.0.0 connectport=9000 connectaddress=172.20.70.33
# 删除
netsh interface portproxy delete v4tov4 lietenport=${pc-port}
netsh interface portproxy delete v4tov4 listenport=9000
# 查看
netsh interface portproxy show all
# 修改防火墙规则
netsh advfirewall firewall  add rule name="TCP 9000" protocol=TCP dir=in localport=9000 action=allow
```
