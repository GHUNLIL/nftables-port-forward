# nftables 端口转发 (nftables-port-forward)

`ix.sh` —— 基于 nftables 的全端口转发（DNAT + masquerade）脚本，用于把本机收到的流量转发到另一台目标机器（中转 / IX 落地常用）。

## 用途

将本机 TCP/UDP 端口 `23-65535` 的流量整段 DNAT 转发到指定目标 IP，并做源地址伪装（masquerade），把本机当作一个透明中转节点。

## 工作内容

- 开启内核 IP 转发：写入 `/etc/sysctl.d/99-ipforward.conf`
- 安装并启用 `nftables`
- 生成 `/etc/nftables.conf`：
  - `prerouting`：TCP/UDP `23-65535` DNAT 到 `TARGET_IP`
  - `postrouting`：对目标流量 masquerade

## 使用方法

**先编辑脚本顶部的配置变量**，再运行：

```bash
# 修改 TARGET_IP 为目标机器 IP（默认占位符为「IX的IP」）
# 如需调整端口范围，改 PORT_START / PORT_END
sudo bash ix.sh
```

## 依赖与环境

- 权限：**必须 root**
- 依赖：`nftables`（脚本会尝试用 apt/yum 安装）

## 注意事项

- 默认转发 `23-65535` 几乎全部端口，**会把本机除低端口外的入站流量都转给目标机**；如只需特定端口，请缩小 `PORT_START`/`PORT_END`
- `flush ruleset` 会清空现有 nftables 规则，若机器上已有其他防火墙规则请先备份
- 务必把 `TARGET_IP` 改成真实 IP，否则规则无效
