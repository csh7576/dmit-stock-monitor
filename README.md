# DMIT 库存监控：盯住洛杉矶/香港/东京的补货，不用手刷 F5 也能抢到合适的套餐

DMIT 的套餐不难选，难的是在你还想买的时候它还在。热门配置上架几个小时就变灰，而补货时间没有任何规律——可能是工作日上午，也可能在你睡着的时候。所以"DMIT 库存监控"这件事才会被反复搜：真正要解决的不是选哪台，而是怎么在它还有货的那一刻知道。

这篇文章把几件事说清楚：DMIT 现在的库存状况为什么这么紧、有哪些现成的监控渠道、自建监控长什么样、以及盯到有货之后该怎么选。

## DMIT 的库存为什么值得专门盯

2026 年这一轮紧张不是错觉。据蓝点网的报道，内存等硬件成本上涨后，DMIT 与 ByteVirt 等商家都对洛杉矶 CN2 GIA 服务器做了涨价调整，DMIT 当时直接暂停销售 LAX AN5 系列，用 AN4 系列过渡 [1]。随后 AN4 的硬件资源也耗尽，官网一度关闭销售 [2]。

再往后是 AN5 重新上架，价格随之上调。多个渠道的记录显示，洛杉矶 AN5 的入门款 LAX.AN5.Pro.TINY 从原先的 $9.99/月变成 $12.98/月，年付门槛从 88 美元升到 119.99 美元 [3]。日本那边，TYO 的 Eyeball 系列已经下架 [4]。

换句话说，DMIT 现在是"上新—售罄—调价—再上架"的循环，而不是常年稳定铺货。同一款套餐在不同月份的价格可能差 30%，这也是为什么盯着库存本身就有价值：你不只是抢货，也是在挑时间点。

## 现成的 DMIT 库存监控渠道，各自什么脾气

不想折腾服务器的人，直接用现成的就行。目前社区里流传比较广的有这么几类：

| 渠道 | 形态 | 覆盖 | 通知方式 | 适合谁 |
| --- | --- | --- | --- | --- |
| stock.qixi.me | 网页看板 | 以 DMIT 为主 | 页面查看 | 想随手扫一眼的人 |
| vpsrestock.com | 网页看板 | 搬瓦工、DMIT、V.PS、绿云、HostDare | 页面 + Telegram 频道 @vpsrestock_com | 同时蹲多家精品线路的人 |
| vpsme.de | 网页 + 订阅 | 搬瓦工、DMIT、VMISS、绿云、V.PS、斯巴达等 | 邮件、Telegram、WxPusher、息知微信 | 想用微信/邮件收通知的人 |
| Telegram 频道 @vpsts | 推送频道 | 搬瓦工、DMIT、RFC、斯巴达、VMISS、绿云等 | Telegram 单向推送 | 只想要"响一下"的人 |
| VPSMarket 机器人 | Telegram 私聊 | 不是查库存，是订阅 NodeSeek / LinuxDo 关键词 | Telegram 私聊 | 想第一时间看到"补货帖"的人 |

这里有个必须说清楚的差别：**检测频率决定了你能不能真的抢到。**

以 vpsrestock.com 为例，它的搬瓦工条目大约每 5 分钟检测一次，V.PS 和 HostDare 约 10 分钟，绿云约 15 分钟，而 DMIT 因为官网反爬较严，检测间隔被保守地拉到约 2 小时 [5]。也就是说，同一个看板上，DMIT 的绿灯可能是两小时前的状态。蹲 DMIT 的时候，看板只能当参考，官网购物车才是准的。

VPSMarket 那条路换个思路：你不监控库存，而是订阅关键词。在 Telegram 里给机器人发 `/sub dmit`，NodeSeek 和 LinuxDo 上出现标题带 dmit 的新帖时就会私聊你，双源轮询约 10 秒一轮 [6]。它的优势是快，缺点是快的是"有人发帖说补货了"，不是官方库存变动本身。

## 自建监控：三种开源方案，和它们各自的坑

如果你手上有闲置的 VPS，自建的响应速度通常比公共看板更好，代价是要自己维护。

**Costinel**（GitHub: harrisonwang/costinel）是 Node.js + Puppeteer 方案，用真实浏览器模拟去读页面，目前支持 Bandwagonhost、VMISS、DMIT 三家。配置方式是在 `.env` 里填 Telegram Bot Token 和 Chat ID，在 `src/config/products.config.js` 里加套餐，在 `src/config/sites.config.js` 里加商家的 CSS 选择器和缺货文本。部署完用 crontab 定时跑，README 给的示例是每 5 分钟一次 [7]。

**Jito**（GitHub: DaBoWin/Jito）是 Python 方案，逻辑更简单——直接读页面上的缺货关键字，命中就通过 Telegram 频道发消息。它的配置示例恰好拿 DMIT 举例：缺货状态文本是 `Out of Stock`，监控周期默认 60 秒，一旦发现有货就进入 24 小时冷却 [8]。这个设计挺实在的，避免了抢到之后还在一分钟一次地轰炸你。

**vps-stock-monitor**（GitHub: vpslog/vps-stock-monitor）带 Flask 的 Web 管理界面，支持 Docker 一键跑，通知可以走 Telegram、微信（息知）或自定义 URL [9]。作者自己写明的限制是：不支持监控具体库存数量，一次只能加一个监控项，而且监控链接打开后必须能看到 "Out Of Stock" 或缺货字样才有效——换句话说，它吃的是 WHMCS 这类标准购物车页面 [10]。

把这三条路放在一起看：

| 方案 | 语言/形态 | 部署成本 | 通知 | 主要限制 |
| --- | --- | --- | --- | --- |
| Costinel | Node.js + Puppeteer | 中，需要装 Chrome 和 xvfb | Telegram | 靠 CSS 选择器，页面改版要跟着改 |
| Jito | Python 脚本 | 低 | Telegram 频道 | 依赖缺货关键字文本 |
| vps-stock-monitor | Python + Flask + Docker | 低，有 Web 界面 | Telegram / 微信 / 自定义 URL | 不监控库存数量，需要页面出现缺货字样 |

三者共同的软肋是一样的：**DMIT 的购物车是 WHMCS 结构，判断库存基本靠"页面上有没有 Out of Stock 字样"。** 这意味着两件事——官网改一次模板，选择器可能就失效；库存从 5 台变成 1 台，脚本看不出来。

另外，别把间隔设得太短。5 分钟一次对 DMIT 来说已经足够勤快，再密除了给自己添麻烦没什么收益，还容易触发官网的风控。如果你想确认一下当前实时的库存和价格是绿是红，直接开官网购物车看最快：👉 [打开 DMIT 官方购物车看实时库存](https://www.dmit.io/aff.php?aff=12046)。

## 有货了，接下来选哪条线

抢到通知只是第一步。DMIT 的三条产品线差别挺大，选错比没抢到更亏。

**Pro（Premium）** 走的是三网 CN2 GIA。以洛杉矶为例，电信去程走 AS4809、联通直连 AS4837、移动经香港移动 AS58453，三网回程都回到 CN2 GIA [11]。这是给大陆访问延迟敏感的站点准备的，也是 DMIT 最常缺货的一条线。

**EB（Eyeball）** 用电信/联通 AS9929 加移动 CMIN2 的组合。在配置相近的情况下，EB 给的月流量通常是 Pro 的两倍左右 [11]。价格差不多、流量翻倍，代价是线路等级略低一档——对联通和移动用户来说，体感差别往往没有价格差那么大。

**T1（Tier 1）** 是纯国际线路，对大陆访问不做任何优化。它的好处在别的地方：流量用完之后不停机，只限速继续跑。比如 T1.WEE 打完 1TB 之后限到 50Mbps 不限量，香港和日本的 T1.WEE 都是这个策略 [12]。用来做海外业务、备份节点或者流量中转，性价比很突出；用来给国内用户建站，就属于花钱买罪受。

选购上还有个容易被忽略的点：DMIT 的套餐在不同机房之间不是简单换算的。香港 Pro 的 $39.90/月只给 500GB 流量，同样的钱在洛杉矶能拿到多几倍的配置。香港贵在延迟，洛杉矶贵在线路质量本身，东京则介于两者之间。想清楚你的流量是往哪去的，再决定往哪加钱。

## DMIT 主要套餐与价格对照

下面这张表按机房和产品线整理了目前公开页面上的主要套餐。**需要提醒的是，DMIT 在 2026 年的调价和上下架比较频繁，同一款套餐在不同渠道的报价已经出现差异，实际价格和库存请以官网购物车为准。**

| 机房 / 线路 | 套餐 | 配置 | 流量 · 带宽 | 超量处理 | 价格（USD） | 购买 |
| --- | --- | --- | --- | --- | --- | --- |
| LAX · Pro（CN2 GIA） | AN5.Pro.TINY | 1核 2G / 20G SSD | 1T · 1Gbps | 限速 4Mbps 不限量 | $12.98/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.Pocket | 1核 2G / 40G SSD | 1.5T · 4Gbps | 限速 4Mbps 不限量 | $18.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.STARTER | 2核 2G / 80G SSD | 3T · 10Gbps | 限速 4Mbps 不限量 | $38.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.MINI | 2核 4G / 80G SSD | 5T · 10Gbps | 限速 8Mbps 不限量 | $79.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.MICRO | 4核 4G / 160G SSD | 7T · 10Gbps | 限速 8Mbps 不限量 | $110.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.MEDIUM | 6核 8G / 160G SSD | 14T · 10Gbps | 限速 10Mbps 不限量 | $289.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.LARGE | 8核 16G / 320G SSD | 25T · 10Gbps | 限速 10Mbps 不限量 | $499.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · Pro | AN5.Pro.GIANT | 12核 24G / 640G SSD | 50T · 10Gbps | 限速 10Mbps 不限量 | $1009.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · EB（9929/CMIN2） | AN5.EB.MINI | 4核 4G / 80G SSD | 10T · 10Gbps | 限速 8Mbps 不限量 | $79.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · EB | AN5.EB.MICRO | 4核 4G / 160G SSD | 14T · 10Gbps | 限速 8Mbps 不限量 | $110.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · EB | AN5.EB.MEDIUM | 6核 8G / 160G SSD | 30T · 10Gbps | 限速 10Mbps 不限量 | $289.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · EB | AN5.EB.LARGE | 8核 16G / 320G SSD | 50T · 10Gbps | 限速 10Mbps 不限量 | $499.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · EB | AN5.EB.GIANT | 12核 24G / 640G SSD | 100T · 10Gbps | 限速 10Mbps 不限量 | $1009.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · T1（国际线路） | AN5.T1.V2C2G | 2核 2G / 40G SSD | 5T · 10Gbps | 限速 100Mbps 不限量 | $14.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · T1 | AN5.T1.V2C4G | 2核 4G / 80G SSD | 10T · 10Gbps | 限速 200Mbps 不限量 | $23.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · T1 | AN5.T1.V4C4G | 4核 4G / 120G SSD | 20T · 10Gbps | 限速 200Mbps 不限量 | $36.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · T1 | AN5.T1.G2C4G | 2核 4G / 80G SSD | 4T · 10Gbps | 限速 100Mbps 不限量 | $16.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| LAX · T1 | AN5.T1.G4C8G | 4核 8G / 160G SSD | 8T · 10Gbps | 限速 100Mbps 不限量 | $36.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · Pro（CN2 GIA） | AS3.Pro.TINY | 1核 1G / 20G SSD | 500G · 1Gbps | 超量暂停 | $39.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · Pro | AS3.Pro.STARTER | 1核 2G / 40G SSD | 1T · 1Gbps | 超量暂停 | $79.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · Pro | AS3.Pro.MINI | 2核 2G / 60G SSD | 1.5T · 1Gbps | 超量暂停 | $126.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · Pro | AS3.Pro.MICRO | 2核 4G / 80G SSD | 2T · 1Gbps | 超量暂停 | $179.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · Pro | AS3.Pro.MEDIUM | 4核 4G / 160G SSD | 2.5T · 1Gbps | 超量暂停 | $239.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · EB（三网 CMI） | AS3.EB.TINY | 1核 1G / 20G SSD | 1T · 1Gbps | 超量暂停 | $29.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · EB | AS3.EB.STARTER | 1核 2G / 40G SSD | 2T · 2Gbps | 超量暂停 | $59.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · EB | AS3.EB.MINI | 2核 2G / 60G SSD | 3T · 2Gbps | 超量暂停 | $89.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · EB | AS3.EB.MICRO | 4核 4G / 80G SSD | 4T · 4Gbps | 超量暂停 | $129.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · EB | AS3.EB.MEDIUM | 4核 8G / 160G SSD | 6T · 4Gbps | 超量暂停 | $199.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1（国际线路） | T1.WEE | 1核 1G / 20G SSD | 1T · 4Gbps | 限速 50Mbps 不限量 | $36.90/年 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.TINY | 1核 1G / 20G SSD | 2T · 4Gbps | 限速 100Mbps 不限量 | $6.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.STARTER | 1核 2G / 40G SSD | 4T · 10Gbps | 限速 100Mbps 不限量 | $12.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.MINI | 2核 2G / 60G SSD | 8T · 10Gbps | 限速 100Mbps 不限量 | $21.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.MICRO | 4核 4G / 80G SSD | 16T · 10Gbps | 限速 200Mbps 不限量 | $32.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.MEDIUM | 4核 8G / 160G SSD | 32T · 10Gbps | 限速 200Mbps 不限量 | $49.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.LARGE | 8核 16G / 320G SSD | 64T · 10Gbps | 限速 500Mbps 不限量 | $99.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| HKG · T1 | T1.GIANT | 8核 24G / 640G SSD | 128T · 10Gbps | 限速 1Gbps 不限量 | $199.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · Pro（CN2 GIA） | AS3.Pro.TINY | 1核 1G / 20G SSD | 500G · 1Gbps | 超量暂停 | $21.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · Pro | AS3.Pro.STARTER | 1核 2G / 40G SSD | 1T · 1Gbps | 超量暂停 | $39.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · Pro | AS3.Pro.MINI | 2核 2G / 60G SSD | 2T · 1Gbps | 超量暂停 | $79.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · Pro | AS3.Pro.MICRO | 4核 4G / 80G SSD | 4T · 1Gbps | 超量暂停 | $159.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · Pro | AS3.Pro.MEDIUM | 4核 8G / 160G SSD | 5T · 1Gbps | 超量暂停 | $259.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · T1（国际线路） | T1.WEE | 1核 1G / 20G SSD | 1T · 4Gbps | 限速 50Mbps 不限量 | $36.90/年 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · T1 | T1.TINY | 1核 2G / 40G SSD | 2T · 4Gbps | 限速 100Mbps 不限量 | $6.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · T1 | T1.STARTER | 2核 2G / 40G SSD | 4T · 10Gbps | 限速 100Mbps 不限量 | $12.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |
| TYO · T1 | T1.MINI | 2核 4G / 80G SSD | 8T · 10Gbps | 限速 100Mbps 不限量 | $21.90/月 | [查看当前库存](https://www.dmit.io/aff.php?aff=12046) |

两个空白对照着看：**LAX.AN4 系列已经全线下架**，官网购物车上看不到任何 AN4 产品 [3]；**TYO 的 EB 系列也已下架** [4]。如果你在监控脚本里还挂着这两条线的 pid，它们会一直显示缺货，不是你抢不到，是已经不存在了。

## 关于优惠码

DMIT 的购物车经常自带年付折扣，结账时在结算页面填入优惠码后点 "Validate Code" 验证是否生效就行——验证不通过的码不是折扣已过，就是和当前付款周期不匹配。

需要提醒的是，网上流传的 DMIT 优惠码列表更新很乱，同一批代码在不同文章里的适用范围说法不一，也有明显已经过期的。与其照抄别人的码，不如在下单前先看一眼官网购物车的实际报价：👉 [查看 DMIT 当前套餐与结账页价格](https://www.dmit.io/aff.php?aff=12046)。付款方式上，支付宝、微信、PayPal 和信用卡都能用，国内用户不需要折腾外币卡。

## 常见问题

**监控显示有货，买的时候却没货了。**
这是常态，不是工具坏了。库存从缺货变现货只需要几秒，而监控脚本受检测间隔限制，天然有延迟。看到通知立刻去下单，但也接受可能扑空的现实。

**DMIT 多久补一次货？**
没有固定周期。从 2026 年的情况看，补货和调价是绑在一起的——硬件资源谈下来、新平台部署完才会上架，中间可能隔好几周。这也是为什么单靠定时刷页面不如同时盯社区：NodeSeek 和 LinuxDo 上的补货帖往往比页面变化更早知道消息 [6]。

**自建监控要不要设代理？**
如果你的监控机器在境外，一般不需要。DMIT 的购物车页面本身不重，主要问题是反爬强度比一般 WHMCS 商家高，这也是公共看板把 DMIT 的检测间隔拉到 2 小时的原因 [5]。自建的时候把间隔放在 5 到 10 分钟之间，加上出错重试就行，不用刻意加速。

**能不能监控具体还有几台？**
开源方案基本都做不到。vps-stock-monitor 的作者就明确写了不支持监控库存数量，只能判断有货/缺货 [10]。想看"还剩几台"目前只能靠人肉刷购物车。

**只蹲一家，还是多开几个渠道？**
如果你只想要 DMIT，用 stock.qixi.me 或 @vpsts 这类单向推送就够了，不用搭机器。如果你同时在蹲搬瓦工、V.PS 这类同样缺货严重的商家，vpsrestock.com 那种一屏看全部多家的看板更省事，但记得它对 DMIT 的刷新频率是最慢的。

## 最后一句实用的

库存监控这件事，工具只能帮你把"该不该看一眼"变成"它来告诉你"。真正决定你能不能买到合适套餐的，还是看到通知之后那三十秒里你有没有想清楚自己要哪一档。提前把目标套餐和预算定好，通知一来直接下单——这比把检测间隔从五分钟改成三分钟有用得多。
