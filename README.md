### # 系统推荐使用Ubuntu22.04 X64

### # 推荐使用原版BBR加速

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
chmod +x tcp.sh
./tcp.sh
```

### # 编译与安装Caddy
```
apt install golang-go
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
~/go/bin/xcaddy build --with github.com/caddyserver/forwardproxy@caddy2=github.com/klzgrad/forwardproxy@naive
```

### # 配置Caddy
在Caddy同目录下新建文件Caddyfile，并填入以下内容

```
:443, example.com #你的域名
tls example@example.com #你的邮箱
route {
 forward_proxy {
   basic_auth user pass #用户名和密码
   hide_ip
   hide_via
   probe_resistance
  }
 #支持多用户
 forward_proxy {
   basic_auth user2 pass2 #用户名和密码
   hide_ip
   hide_via
   probe_resistance
  }
 reverse_proxy  https://demo.cloudreve.org  { #伪装网址
   header_up  Host  {upstream_hostport}
   header_up  X-Forwarded-Host  {host}
  }
}
```

### # 配置Caddy为自启动服务
https://github.com/klzgrad/naiveproxy/wiki/Run-Caddy-as-a-daemon

移动Caddy和配置文件到指定目录
```
chmod +x caddy
mv caddy /usr/bin/
mkdir /etc/caddy
mv Caddyfile /etc/caddy/
```

测试移动后是否能正常工作
```
/usr/bin/caddy run --config /etc/caddy/Caddyfile
```

创建组和用户
```
groupadd --system caddy

useradd --system \
  --gid caddy \
  --create-home \
  --home-dir /var/lib/caddy \
  --shell /usr/sbin/nologin \
  --comment "Caddy web server" \
  caddy
  ```
  
  在/etc/systemd/system/下创建caddy.service文件，并填入以下内容
  ```
  [Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
启动Caddy
```
systemctl daemon-reload
systemctl enable caddy
systemctl start caddy
```

查看Caddy运行状态
```
systemctl status caddy
```

重新加载配置文件
```
systemctl reload caddy
```
