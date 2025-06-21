# Kubo

[![PFM-Upstream-Sync](https://github.com/PFM-PowerForMe/kubo/actions/workflows/fork-sync.yml/badge.svg)](https://github.com/PFM-PowerForMe/kubo/actions/workflows/fork-sync.yml)

## 简介
IPFS网关

## 如何部署?


### 初始化节点
1. 运行
```
podman run -d --name kubo \
    -v kubo-export:/export \
    -v kubo-data:/data/ipfs \
    -p 4001:4001 \
    -p 4001:4001/udp \
    -p 127.0.0.1:8080:8080 \
    -p 127.0.0.1:5001:5001 \
    ghcr.io/pfm-powerforme/kubo:latest
```
2. 等待启动完成
```
podman logs kubo -f
```
3. 执行初始化
```
podman exec kubo ipfs swarm peers
```
4. 添加一些文件
```
echo "Hello IPFS" > test.txt && \
podman cp test.txt kubo:/export && \
podman exec kubo ipfs add -r /export/test.txt && \
rm test.txt


podman exec kubo ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["http://localhost:6012", "https://kubo.singularpoint.cc", "https://webui.ipfs.io"]'
podman exec kubo ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "POST"]'

```
5. 停止容器
```
podman stop kubo
```

### 使用 Podman

```
nvim /etc/containers/systemd/kubo.container && \
systemctl daemon-reload && \
systemctl restart kubo
```
```
# /etc/containers/systemd/kubo.container

[Unit]
Description=The kubo container
Wants=network-online.target
After=network-online.target

[Container]
AutoUpdate=registry
ContainerName=kubo
Timezone=local
UserNS=host
Network=deploy
Volume=kubo-export:/export
Volume=kubo-data:/data/ipfs
Environment=IPFS_PROFILE=server
Environment=GOMAXPROCS=2
Environment=GOMEMLIMIT=256MiB
PublishPort=4001:4001
PublishPort=4001:4001/udp
PublishPort=127.0.0.1:6011:8080
PublishPort=127.0.0.1:6012:5001
Image=ghcr.io/pfm-powerforme/kubo:latest

[Service]
Restart=on-failure
RestartSec=30s
StartLimitInterval=30
TimeoutStartSec=900
TimeoutStopSec=70
CPUQuota=50%
MemoryLimit=512M
CPUAccounting=yes

[Install]
WantedBy=multi-user.target default.target
```