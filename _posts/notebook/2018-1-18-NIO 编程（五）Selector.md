---
layout: post
title: NIO 编程（五）Selector
categories: io
tags: [java]
---

Selector 是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。


