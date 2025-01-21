---
title: 短链接分页查询
comment: true   # true/false对应开启/关闭本文章评论
sticky  : false   # 置顶文章
date: 2024-03-09
tags:
  - JAVA
  - ShortLink
  - Project
categories:
  - [JAVA]
  - [项目]
---

# 短链接分页查询
根据 **分组标识 (gid)** 和分页参数，查询符合条件的短链接列表。

### 功能说明：

该方法实现了通过分页查询短链接，查询条件包括 **分组标识** 和 **启用状态** 以及 **删除标记**，并按照创建时间降序排序。返回的结果是经过分页后的短链接数据，并且在最终结果中，使用 `BeanUtil.toBean` 将 `ShortLinkDO` 转换为 `ShortLinkPageRespDTO`。

### 具体实现：

1. **查询条件设置：**
   - **gid (分组标识)**：用于筛选属于特定分组的短链接，`ShortLinkDO::getGid` 与请求参数中的 `requestParam.getGid()` 进行匹配。
   - **启用状态 (enableStatus)**：只有启用状态为 `0`（假设代表有效状态）的短链接才会被查询出来。
   - **删除标记 (delFlag)**：只有删除标记为 `0`（表示未删除）的短链接才会被查询出来。
2. **排序：**
   - 通过 `orderByDesc(ShortLinkDO::getCreateTime)` 对结果按创建时间降序排列，确保最新的短链接排在前面。
3. **分页查询：**
   - 使用 `baseMapper.selectPage(requestParam, queryWrapper)` 执行分页查询。`requestParam` 包含了分页相关的参数（如页码、每页条数），`queryWrapper` 则是构造好的查询条件。
4. **结果转换：**
   - 查询结果是 `ShortLinkDO` 类型的分页数据，通过 `convert` 方法将其转换为 `ShortLinkPageRespDTO` 类型。这样可以返回给前端更加合适的响应格式。

### 请求参数：

- **gid**: 分组标识，用于筛选特定分组的短链接。
- **分页参数**: 包括页码和每页条数，通常由 `requestParam` 中的 `pageNum` 和 `pageSize` 来控制。

### 返回结果：

- 返回一个分页对象 `IPage<ShortLinkPageRespDTO>`，其中包含符合查询条件的短链接列表，经过转换后的数据类型为 `ShortLinkPageRespDTO`。

### 代码逻辑总结：

```java
public IPage<ShortLinkPageRespDTO> pageShortLink(ShortLinkPageReqDTO requestParam) {
    // 构造查询条件
    LambdaQueryWrapper<ShortLinkDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkDO.class)
            .eq(ShortLinkDO::getGid, requestParam.getGid())  // 根据分组标识过滤
            .eq(ShortLinkDO::getEnableStatus, 0)  // 只查询启用状态为0的记录
            .eq(ShortLinkDO::getDelFlag, 0)  // 只查询未删除的记录
            .orderByDesc(ShortLinkDO::getCreateTime);  // 按创建时间降序排列

    // 执行分页查询
    IPage<ShortLinkDO> resultPage = baseMapper.selectPage(requestParam, queryWrapper);

    // 将查询结果转换为响应对象并返回
    return resultPage.convert(each -> BeanUtil.toBean(each, ShortLinkPageRespDTO.class));
}
```

### 总结：

这个方法实现了一个基本的分页查询功能，适用于短链接的管理系统。它通过分页查询符合特定条件（分组标识、启用状态、删除标记）的短链接，并且按创建时间降序排列，最后将查询结果转换为响应对象进行返回。