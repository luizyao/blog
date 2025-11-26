+++
title = "Fedora 42 构建 Windows 11 虚拟机"
date = "2025-11-07"
[taxonomies]
categories = ["VM"]
tags = ["Fedora", "KVM"]
+++

> Fedora uses the libvirt family of tools as its virtualization solution.

# 安装依赖

0. 检查 CPU 是否支持虚拟扩展。

```bash
❯ grep -E '^flags.*(vmx|svm)' /proc/cpuinfo
```

如果上述命令没有输出，说明你的主机不支持相关的虚拟化扩展。仍然可以使用 QEMU/KVM，但是模拟器会回退到软件虚拟化，这会慢很多。

1. 安装 `virtualization` 软件组。

```bash
❯ sudo dnf install @virtualization
```

2. 启动 `libvirtd` 服务，并设置自启。

```bash
❯ sudo systemctl start libvirtd

❯ sudo systemctl enable libvirtd
```

3. 检查 KVM 内核模块是否加载。

```bash
❯ lsmod | grep kvm
kvm_intel             544768  0
kvm                  1490944  1 kvm_intel
irqbypass              16384  1 kvm
```

如果输出有 `kvm_amd` 或者 `kvm_intel`，说明 KVM 配置正确。

# 参考资料

Ref: [How to Properly Install a Windows 11 Virtual Machine on KVM](https://sysguides.com/install-a-windows-11-virtual-machine-on-kvm)