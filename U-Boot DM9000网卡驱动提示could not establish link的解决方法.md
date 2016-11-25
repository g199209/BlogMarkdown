title: U-Boot DM9000网卡驱动提示"could not establish link"的解决方法
date: 2016-06-26 21:37:18
tags: [Bootloader]
categories: 嵌入式

---

使用默认的DM9000网卡驱动进行网络操作时，好像都会提示"could not establish link"，而且感觉因此造成的延迟还很严重。在网上搜索了一下，解决方法是屏蔽`dm9000x.c`文件中下面这段代码：

<!--more-->

```
static int dm9000_init(struct eth_device *dev, bd_t *bd)
{
	// 不做修改

#if 0
// 屏蔽掉这段代码
	i = 0;
	while (!(dm9000_phy_read(1) & 0x20)) {	/* autonegation complete bit */
		udelay(1000);
		i++;
		if (i == 10000) {
			printf("could not establish link\n");
			return 0;
		}
	}
#endif

	// 不做修改
}
```

这样完全可以正常工作，不仅不会提示"could not establish link"，而且响应速度也得到了很大的提升。不过这段代码的作用究竟是什么还不清楚，将来有空了可以来仔细研究下。
