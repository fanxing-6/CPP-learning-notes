# 主角的创建与选择 Learn Unreal Engine (with C++)

主角创建有两种方式,本教程以[SpaceshipBattle · fanxingin/UE4项目 - 码云 - 开源中国 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects/tree/master/SpaceshipBattle)

## 1. 新建游戏模式方式

- 新建一个蓝图类,选择游戏模式基础

  <img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201082301495.png" alt="image-20220108230101319" style="zoom:50%;" />

- 在蓝图类的细节中将默认`pawn`类选择主角的蓝图类

  ![image-20220108230316240](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201082303271.png)

- 在项目设置->地图和模式->默认模式->默认游戏模式

  ![image-20220108230515657](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201082305690.png)

  默认游戏模式选择新建的游戏模式蓝图类

## 2. 直接选取玩家

- 依旧采用默认游戏模式

- 将主角蓝图拖放到视图中,并设置为玩家0

- 在世界大纲中选择主角`Actor`,并将自动控制玩家选为玩家0(意味着将本身设置为玩家0)

  <img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201082310917.png" alt="image-20220108231025774" style="zoom: 50%;" />

  

