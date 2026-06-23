---
title: 🌐 CDN 回源开启 HTTP/2 导致 Content-Length 响应头丢失变 chunked 的坑
published: 2026-06-23
updated: 2026-06-23
description: 'CDN 回源开 HTTP/2 后，源站的 Content-Length 响应头消失、退化成分块传输，依赖总大小的下载器全废，关闭回源 HTTP/2 即可。'
image: '/images/cdn_http2_origin_content_length.png'
tags: [CDN, HTTP2, 运维]
category: '运维'
draft: false
---

> 客户端下载进度条卡死、拿不到文件总大小，最后定位到是 **CDN 回源开了 HTTP/2**——它把源站的 `Content-Length` 响应头弄没了。记录这个不直观的坑。

## 现象

同一个文件、同一个 CDN，某次配置改动后：

- 响应里的 `Content-Length` **没了**；
- 取而代之的是 `Transfer-Encoding: chunked`，变成**分块传输**；
- 直接抓源站，源站明明是带 `Content-Length` 的。

长度头是在 **CDN 这一层**被弄丢的。

## 坑：HTTP/2 帧层根本没有 Content-Length

- **HTTP/1.1** 靠响应头界定 body：`Content-Length` 和 `Transfer-Encoding: chunked` 二选一。
- **HTTP/2** 把数据拆成 DATA 帧在流里传，靠帧的 `END_STREAM` 标志收尾，**协议层禁用 Transfer-Encoding**，而 `Content-Length` 只是个可选的参考头、不参与长度界定。

所以**当 CDN 用 HTTP/2 回源时**，源站流式发回 DATA 帧，CDN 节点拿到的是"一段段的流"、并不知道确定的总长度；它再转发给客户端（HTTP/1.1）时，**只能用 chunked 分块往外吐**，源站的 `Content-Length` 就这样在协议转换里被吃掉了。

![HTTP/2 回源丢长度，退化成 chunked](/images/cdn_http2_origin_content_length_flow.png)

这会直接搞死一票依赖总大小的逻辑：下载进度条、断点续传 / Range、完整性校验。

## 解决

**把 CDN 控制台里的"回源 HTTP/2"开关关掉，回源走 HTTP/1.1 即可**——客户端 ↔ CDN 这段照常用 HTTP/2 不受影响，只改 CDN ↔ 源站这段。回源走 1.1 后 CDN 能从源站拿到明确的 `Content-Length` 并原样下发，分块问题消失。

## 结论

`Content-Length` 在 CDN 层莫名消失、响应变 `chunked`，大概率是**回源开了 HTTP/2**。关闭回源 HTTP/2 即可恢复，客户端侧 HTTP/2 不受影响。
