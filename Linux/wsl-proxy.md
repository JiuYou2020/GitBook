# WSL2如何进行代理设置

参考博客[为WSL2.0设置本机代理的配置方案（通过隧道）_wsl 如何应用本机代理](https://blog.csdn.net/iftodayhappy/article/details/137236279)

官方解答https://learn.microsoft.com/en-us/windows/wsl/troubleshooting#networking-considerations-with-dns-tunneling

1. 把之前在.bashrc启动文件中配置http_proxy和https_proxy的逻辑删去，并且关闭 WSL

```shell
wsl --shutdown
```

2. 在我的Windows主机，去`C:\Users\你的名字\`下面新建一个`.wslconfig`，然后用记事本打开往里面放这些内容

如`C:\Users\jiuyou2020\.wslconfig`

```sh
[wsl2]
memory=8GB
processors=8
[experimental]
autoMemoryReclaim=gradual
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
sparseVhd=true
==================以下内容为解释，使用时请不要复制==================
[wsl2] 部分
这部分定义了WSL2实例的资源限制，包括内存和处理器。

memory=8GB
这个选项限制WSL2实例可以使用的最大内存为8GB。这有助于防止WSL2实例消耗过多的系统内存，从而影响主机操作系统的性能。

processors=8
这个选项限制WSL2实例可以使用的处理器核心数为8个。这有助于控制WSL2实例的CPU使用率，确保主机操作系统有足够的CPU资源。

[experimental] 部分
这部分包含一些实验性特性，可以提供额外的功能和优化。

autoMemoryReclaim=gradual
自动内存回收设置为“gradual”，意味着WSL2会逐步回收未使用的内存。这有助于优化内存使用，减少WSL2实例在不需要时占用的内存。

networkingMode=mirrored
网络模式设置为“mirrored”，这意味着WSL2实例的网络设置会镜像主机的网络设置。这可以简化网络配置和集成。

dnsTunneling=true
启用DNS隧道，允许WSL2实例通过主机的DNS设置进行DNS查询。这可以确保WSL2实例能够使用与主机相同的DNS配置。

firewall=true
启用防火墙功能，为WSL2实例增加一层安全保护，控制进出WSL2实例的网络流量。

autoProxy=true
启用自动代理检测，允许WSL2实例自动使用主机的代理设置。这对于在企业网络环境中使用WSL2特别有用。

sparseVhd=true
启用稀疏VHD，意味着WSL2的虚拟硬盘 (VHD) 会以稀疏文件的形式存在，仅占用实际使用的磁盘空间。这有助于节省磁盘空间。
```
3. 重启WSL