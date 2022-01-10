# 旋转根组件 Learn Unreal Engine (with C++)

在UE4中,根组件是无法旋转定位的,只能够缩放,在一些情况下,我们有旋转根组件的需求

[SpaceshipBattle · fanxingin/UE4项目 - 码云 - 开源中国 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects/tree/master/SpaceshipBattle)

## 旋转根组件

1. 将`SceneComponent`设为根组件

2. 然后将`StaticMeshComponent`attach to 根组件

3. 设置想要调节的组建的属性

   ```cpp
   	RootComp = CreateDefaultSubobject<USceneComponent>(TEXT("RootComp"));
   	RootComponent = RootComp;
   
   	BulletSM = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BulletSM"));
   	BulletSM->SetupAttachment(RootComponent);
   ```

   ![image-20220110132944899](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201101329968.png)

![扫码_搜索联合传播样式-标准色版](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201101318801.png)



