# WireGuard 使用教程

WireGuard 是新一代的 VPN 协议，特点是简单安全，总共只有几千行代码，由于过于优秀，已经被吸收进了 Linux5.6 的内核中。

## 安装

由于我们现在使用的内核普遍还在 4.x，所以还是需要安装下 WireGuard 的。以 Ubuntu 18.04 为例:

```sh
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt-get update
sudo apt-get install wireguard
```

## 网络规划

我们使用 `10.100.0.0/16` 作为 VPN 的子网网段。服务器所在的内网网段为：`172.17.0.11/20`。服务器的公网 IP 为 `1.1.1.1`。

WireGuard 的原理是生成一个虚拟网卡，一般来说是 `wg0`，然后通过这个虚拟网卡通信。

WireGuard 的配置文件是 ini 风格的，每个配置文件都有 `[Interface]` 和 `[Peer]` 两个部分。

其中 Interface 就是自身的接口配置，比如说对于服务器来说，需要配置 ListenPort，对于服务端和客户端来说都需要配置自己的 Address。Privatekey 填写自己的私钥内容。

Peer 这部分就比较有意思了，Peer 指的是 WireGuard 连接的另一端，对于服务器来说，Peer 就是客户端，而对于客户端来说，Peer 自然就是服务器。一个配置文件中可以有多个 Peer，这样的话，服务端就可以有多个客户端连接了。Peer 中可以指定 AllowedIPs，也就是允许连接的 IP。比如说，客户端指定了 `10.100.0.0/16,172.17.0.11/20`，那么就只有这部分 IP 的流量会通过 VPN 路由，而如果指定了 `0.0.0.0/0`，那么就是所有的流量通过 VPN 了。所以如果你只是想访问内网，也就是远程办公的话，那么可以选择前一种方法，如果是想通过国外主机访问某些受限的信息的话，当然要使用后一种了。Publickey 填写对方的公钥内容。

## 服务器配置

首先，生成 WireGuard 两对公钥和私钥，分别是服务器和客户端的：

```
wg genkey | tee server_privatekey | wg pubkey > server_publickey
wg genkey | tee client_privatekey | wg pubkey > client_publickey
```

这时候就得到了 privatekey 和 publickey 这四个文件。

wireguard 的配置都在 `/etc/wireguard` 目录下，在服务器上编辑 `/etc/wireguard/wg0.conf` 并输入一下内容。

```ini
[Interface]
Address = 10.100.0.1/16  # 这里指的是使用 10.100.0.1，网段大小是 16 位
SaveConfig = true
ListenPort = 51820  # 监听的 UDP 端口
PrivateKey = < 这里填写 Server 上 privatekey 的内容 >
# 下面这两行规则允许访问服务器的内网
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Client，可以有很多 Peer
[Peer]
PublicKey = < 这里填写 Client 上 publickey 的内容 >
AllowedIPs = 10.100.0.2/32  # 这个 Peer 只能是 10.100.0.2
# 如果想把所有流量都通过服务器的话，这样配置：
# AllowedIPs = 0.0.0.0/0, ::/0
```

然后，我们可以使用 `wg-quick up wg0` 启动 wg0 这个设备了。

通过 `wg show` 可以看到:

```
-> % sudo wg show
interface: wg0
  public key: xxxxxx
  private key: (hidden)
  listening port: 51820

peer: xxxxx
  allowed ips: 10.100.0.2/32
```

## 客户端配置

在客户端新建文件，可以放到 `/etc/wireguard/wg0.conf` 中，也可以随便放哪儿，用客户端读取就行。我这里使用的是 Mac 官方客户端，在 Mac AppStore 里可以找到。

```ini
[Interface]
PrivateKey = < 这里填写 Client 上 privatekey 的内容 >
Address = 10.100.0.2/32
DNS = 8.8.8.8

[Peer]
PublicKey = < 这里填写 Server 上 publickey 的内容 >
Endpoint = 1.1.1.1:51820  # 服务端公网暴露地址，51280 是上面指定的
AllowedIPs = 10.100.0.0/16,172.17.0.11/20  # 服务器可以相应整个 VPN 网段以及服务器的内网
PersistentKeepalive = 25
```

我们在客户端中点击导入，然后选择刚刚的文件，然后就可以连接啦：

![](images/wireguard.png)

可以使用内网地址访问，当然可以使用 172.17.0.11：

![](images/wireguard2.png)

## 创建服务

在服务器上，我们可以使用 systemd 开启 WireGuard 服务，重启后就不用再配置了。

```sh
sudo systemctl enable wg-quick@wg0.service
```

## 参考

1. https://www.reddit.com/r/WireGuard/comments/d524bj/only_route_traffic_for_ip_range_through_vpn/
2. https://github.com/pirate/wireguard-docs
3. https://www.stavros.io/posts/how-to-configure-wireguard/
4. https://lists.zx2c4.com/pipermail/wireguard/2017-May/001392.html