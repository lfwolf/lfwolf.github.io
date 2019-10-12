---
title: wechat extension
date: 2019-09-30 17:09:49
tags:
---

## 为什么需要微信扩展

这两年大火的社群营销（S2b2c）主要玩法就是微信群营销，通过好衣库、唯品会提供的素材在各自维护的群内做播客。主要群体就是妈妈们，当日也包括了我老婆。
所以我希望做个工具减轻她的公众。

## 工作流程

做个微信扩展，监控素材群，素材群主要就是发送当日的推荐商品图片，其中二维码是上家的，需要替换后转发到其他微信群。所以流程应该是：

```flow
st=>start: Start:>http://www.google.com[blank]
e=>end:>http://www.google.com
op1=>operation: 监控素材群
sub1=>subroutine: 图片替换二维码
sub2=>subroutine: 转发微信群
cond=>condition: 图片消息？:>http://www.google.com
cond1=>condition: 是否退出

st->op1->cond
cond(yes)->sub1(right)->sub2->op1
cond(no)->cond1(yes)->e
cond1(no)->op1

```

## 开源组件

1. itchat: 基于微信网页版的webapi。[code](https://github.com/littlecodersh/itchat) [文档](https://itchat.readthedocs.io/zh/latest/)
2. wechaty: 基于微信网页版的webapi。[code](https://github.com/Chatie/wechaty)
3. **WeChatExtension-ForMac**: 这个就比较NB了，是微信mac版的扩展，上面2个基于web api的都挂了，无法登录，目前只剩这个能用了。[code](https://github.com/MustangYM/WeChatExtension-ForMac)


