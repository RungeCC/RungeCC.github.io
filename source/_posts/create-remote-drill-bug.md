---
title: 机械动力远程钻头
date: 2022-09-14 00:20:40
tags:
  - Minecraft
  - Create
---

## 提要

- MC 1.18.2
- Create 0.5.0d

被装配在移动结构上的钻头工作到一般半通过特殊手段移动（譬如，装配进矿车再放下来）之后会保留原来的工作状态，这意味着他可以隔空破坏方块。

## 解释

```java
package com.simibubi.create.content.contraptions.components.actors;

public class BlockBreakingMovementBehaviour implements MovementBehaviour;
```

此类的方法 `tickBreaker`:

```java
public void tickBreaker(MovementContext context); // line 143
```

在每一个 `process` 中并不会检查被破坏的方块的位置 `breakingPos` 和移动结构中 `breaker` 的距离，从而执行破坏动作可以是隔空的：

```java
BlockHelper.destroyBlock(context.world, breakingPos, 1f, stack -> this.dropItem(context, stack)); // line 200
```

这里 `dropItem` 会判断是否存入 `contraption` 的空间，空间满的时候的行为等等，这里不再展开，总之默认掉落物会吸入在移动结构的容器中，满时则丢出。

我们知道，移动结构是通过维护一个 nbt 标签树来实现的，而装配的矿车被扳手右键取下时不会改变 nbt 标签：

```java
package com.simibubi.create.content.contraptions.components.structureMovement.mounted;

public class MinecartContraptionItem extends Item;
```

中监听交互事件的函数：

```java
@SubscribeEvent
public static void wrenchCanBeUsedToPickUpMinecartContraptions(PlayerInteractEvent.EntityInteract event);
```

不会尝试检测工作方块的状态，仅仅只会检查 `ContraptionMovementSetting.isNoPickup` 等等然后将矿车连同移动结构的数据序列化：

```java
// line 235-259
ItemStack generatedStack = create(type, contraption).setHoverName(entity.getCustomName());

try {
  ByteArrayDataOutput dataOutput = ByteStreams.newDataOutput();
  NbtIo.write(generatedStack.serializeNBT(), dataOutput);
  int estimatedPacketSize = dataOutput.toByteArray().length;
  if (estimatedPacketSize > 2_000_000) {
    player.displayClientMessage(Lang.translateDirect("contraption.minecart_contraption_too_big")
      .withStyle(ChatFormatting.RED), true);
    return;
  }

} catch (IOException e) {
  e.printStackTrace();
  return;
}

if (contraption.getContraption()
  .getBlocks()
  .size() > 200)
  AllAdvancements.CART_PICKUP.awardTo(player);
    
  player.getInventory().placeItemBackInInventory(generatedStack);
  contraption.discard();
  entity.discard();
```

类似的反序列代码也不会检查，不再赘述。

## fix

- 在交互时间中取消 `contraption` 内工作构建的一切行为；
- 在 `tickBreaker` 的时候额外检查 `breakingPos`。