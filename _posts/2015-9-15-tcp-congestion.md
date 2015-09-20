---
categories:
  - Network
tags:
  - 'by Snail'
comment: 
info: 
date: '2015-9-15'
type: post
layout: post
published: true
sha: bd7bdf473f6c04d3c0882b6a1732942d0f768875
slug: tcp-congestion
title: TCP之拥塞处理

---
拥塞控制是发送方使用的流量控制，拥塞控制主要有四种算法：慢启动、拥塞避免、快速重传与快速恢复。

<!--more-->

# 慢启动

慢启动为发送方的 TCP 增加了一个窗口：**拥塞窗口**，记为`cwnd`，慢启动算法如下：

 1. 连接建好的开始先初始化 cwnd = 1，表明可以传一个 MSS 大小的数据。
 2. 每当收到一个 ACK，cwnd++，这是一种指数增加的关系。
 3. 发送方取拥塞窗口与通告窗口的最小值作为发送上限。

#  拥塞避免算法

慢启动算法是在一个连接上发起数据流的方法，但有时我们会达到中间路由器的极限，此时分组将被丢弃，拥塞避免算法是一种处理丢失分组的方法，有两种分组丢失的指示：发生超时和接收到重复的确认。

拥塞避免算法和慢启动算法需要地每个连接维持两个变量：一个拥塞窗口 cwnd 和一个慢启动门限 ssthresh，算法如下：

 1. 初始化 cwnd = 1，ssthresh = 65535。
 2. 当拥塞发生时（超时或收到重复确认），ssthreash 设置当前窗口大小的一半（cwnd 和通告窗口大小的最小值，但最小为2个报文）。如果是超时引起的拥塞，则 cwnd 设置为1，进入慢启动过程，否则进入慢速重传与恢复算法。
 3. cwnd <= ssthresh，进行慢启动；否则正在进行拥塞避免，每次收到一个确认时将 cwnd 增加 1 / cwnd，这是一种加性增长。

# 快速重传与快速恢复

在收到一个失序的报文段时，TCP 立即需要产生一个重复的 ACK，我们不知道一个重复的 ACK 是不是丢失的报文引起的。如果收到3个或3个以上的重复 ACK，那非常有可能是一个报文段丢失了，快速重传与快速恢复算法如下：

 1. 当收到3个重复的 ACK 时，ssthresh = cwnd / 2，cwnd = sshthresh + 3 * MSS，重传丢失报文段。
 2. 再次收到重复的 ACK 时，cwnd = cwnd + 1。
 3. 如果收到了新的 ACK，cwnd = sshthresh，进入拥塞避免算法。