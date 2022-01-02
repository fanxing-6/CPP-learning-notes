# 静态网格体Actor

## 本文介绍如何放置和使用静态网格体Actor在环境中创建世界场景几何体。

本页面的内容

- [放置](https://docs.unrealengine.com/4.27/zh-CN/Basics/Actors/StaticMeshActor/#放置)
- [让静态网格体Actor可移动](https://docs.unrealengine.com/4.27/zh-CN/Basics/Actors/StaticMeshActor/#让静态网格体actor可移动)
- [让静态网格体模拟物理](https://docs.unrealengine.com/4.27/zh-CN/Basics/Actors/StaticMeshActor/#让静态网格体模拟物理)
- [材质覆盖](https://docs.unrealengine.com/4.27/zh-CN/Basics/Actors/StaticMeshActor/#材质覆盖)
- [碰撞](https://docs.unrealengine.com/4.27/zh-CN/Basics/Actors/StaticMeshActor/#碰撞)

**静态网格体（Static Meshes）** 是虚幻引擎中一种基本类型的可渲染几何体。为了在场景中方便地布置， **静态网格体Actor** 应运而生。在 **内容浏览器** 中将静态网格体拖拽到关卡中，网格体会 自动转换为静态网格体Actor。

尽管它们被称为静态网格体Actor，但这只是指静态网格体Actor的网格体是静态的。静态网格体Actor是可以移动的，比如用于表现电梯；它也可以具备物理模拟效果，以便和玩家发生碰撞。 有关详细信息，请参阅[使静态网格体Actor可移动](https://docs.unrealengine.com/4.27/zh-CN/Basics/Actors/StaticMeshActor/)。

![SMA_header.png](https://docs.unrealengine.com/4.27/Images/Basics/Actors/StaticMeshActor/SMA_header.jpg)

## 放置

静态网格体Actor可以使用标准的Actor放置方法放置在地图上，即通过 **右键单击** 视口中的快捷 菜单，或从[内容浏览器](https://docs.unrealengine.com/4.27/zh-CN/Basics/ContentBrowser)中进行拖拽并放置。

**拖拽并放置**

![SMA_clickNDragCreate.png](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201022146032.jpeg)

1. 在 **内容浏览器** 中，找到要在地图上添加为静态网格体Actor的静态网格体。
2. 在 **内容浏览器** 中 **左键单击** 静态网格体，然后从 **内容浏览器** 中将鼠标拖拽（同时按住 **鼠标左键**）到 **视口** 中要放置网格体的位置处。该位置不必非常精确。你可在以后随时重新放置、旋转和缩放网格体。
3. 松开 **鼠标左键** 将网格体放置到地图中作为静态网格体Actor，如 **属性（Property）** 窗口中所示。

**快捷菜单**

![SMA_rightClickAdd.png](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201022146984.jpeg)

1. 在 **内容浏览器** 中，选择要在地图上添加为静态网格体Actor的静态网格体。
2. 在视口中要放置该网格体的地方 **右键单击**，然后从 **快捷菜单** 中选择 **放置Actor：选择（Place Actor: Selection）**。该位置不必非常精确。你可在以后随时重新放置、旋转和缩放网格体。
3. 静态网格体已经以静态网格体Actor的形式放置在地图中了，如 **属性** 窗口中所示。

## 让静态网格体Actor可移动

要想在游戏过程中移动、旋转或缩放静态网格体Actor，必须先让它 **可以移动**。要做到这一点，选择静态网格体Actor，然后从 **细节（Details）** 面板顶部的 **移动性（Mobility）** 下选择 **可移动（Moveable）**。

## 让静态网格体模拟物理

![SMA_PhysicsConvert.png](https://docs.unrealengine.com/4.27/Images/Basics/Actors/StaticMeshActor/SMA_PhysicsConvert.jpg)

## 材质覆盖

静态网格体的材质可以被覆盖。在一张地图中， 你可以重复使用同一个静态网格体资产，然后让它们每个都呈现不同的材质效果。 **材质（Materials）** 属性位于静态网格体Actor的静态网格体组件的 **渲染（Rendering）** 类别中， 是一个材质数组。这些材质与你在[静态网格体编辑器](https://docs.unrealengine.com/4.27/zh-CN/WorkingWithContent/Types/StaticMeshes/Editor)中指定给静态网格体资产的材质一一对应。 你可以手动将材质分配给一个数组，也可以在 **内容浏览器** 中 将它们直接拖放并应用到 **视口** 中的网格体。

**手动分配**

![SMA_MaterialSingle.png](https://docs.unrealengine.com/4.27/Images/Basics/Actors/StaticMeshActor/SMA_MaterialSingle.jpg)

1. 在 **视口** 中，选择要分配的静态网格体Actor。

2. 在 **细节（Details）** 面板中的 **材质（Materials）** 类别下，你会看到分配给静态网格体Actor的所有材质。

3. 在 **内容浏览器** 中，选择要应用于地图中静态网格体Actor的材质，然后执行以下操作之一：

4. 按下 ![button_assign_left_16x.png](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201022146251.jpeg) 按钮（**材质** 数组中对应项的按钮）来分配材质。现在显示材质已经应用到网格体上了。

   **或者**

5. 从 **内容浏览器** 中 **左键单击** 并拖拽材质到静态网格体Actor细节面板中期望的材质槽中。

**拖拽并放置**

[![SMA_clickNDragMaterial.png](https://docs.unrealengine.com/4.27/Images/Basics/Actors/StaticMeshActor/SMA_clickNDragMaterial.jpg)](https://docs.unrealengine.com/4.27/Images/Basics/Actors/StaticMeshActor/SMA_clickNDragMaterial.png)

1. 在 **内容浏览器** 中，找到要应用于地图中静态网格体Actor的材质。

2. 在 **内容浏览器** 中 **左键单击** 材质，从 **内容浏览器** 中将鼠标拖拽（同时按住 **鼠标左键**）到视口中要应用材质的静态网格体Actor部分处。

3. 松开 **鼠标左键** 来应用该材质。现在所看到的网格体已经应用了材质，并且属性窗口中的 **材质（Materials）** 数组已经更新。

   这会替换掉静态网格体Actor上的所有材质。

## 碰撞

默认情况下，如果静态网格体包含物理形体——无论是在你常用的3D软件中生成 （见：[工作流程：静态网格体](https://docs.unrealengine.com/4.27/zh-CN/WorkingWithContent/Importing/FBX/StaticMeshes)），还是在静态网格体编辑器中生成 （见：[碰撞响应参考](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Physics/Collision/Reference)）——都能产生碰撞效果，因此需将其设置为 **阻挡所有（Block All）**。请参阅 [碰撞](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Physics/Collision)，了解有关碰撞通道和调整碰撞设置的更多信息