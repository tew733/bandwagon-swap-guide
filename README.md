# 搬瓦工 OpenVZ 为什么没有 Swap？KVM 怎么添加 Swap？默认 Swap 太小怎么解决——附搬瓦工全套餐配置对比

搬瓦工后台的 KiwiVM 面板里，有一条 Swap 状态条。第一次看到它变红，多少会有点慌——满了，然后呢？

先说结论：**搬瓦工现在所有在售套餐都是 KVM 架构，OpenVZ 已经完全退出历史舞台**。如果你搜"搬瓦工 OpenVZ swap"，搜到的大概率是五六年前的帖子，那时候 OpenVZ 用户想加 swap 是真的加不了——OpenVZ 这个虚拟化技术本身不支持。

KVM 不一样。Swap 大小不满意，完全可以自己调。下面把来龙去脉和操作步骤都说清楚。

---

## OpenVZ 和 Swap：为什么当年加不了

Swap，就是 Linux 的虚拟内存。物理内存不够用的时候，系统把暂时用不上的数据挪到硬盘上腾地方，这块硬盘空间就叫 Swap。

OpenVZ 是一种容器化虚拟技术，多个 VPS 共享同一个宿主机内核。这个架构决定了它不能让每个用户自己去划 Swap 分区——内核层面根本不给这个权限。所以搬瓦工 OpenVZ 时代，有些套餐给分配了一点 vSWAP（虚拟化 Swap），有些直接没有，用户自己是没法改的。

KVM 就完全不同了。每个 KVM 虚拟机有自己独立的内核，你有 root 权限，想建多大的 Swap 文件就建多大。

搬瓦工在 2018 年底宣布 OpenVZ 方案全线停售，不再续费，此后全部迁移到 KVM 架构。现在在官网买到的，不管哪个价位、哪条线路，全都是 KVM。

---

## 搬瓦工 KVM 默认 Swap 有多大

不大。

入门 KVM 套餐默认分配的 Swap 大概在 130MB 到 256MB 之间，具体看套餐配置。1GB 内存的机器配这点 Swap，跑个 Nginx + MySQL + PHP，流量稍微大一点就会看到 KiwiVM 面板里 Swap 那条变红。

红了不代表系统坏了。Swap 满了，系统会自动清掉旧的缓存数据来腾位置——只要内存本身没真的溢出，Swap 满着其实不影响正常运行。但如果你的程序对内存压力比较敏感，还是建议扩一下。

---

## 搬瓦工 KVM 添加 Swap：完整步骤

以下操作需要 root 权限，SSH 登录之后执行。

**1. 先查当前 Swap 状态**

bash
free -h


输出里 Swap 那行，total 那列就是当前分配的大小。

**2. 关闭并删除原来的 Swap（如果已有）**

bash
swapoff -a
rm -f /root/swapfile


**3. 创建新的 Swap 文件（以 1GB 为例）**

bash
dd if=/dev/zero of=/root/swapfile bs=1M count=1024


`bs` 是每块大小，`count` 是块数量，两者相乘就是总大小。要 2GB 就把 count 改成 2048，以此类推。

**4. 设置权限、格式化、启用**

bash
chmod 600 /root/swapfile
mkswap /root/swapfile
swapon /root/swapfile


**5. 设置开机自动挂载**

打开 `/etc/fstab`，在末尾加一行：


/root/swapfile swap swap defaults 0 0


**6. 验证**

bash
free -h


Swap 那行 total 变成你设置的大小，就成了。

---

通俗总结一下：先查现状，再关掉旧的，用 `dd` 建个新文件，格式化成 swap 格式，启用，写进 fstab 让它开机自动生效。六步，没有什么弯弯绕。

---

## Swap 该设多大？

一个粗略的参考：

- 内存 1GB 以内 → Swap 设 2-4 倍，也就是 2-4GB
- 内存 1-4GB → Swap 设 1-2 倍
- 内存 4GB 以上 → Swap 设 1 倍或更少，Swap 本身比物理内存慢很多，内存够的时候用不着太大的 Swap

搬瓦工默认给的那百来 MB，在 1GB 内存的机器上确实有点小。如果你跑 WordPress 或者部署了几个服务，建议直接拉到 2GB。

---

## 搬瓦工当前全套餐对比

搬瓦工目前在售套餐分四个系列：KVM 常规方案、CN2 GIA-E 方案、香港 CN2 GIA 方案、日本 CN2 GIA 方案。

### KVM 常规方案（入门首选）

| 套餐 | 内存 | 硬盘 | 流量/月 | 带宽 | 价格 | 购买 |
|---|---|---|---|---|---|---|
| KVM A | 1GB | 20GB SSD | 1TB | 1Gbps | $49.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=57) |
| KVM B | 2GB | 40GB SSD | 2TB | 1Gbps | $52.99/半年，$99.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=58) |

### CN2 GIA-E 方案（大多数人的首选）

| 套餐 | CPU | 内存 | 硬盘 | 流量/月 | 带宽 | 价格 | 购买 |
|---|---|---|---|---|---|---|---|
| GIA-E 1G | 2核 | 1GB | 20GB SSD | 1TB | 2.5Gbps | $49.99/季，$169.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=87) |
| GIA-E 2G | 3核 | 2GB | 40GB SSD | 2TB | 2.5Gbps | $89.99/季，$299.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=88) |
| GIA-E 4G | 4核 | 4GB | 80GB SSD | 3TB | 2.5Gbps | $549.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=89) |
| GIA-E 8G | 6核 | 8GB | 160GB SSD | 5TB | 5Gbps | $879.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=90) |
| GIA-E 16G | 8核 | 16GB | 320GB SSD | 8TB | 5Gbps | $1599.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=91) |
| GIA-E 32G | 10核 | 32GB | 640GB SSD | 10TB | 10Gbps | $2759.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=92) |
| GIA-E 64G | 12核 | 64GB | 1280GB SSD | 12TB | 10Gbps | $5399.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=93) |

CN2 GIA-E 支持 11 个机房随时切换，包括 DC6 CN2 GIA-E、DC9 CN2 GIA、日本大阪软银等。这是目前性价比最均衡的一档。

### 香港 CN2 GIA 方案（极致低延迟）

| 套餐 | CPU | 内存 | 硬盘 | 流量/月 | 带宽 | 价格 | 购买 |
|---|---|---|---|---|---|---|---|
| 香港 2G | 2核 | 2GB | 40GB SSD | 500GB | 1Gbps | $89.99/月，$899.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=95) |
| 香港 4G | 4核 | 4GB | 80GB SSD | 1TB | 1Gbps | $159.99/月，$1559.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=96) |
| 香港 8G | 6核 | 8GB | 160GB SSD | 2TB | 1Gbps | $299.99/月，$2999.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=97) |
| 香港 16G | 8核 | 16GB | 320GB SSD | 4TB | 1Gbps | $589.99/月，$5899.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=98) |

香港机房不可迁移，但延迟是搬瓦工所有机房里最低的，到大陆通常在 10ms 以内。价格是天花板，适合对延迟有硬性要求的场景。

### 日本 CN2 GIA 方案（移动用户友好）

| 套餐 | CPU | 内存 | 硬盘 | 流量/月 | 带宽 | 价格 | 购买 |
|---|---|---|---|---|---|---|---|
| 日本 2G | 2核 | 2GB | 40GB SSD | 500GB | 1Gbps | $89.99/月，$899.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=108) |
| 日本 4G | 4核 | 4GB | 80GB SSD | 1TB | 1Gbps | $159.99/月，$1559.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=109) |
| 日本 8G | 6核 | 8GB | 160GB SSD | 2TB | 1Gbps | $299.99/月，$2999.99/年 |  [选择此方案](https://bandwagonhost.com/aff.php?aff=80238&pid=110) |

👉 [查看搬瓦工当前所有套餐与实时库存](https://bit.ly/BanWaGon)

---

## Swap 加完之后还是不够用怎么办

说实话，Swap 是应急措施，不是银弹。如果你的 VPS 经常跑满内存，加 Swap 能缓解，但根本问题还是内存不足。

Swap 比物理内存慢不是一点点——读写速度差了一个数量级。程序频繁从 Swap 里换入换出，CPU 会跟着飙，整体体验会很糟糕。

这种情况下，换内存更大的套餐是更直接的解决方式。30 天内不满意可以全额退款，先试试没有风险。

👉 [以 $49.99/年 起用搬瓦工 KVM，30 天退款保证](https://bit.ly/BanWaGon)

---

## 常见问题

**Q：我现在还在用搬瓦工 OpenVZ，可以加 Swap 吗？**

不能。OpenVZ 架构本身不支持用户自行添加 Swap 分区。而且搬瓦工已经停止了所有 OpenVZ 方案的销售和续费，如果你的机器是 OpenVZ，到期后就得迁移数据到 KVM 方案了。

**Q：搬瓦工 KVM 添加 Swap 会影响 SSD 寿命吗？**

有一定影响，但不大。Swap 是把硬盘当内存用，频繁读写会消耗 SSD 写入寿命。实际情况是，如果你的 VPS 需要频繁用到 Swap，说明内存本身不够用，升级内存才是治本之法。偶尔跑满用到 Swap，影响可以忽略不计。

**Q：swappiness 值该怎么设？**

`swappiness` 控制系统倾向于用 Swap 的程度，值越低越倾向于保留物理内存不动，只在迫不得已时才用 Swap。对于 VPS，设成 10-20 之间通常比较合理，比默认的 60 更保守。用 `cat /proc/sys/vm/swappiness` 查当前值，编辑 `/etc/sysctl.conf` 写入 `vm.swappiness=10` 后执行 `sysctl -p` 生效。

**Q：搬瓦工哪个套餐最适合建站？**

内存需求是关键。WordPress 加上数据库，1GB 内存勉强够，访问量一上来就吃紧。从舒适度来说，CN2 GIA-E 的 1GB 套餐（$169.99/年）是个不错的起点——线路好，机房多，Swap 不够用了自己加也方便。

**Q：Swap 文件路径放哪里比较好？**

`/root/swapfile` 或者 `/var/swapfile` 都行，没有强制要求。放在磁盘根目录的位置通常性能略好一点，因为文件系统碎片少。权限记得设 600，否则系统会拒绝启用。

---

如果你是因为 OpenVZ 时代的帖子搜到这里，直接跳过那些"OpenVZ 怎么加 Swap"的旧教程吧——那条路走不通。现在搬瓦工全是 KVM，Swap 的问题用上面的步骤就能解决，五分钟搞定。

👉 [前往搬瓦工选购 KVM 方案，轻松管理 Swap](https://bit.ly/BanWaGon)
