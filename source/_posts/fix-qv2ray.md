---
title: 为 Qv2Ray 写 Patch
date: 2022-09-25 06:13:10
tags:
  - C++
  - Qv2Ray
---

## qv2ray not work by core api breaking up

我们知道 `v2ray-core` 从 v4 升级到 v5 时 deprecates 了原先的 cli-api，`qv2ray` 现在不能正确检测 `v2ray` 的各种参数，也不能启动核心，大概率返回错误：

```
flag provided but not defined: -version
```

并伴随错误码 2，这也很容易修，最简单的，在 `qv2ray` 里 deprecates 掉 v4 的 api 就好了：

```diff c++
- proc.start(kernelPath, {"--version"});
+ proc.start(kernelPath, {"version"});
```

但很显然，这种处理如果 `core` 还是 v4 会导致 qv2ray crash，我们认为这似乎不明智的。

我修改了 kernel 的 api：

- 现在 `Qv2RayConfig_Kernel` 会保存内核的版本号，这个版本号是每次运行 `ValidateVersionedKernel` 时缓存下来的。
- 同时每次在 Preferences 里提交修改也会尝试验证内核；
- 所有调用内核的请求现在根据 version 字段 switch 合适的参数；
- 使用正则表达式匹配内核版本；

很显然，这样的处理并不完全尽如人意。

## fix qv2ray empty alpn query

```c++
// src/plugins/protocols/core/OutboundHandler.cpp 166
const auto alpnArray = QJsonIO::GetValue(objStream, { tlsKey, "alpn" }).toArray();
        QStringList alpnList;
        for (const auto v : alpnArray)
        {
            const auto alpn = v.toString();
            if (!alpn.isEmpty())
                alpnList << alpn;
        }
        query.addQueryItem("alpn", QUrl::toPercentEncoding(alpnList.join(",")));
```

此处需要对 `alpnList` 判空否则会导致生成的 url 不合法。

