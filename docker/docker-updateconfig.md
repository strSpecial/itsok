# docker 安装
1. docker 配置文件/etc/docker/daemon.json
```sh
sudo cat >/etc/docker/daemon.json<<EOF
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker

```
2. 启动docer服务异常查看服务日志:
```sh
journalctl -xeu docker.service
```