---
title: 新增短链接
comment: true   # true/false对应开启/关闭本文章评论
sticky  : false   # 置顶文章
date: 2024-03-07
tags:
  - JAVA
  - ShortLink
  - Project
categories:
  - [JAVA]
  - [项目]
#cover: https://w.wallhaven.cc/full/nk/wallhaven-nkgz77.jpg # 文章顶部和文章介绍图（将覆盖文章主页轮播图）
---

# 新增短链接

代码中的主要组件包括数据库操作、短链接生成、布隆过滤器等。

##  **服务实现类 `LinkServiceImpl`**

- **继承和实现**：`LinkServiceImpl` 类继承了 MyBatis-Plus 的 `ServiceImpl`，实现了 `ShortLinkService` 接口。通过这种方式，`LinkServiceImpl` 类可以享受 MyBatis-Plus 提供的 CRUD 操作，同时可以扩展自定义的业务逻辑。
- **构造器注入**：使用了 `@RequiredArgsConstructor` 注解，自动生成一个构造器来注入所有的 `final` 字段（依赖注入），这减少了显式的构造方法编写。

##  **短链接创建方法：`create`**

```java
@Override
public ShortLinkCreateRespDTO create(ShortLinkCreateReqDTO requestParam) {
    // 生成短链接后缀
    String shortLinkSuffix = generateSuffix(requestParam);
    String fullShortUrl = requestParam.getDomain() + "/" + shortLinkSuffix;

    // 创建 ShortLinkDO 对象
    ShortLinkDO shortLink = BeanUtil.toBean(requestParam, ShortLinkDO.class);
    shortLink.setShortUri(shortLinkSuffix);
    shortLink.setFullShortUrl(fullShortUrl);
    shortLink.setEnableStatus(0);

    // 防止短链接误判，重复插入数据库，fullShortUrl 为唯一索引，捕获重复异常
    try {
        baseMapper.insert(shortLink);
    } catch (DuplicateKeyException ex) {
        // 误判的短链接如何处理
        LambdaQueryWrapper<ShortLinkDO> hasShortLink = Wrappers.lambdaQuery(ShortLinkDO.class)
                .eq(ShortLinkDO::getFullShortUrl, shortLink.getFullShortUrl());
        if (hasShortLink != null) {
            log.warn("短链接：{} 重复入库", fullShortUrl);
            throw new ServiceException("短链接生成重复");
        }
    }

    // 将生成的短链接加入布隆过滤器，防止重复
    shortUriCreateCachePenetrationBloomFilter.add(fullShortUrl);

    // 返回短链接创建结果
    return ShortLinkCreateRespDTO.builder()
            .fullShortUrl(shortLink.getFullShortUrl())
            .originUrl(shortLink.getOriginUrl())
            .gid(shortLink.getGid())
            .build();
}
```

### **功能描述**：

- **生成短链接后缀**：调用 `generateSuffix(requestParam)` 方法生成短链接的后缀（`shortLinkSuffix`），然后将后缀与用户提供的域名结合，形成完整的短链接 URL（`fullShortUrl`）。
- **创建 `ShortLinkDO` 实体**：将接收到的请求参数 `requestParam` 转换成 `ShortLinkDO` 实体对象，并设置短链接的相关信息，包括：
  - `shortUri`：短链接后缀。
  - `fullShortUrl`：完整的短链接。
  - `enableStatus`：表示短链接的状态，默认为 0（可能代表未启用）。
- **防止重复插入**：为了避免数据库中插入重复的短链接，代码中利用了 **唯一索引** (`fullShortUrl`)，并在插入操作时捕获 `DuplicateKeyException` 异常：
  - 若插入操作抛出 `DuplicateKeyException`，说明该短链接已经存在（可能是由于布隆过滤器误判）。
  - 通过查询数据库，检查是否已经存在相同的 `fullShortUrl`，如果存在，抛出自定义的 `ServiceException` 异常，标识短链接生成失败。
- **布隆过滤器**：使用 `RBloomFilter` 来存储已生成的短链接，避免重复生成。每当生成一个新的短链接时，都会将该短链接 URL 加入布隆过滤器，这样后续的请求可以通过布隆过滤器快速判断该短链接是否已存在。
- **返回短链接信息**：返回包含生成的短链接信息的响应对象 `ShortLinkCreateRespDTO`，包括：
  - `fullShortUrl`：生成的完整短链接。
  - `originUrl`：原始 URL。
  - `gid`：可能是生成短链接时关联的 ID。

##  **短链接后缀生成方法：`generateSuffix`**

```java
private String generateSuffix(ShortLinkCreateReqDTO requestParam) {
    String shortUri;
    int count = 0;
    while (true) {
        if (count > 10) {
            throw new ServiceException("短链接创建频繁，稍后再试");
        }
        // 降低布隆过滤器冲突率
        String originUrl = requestParam.getOriginUrl();
        originUrl += System.currentTimeMillis();
        shortUri = HashUtil.hashToBase62(originUrl);

        // 如果布隆过滤器中不存在该短链接，跳出循环
        if (!shortUriCreateCachePenetrationBloomFilter.contains(requestParam.getDomain() + "/" + shortUri)) {
            break;
        }
        count++;
    }
    return shortUri;
}
```

### **功能描述**：

- **生成短链接后缀**：通过对原始 URL 加上当前时间戳进行哈希处理（`HashUtil.hashToBase62()`）来生成短链接的后缀 `shortUri`。
  - **防止哈希冲突**：每次生成短链接后缀时，代码会加上当前的时间戳（`System.currentTimeMillis()`）来减少布隆过滤器的冲突率。如果相同的 URL 被多次请求生成短链接，这样可以生成不同的短链接后缀，减少短链接冲突的几率。
- **布隆过滤器检查**：生成后缀后，使用布隆过滤器检查该短链接是否已存在。如果已经存在，重新生成短链接后缀，直到生成一个未被占用的短链接。最多重试 10 次，超过 10 次则抛出 `ServiceException`，提示用户短链接创建过于频繁。

##  **异常处理与误判短链接的处理**

- **误判的原因**：布隆过滤器是一个概率性的数据结构，虽然其误判率较低，但仍然可能出现误判的情况。例如，布隆过滤器可能会误判某个短链接已经存在，从而导致生成相同的短链接。
- **处理重复短链接**：
  - 如果在插入数据库时捕获到 `DuplicateKeyException`，说明短链接的唯一性被违反，表明该短链接已存在。
  - 通过 `LambdaQueryWrapper` 查询数据库中是否已存在相同的 `fullShortUrl`。
  - 如果数据库中确实存在该短链接，日志记录该重复入库的情况，并抛出自定义异常 `ServiceException`，通知前端或业务层短链接生成失败。

##  **日志记录与监控**

- 在捕获到 `DuplicateKeyException` 后，代码通过 `log.warn` 记录了警告日志，提示短链接重复入库。这有助于开发人员或运维人员监控系统的健康状态。

##  **设计考量与优化**

- **性能优化**：布隆过滤器在判断短链接是否已存在时具有高效的查找性能，避免了每次都需要访问数据库，从而提升了系统的性能。
- **容错机制**：通过 `DuplicateKeyException` 异常捕获和数据库查询，系统能够处理布隆过滤器的误判，确保最终数据库中的数据正确无误。
- **防止过度请求**：在 `generateSuffix` 方法中加入了重试限制（最多 10 次），防止短链接生成过于频繁导致系统负载过高。

## 总结：

这段代码实现了一个简易的短链接生成服务，通过结合布隆过滤器、唯一索引、哈希算法等技术，确保生成的短链接唯一且高效。在高并发场景下，能够较好地处理重复短链接的生成问题，同时提供了合理的容错机制来应对可能的误判和异常情况。