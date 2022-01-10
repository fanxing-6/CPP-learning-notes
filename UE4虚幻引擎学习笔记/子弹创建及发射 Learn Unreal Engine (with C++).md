# 子弹创建及发射 Learn Unreal Engine (with C++)

[SpaceshipBattle · fanxingin/UE4项目 - 码云 - 开源中国 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects/tree/master/SpaceshipBattle)

## 子弹的创建

声明:

```C++
UPROPERTY(EditAnywhere, Category = "Fire")
		TSubclassOf<ABullet> Bullet;
```

实现:

```c++
//在空组件处生产子弹
		GetWorld()->SpawnActor<ABullet>(Bullet, SpawnPoint->GetComponentLocation(), SpawnPoint->GetComponentRotation(), SpawnParameters);
```

## 子弹的发射

1. 创建`UProjectileMovementComponent`组件,不需要attach to root component

2. 调节`UProjectileMovementComponent`蓝图细节

   ```cpp
   //运动类型组件与根组件并列不需要AttachTo RootComponent
   	ProjectileMovementComp = CreateDefaultSubobject<UProjectileMovementComponent>(TEXT("ProjectileMovementComp"));
   ```

   <img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201101352931.png" alt="image-20220110135234687" style="zoom: 33%;" />







![](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201101331171.png)

