---
title: UTCP
date: 2025-10-25T05:30
tags: []
categories: []
---

# [UTCP](https://www.utcp.io) Universal Tool Calling Protocol

UTCP is a lightweight, secure, and scalable standard that enables AI agents and applications to discover and call tools directly using their native protocols - **no wrapper servers required**.

UTCP 是一个轻量级、安全且可扩展的标准，支持 AI 智能体和应用直接通过各自原生协议发现和调用工具——**无需包装中间服务器**。


utcp 提供了一个 `/utcp` 路由，用于处理 UTCP 请求。

`/utcp` 返回
```json
{
  "manual_version": "1.0.0",
  "utcp_version": "1.0.1",
  "tools": [{
    "name": "get_weather",
    "description": "Get current weather for a location",
    "inputs": {
      "type": "object",
      "properties": {"location": {"type": "string"}},
      "required": ["location"]
    },
    "tool_call_template": {
      "call_template_type": "http",
      "url": "http://localhost:8000/weather",
      "http_method": "GET"
    }
  }],
  "auth": {
    "auth_type": "api_key",
    "api_key": "${WEATHER_API_KEY}",
    "var_name": "appid",
    "location": "query"
  }
}
```

可以通过以下方式来调用工具
```json
{
  "manual_call_templates": [{
    "name": "weather_api", 
    "call_template_type": "http",
    "url": "http://localhost:8000/utcp", // Or http://api.github.com/openapi.json, the openapi spec gets converted automatically
    "http_method": "GET"
  }]
}
```