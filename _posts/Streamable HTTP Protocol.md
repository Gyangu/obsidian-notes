---
title: Streamable HTTP Protocol
date: 2025-10-25T05:20
tags: []
categories: []
---

# Streamable HTTP Protocol

Streamable HTTP Protocol 基于 HTTP 协议，可以流式返回数据的协议。

## 与 SSE 的区别

| 特性      | Streamable HTTP Protocol | Server-Sent Events (SSE) |
| ------- | ------------------------ | ------------------------ |
| 传输协议    | HTTP/1.1、HTTP/2、HTTP/3   | 仅支持 HTTP/1.1             |
| 数据格式    | 可自定义（如 JSON、任意数据片段）      | 固定为 text/event-stream    |
| 单连接多流   | 支持（可多路复用）                | 不支持，每个流需独立连接             |
| 客户端兼容性  | 需自实现，主流库支持需评估            | 浏览器原生支持（EventSource API） |
| 重连机制    | 需自行实现                    | 浏览器自动重连                  |
| 二进制支持   | 支持（如编码 chunked 数据）       | 不支持，只能传 text 类型          |
| 单向/双向通信 | 通常单向，也支持带反馈              | 仅单向（服务器推送）               |
| 适合场景    | 流式接口、大文件、ChatGPT响应等      | 简易实时推送、新闻、消息等            |

> 简而言之，SSE 更适合轻量级实时推送，Streamable HTTP Protocol 更灵活、协议兼容性广、适合大规模流式数据传输场景。


> The Streamable HTTP protocol plugin (utcp-http) enables UTCP to handle large HTTP responses by streaming them in chunks. This is ideal for tools that return large datasets, files, or progressive results that don't fit into a single response payload. It leverages HTTP Chunked Transfer Encoding.
>
> Streamable HTTP 协议插件（utcp-http）允许 [[UTCP]] 通过分块流式传输来处理大型 HTTP 响应。这对于需要返回大量数据集、文件或分阶段结果、无法一次性载入所有响应内容的工具特别理想。该方式利用了 HTTP 的分块传输编码（Chunked Transfer Encoding）。