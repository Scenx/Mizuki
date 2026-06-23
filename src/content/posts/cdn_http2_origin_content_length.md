---
title: 🌐 CDN 回源开 HTTP/2 把 Content-Length 弄没了：响应变 chunked 的坑
published: 2026-06-23
updated: 2026-06-23
description: 'CDN 开启 HTTP/2 回源后，源站响应的 Content-Length 头消失、改成分块传输，导致依赖总大小的下载器拿不到长度。根因与解决办法。'
image: '/images/cdn_http2_origin_content_length.png'
tags: [CDN, HTTP2, 运维]
category: '运维'
draft: false
---

排查一个"客户端下载进度条不动 / 拿不到文件总大小"的问题，最后定位到是 **CDN 回源开了 HTTP/2**。这篇记录一下这个不太直观的坑。

## 问题现象

客户端从 CDN 下载资源时，需要先读响应头里的 `Content-Length` 来知道文件总大小（算进度、做断点续传校验）。在某个时间点之后，突然发现：

- 同一个文件、同一个 CDN，**响应里的 `Content-Length` 没了**；
- 取而代之的是 `Transfer-Encoding: chunked`，响应变成了**分块传输**；
- 直接回源到 OSS / 源站抓包，源站明明是带 `Content-Length` 的。

也就是说，**长度头是在 CDN 这一层被"弄丢"的**。

## 根因：HTTP/2 帧层根本没有 Content-Length

要理解这个现象，得先知道 HTTP/1.1 和 HTTP/2 在"怎么表示 body 长度"上的本质区别：

- **HTTP/1.1** 用响应头来界定 body：要么 `Content-Length: N` 明确告诉你有多少字节，要么 `Transfer-Encoding: chunked` 一块一块地传、最后用一个零长度块收尾。两者**二选一，不能共存**。
- **HTTP/2** 把数据拆成 `DATA` 帧在流（stream）里传，靠帧的 `END_STREAM` 标志来标记结束。它**在协议层禁用了 `Transfer-Encoding`**，而 `Content-Length` 在 HTTP/2 里只是一个**可选的、纯参考性**的头，并不参与帧的长度界定。

关键就在这里：**当 CDN 用 HTTP/2 去回源时**，源站把数据通过 DATA 帧流式发回来，CDN 节点拿到的是"一段一段的流"，**并不要求源站在帧层提供一个确定的总长度**。于是 CDN 在把内容转发给客户端（HTTP/1.1）时，因为自己也"不知道"完整长度，就**只能用 `chunked` 分块往外吐**，而不会补一个 `Content-Length`。

源站的 `Content-Length` 就这样在"HTTP/2 回源 → HTTP/1.1 下发"的协议转换里被吃掉了。

> 一句话：**HTTP/2 回源让 CDN 失去了"确定的总长度"这个信息**，下发时只能退化成分块传输，`Content-Length` 自然就没了。

## 为什么这会出事

很多场景默认依赖 `Content-Length`：

- **下载进度条**：没有总大小就算不出百分比；
- **断点续传 / Range 请求**：要靠总长度校验、计算偏移；
- **客户端预分配 buffer / 完整性校验**：拿不到长度就没法判断"是不是下全了"。

一旦响应退化成 `chunked`，这些逻辑要么失效、要么得额外兜底处理。

## 解决办法：关闭 HTTP/2 回源

最干脆的做法——**在 CDN 控制台把"回源 HTTP/2"开关关掉，让 CDN 回源走 HTTP/1.1**。

- **客户端 ↔ CDN** 这一段可以照常用 HTTP/2（对用户的访问体验没影响）；
- 只需要把 **CDN ↔ 源站** 这一段回源协议改回 HTTP/1.1。

回源走 HTTP/1.1 后，CDN 从源站就能拿到明确的 `Content-Length`，下发给客户端时也会原样带上，分块传输的问题随之消失。

各家 CDN 的开关名称略有不同（"回源 HTTP/2"、"Origin HTTP/2"、"源站 HTTP/2"等），但位置大同小异，一般都在**源站配置 / 回源设置**里。

## 结论

`Content-Length` 在 CDN 这层莫名消失、响应变 `chunked`，大概率是**回源开了 HTTP/2**——因为 HTTP/2 帧层不依赖确定的 body 长度，CDN 回源时拿不到总长度，只能用分块往外发。**关闭 HTTP/2 回源、回源改走 HTTP/1.1** 即可恢复 `Content-Length`，而客户端侧的 HTTP/2 完全不受影响。

---

**参考链接：**
- [Transfer-Encoding - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Transfer-Encoding)
- [Chunked transfer encoding - Wikipedia](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)
- [RFC 7540 - HTTP/2（8.1.2.2 节，禁用逐跳头）](https://datatracker.ietf.org/doc/html/rfc7540#section-8.1.2.2)
