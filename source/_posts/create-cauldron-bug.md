---
title: 炼药锅 bug
date: 2022-09-16 18:05:04
tags:
  - Minecraft
  - Create
---

## 提要

- MC@1.18.2
- Create@0.5.0d

蓝图工具选中的非空的炼药锅会保存状态（例如，熔岩炼药锅），然后用任何炼药锅打印出来时会保留熔岩。

## 解释

```java
package com.simibubi.create.foundation.utility;

public class BlockHelper {
  // line 216
  public static void placeSchematicBlock(Level world, BlockState state,
    BlockPos target, ItemStack stack, @Nullable CompoundTag data);
};
```

此函数没有检查/转换炼药锅的状态。

## fix

```java
if (state.is(BlockTags.CAULDRONS))
  state = Blocks.CAULDRON.defaultBlockState();
```
