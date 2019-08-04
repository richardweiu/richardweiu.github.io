---
title: ELK-Timelion使用
date: 2019-04-27 22:12:38
tags: " ELK "
categories: " ELK "
---

> ** 前言 **
　kibana visualize 没有办法简单的实现某种率的走势图，比如：状态报告率、成功率这样，所以此时只能使用 Timelion 来实现

## Timelion 画图

Timelion 相对于 Visualize 来说更加的灵活、更加的自定义化一下，所以可以通过它来画出某种率的图：

```bash
.es(q='_exists_:receive_time', index=qixin_report_v3*, timefield=create_time).divide(.es(*, index=qixin_report_v3*, timefield=create_time)).precision(4).label('状态报告率'),
.es(q='msg_status:0', index=qixin_report_v3*, timefield=create_time).divide(.es(q='_exists_:receive_time', index=qixin_report_v3*, timefield=create_time)).precision(4).label('短信成功率').yaxis(tickDecimals=4)
```

主要遇到的问题：

- 当 timefield 不是默认的时候需要设置，否则画出来永远是 0
- 使用了除法之后的精度调整(只调整 precision 是没有作用的):
  - yaxis(tickDecimals=4)
  - precision(4)
- Timelion 中 term 是 split

## 参考

- 教程：https://www.elastic.co/blog/timelion-tutorial-from-zero-to-hero
