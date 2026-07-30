---
title: Chapter 4 Information Channels
date: 2025-05-01 11:04:10
tags:
- translation
categories:
- Information and Coding Theory
---

> The equivocation of the fiend that lies like truth. —— Macbeth
> 
> 在本章中，我们考虑这样一种信息传输过程：一个信息源通过一个**不可靠的（unreliable/noisy）信道**向接收者发送消息。信道中的“噪声（noise）”可能表示：
>
> - 机械错误；
> - 人为错误；
> - 来自另一个信息源的干扰。
>
> 一个很好的例子是：一个电源不足的太空探测器发送回一条消息，而这条消息必须从许多其他更强的竞争信号中被提取出来。由于噪声的存在，接收到的符号可能并不与发送出的符号相同。
>
> 本章的目标是利用熵函数的几种不同形式，衡量：
>
> - 有多少信息被传输了；
> - 在这个过程中有多少信息丢失。
>
> 然后进一步研究：这些信息量与所使用编码的平均码长之间的关系。

