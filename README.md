# tun2socks 透明代理
## 脚本依赖
- [脚本依赖 - 安装参考](https://www.zfl9.com/ss-redir.html#%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96)
- curl，获取大陆地址段列表
- ipset，保存大陆地址段列表
- iproute2，配置策略路由，透明代理
- haveged，防止系统出现熵过低的问题
- pdnsd，支持永久性缓存的 DNS 代理服务器
- chinadns，利用大陆地址段列表实现 DNS 分流
- SS、SSR 或其它支持 UDP Relay 的 socks5 代理

## 端口占用
> 请检查是否有端口被占用，如果有请自行解决！

- pdnsd：0.0.0.0:53/udp
- chinadns：0.0.0.0:65353/udp

## 脚本用法
**获取**
- `git clone https://github.com/zfl9/ss-tun2socks.git`

**安装**
- `cd ss-tun2socks`
- `cp -af ss-tun2socks /usr/local/bin/`
- `cp -af tun2socks.bin/tun2socks.ARCH /usr/local/bin/tun2socks`（先解压，注意 ARCH）
- `chown root:root /usr/local/bin/tun2socks /usr/local/bin/ss-tun2socks`
- `chmod +x /usr/local/bin/tun2socks /usr/local/bin/ss-tun2socks`
- `mkdir -m 0755 -p /etc/tun2socks`
- `cp -af pdnsd.conf /etc/tun2socks/`
- `cp -af chnroute.txt /etc/tun2socks/`
- `cp -af chnroute.ipset /etc/tun2socks/`
- `cp -af ss-tun2socks.conf /etc/tun2socks/`
- `chown -R root:root /etc/tun2socks`
- `chmod 0644 /etc/tun2socks/*`

**配置**
- `vim /etc/tun2socks/ss-tun2socks.conf`，修改开头的 `socks5 配置`。
- `socks5_listen="127.0.0.1:1080"`：socks5 监听地址，一般为 1080 端口。
- `socks5_remote="node.proxy.net"`：SS/SSR 服务器的 Hostname/IP，注意修改。
- `socks5_runcmd="nohup ss-local -c /etc/ss-local.json < /dev/null &>> /var/log/ss-local.log &"`<br>
启动 SS/SSR 的命令，此命令必须能够后台运行（即：不能占用前台）。<br>
如 `service [service-name] start`、`systemctl start [service-name]` 等。
- `chinadns_upstream="114.114.114.114,8.8.8.8"`：建议将 114 替换为原网络下的 DNS
- `iptables_intranet=(192.168.0.0/16)`：如果内网网段不是 192.168/16，请修改（多个使用空格隔开）
- `dns_original=(114.114.114.114 119.29.29.29 180.76.76.76)`：建议修改为原网络下的 DNS（最多 3 个）

**自启**（Systemd）
- `cp -af ss-tun2socks.service /etc/systemd/system/`
- `systemctl daemon-reload`
- `systemctl enable ss-tun2socks.service`

**自启**（SysVinit）
- `touch /etc/rc.d/rc.local`
- `chmod +x /etc/rc.d/rc.local`
- `echo "/usr/local/bin/ss-tun2socks start" >> /etc/rc.d/rc.local`

> 配置 ss-tun2socks 开机自启后容易出现一个问题，那就是必须再次运行 `ss-tun2socks restart` 后才能正常代理（这之前查看运行状态，可能看不出任何问题，都是 running 状态），这是因为 ss-tun2socks 启动过早了，且 socks5_remote 为 Hostname，且没有将 socks5_remote 中的 Hostname 加入 /etc/hosts 文件而导致的。因为 ss-tun2socks 启动时，网络还没准备好，此时根本无法解析这个 Hostname。要避免这个问题，可以采取一个非常简单的方法，那就是将 Hostname 加入到 /etc/hosts 中，如 Hostname 为 node.proxy.net，对应的 IP 为 11.22.33.44，则只需执行 `echo "11.22.33.44 node.proxy.net" >> /etc/hosts`。不过得注意个问题，那就是假如这个 IP 变了，别忘了修改 /etc/hosts 文件哦。命令行获取某个域名对应的 IP 地址的方法：`dig +short HOSTNAME`。

**用法**
- `ss-tun2socks help`：查看帮助
- `ss-tun2socks start`：启动代理
- `ss-tun2socks stop`：关闭代理
- `ss-tun2socks restart`：重启代理
- `ss-tun2socks status`：运行状态
- `ss-tun2socks current_ip`：查看当前 IP（一般为本地 IP）
- `ss-tun2socks flush_dnsche`：清空 dns 缓存（pdnsd 的缓存）
- `ss-tun2socks update_chnip`：更新大陆地址段列表（ipset、chinadns）

**日志**
> 如需详细日志，请打开 ss-tun2socks.conf 中相关的 verbose 选项。

- pdnsd：`/var/log/pdnsd.log`
- chinadns：`/var/log/chinadns.log`
- tun2socks：`/var/log/tun2socks.log`

## 相关参考
- [pdnsd](http://members.home.nl/p.a.rombouts/pdnsd/index.html)
- [ChinaDNS](https://github.com/shadowsocks/ChinaDNS)
- [badvpn](https://github.com/ambrop72/badvpn)
- [gotun2socks](https://github.com/yinghuocho/gotun2socks)
- [ss-tproxy 常见问题](https://www.zfl9.com/ss-redir.html#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
