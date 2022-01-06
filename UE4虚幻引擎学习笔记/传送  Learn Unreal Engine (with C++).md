# 传送,条件加速  Learn Unreal Engine (with C++)

本文以吃豆人游戏为例[UE4项目: 自制UE4 小游戏 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects)

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201061512903.png" alt="image-20220106151245772" style="zoom:67%;" />

## 传送

1.  `pawn`进入`box`触发`OnActorBeginOverlap`
2. 获取目标位置,下一帧将`pawn`坐标更改为目标位置

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201061331339.png" alt="image-20220106133113181" style="zoom:50%;" />

- 首先需要重叠函数与开始重叠事件绑定

  ```cpp
  OnActorBeginOverlap.AddDynamic(this, &ATeleporterActor::OnOverlapBegin);
  ```

- 头文件声明

  ```cpp
  	UPROPERTY(EditAnywhere)
  		ATeleporterActor* Target = nullptr; //目标位置,在蓝图中设置比较方便
  	UPROPERTY(EditAnywhere)
  		USoundCue* TeleportSound;//声音
  	UFUNCTION()
  		void OnOverlapBegin(AActor* TeleporterActor, AActor* OtherActor);//触发重叠后执行的操作
  ```

- 实现

  ```cpp
  void ATeleporterActor::TeleportToTarget(AActor * Actor)
  {
  	//获取传送目标名为"Spawn"的场景组件
  	USceneComponent* TargetSpawn = Cast<USceneComponent>(Target->GetDefaultSubobjectByName("Spawn"));
  	UGameplayStatics::PlaySound2D(this, TeleportSound);
  	Actor->SetActorLocation(TargetSpawn->GetComponentLocation());//更改坐标
  }
  
  void ATeleporterActor::OnOverlapBegin(AActor * TeleporterActor, AActor * OtherActor)
  {
  	if (OtherActor->ActorHasTag("Pacman")) {
  		//下一帧,调用传送函数
  		GetWorldTimerManager().SetTimerForNextTick([OtherActor, this]() { TeleportToTarget(OtherActor); });
  	}
  }
  
  ```

`Target->GetDefaultSubobjectByName`获取名为xxx的子对象,例如本游戏中就是获取`ATeleporterActor`名为`spawn`的子对象

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201061345621.png" alt="image-20220106134532507" style="zoom:50%;" />

## 条件加速

当吃豆人吃到特殊的豆子的时候就会加速,这使用了**动态多播**[^1]主要目的是**降低对象之间的耦合**,代码更加清晰简洁

**动态多播:观察者模式**, 动态即支持蓝图序列化，即**可在蓝图中绑定事件**，但蓝图获取不到在C++中定义的动态多播的实例引用，即使用元数据 BlueprintReadWrite 标记也不行，但可以通过 【**Assign 实例名称】** 的蓝图节点为在C++中定义的动态多播对象绑定新的委托函数

1. 将**加速豆被吃操作**绑定到**豆被吃**事件上
2. 每次有**豆被吃**事件发生的时候,都会广播当前**豆子的类型**
3. 普通豆被吃无反应
4. 加速豆被吃就会执行**加速豆被吃事件**

- 注册动态多播

  ```cpp
  DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FFoodieEatenEvent, EFoodieType, FoodieType);
  ```

- 绑定事件

  <img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201061456661.png" alt="image-20220106145651596" style="zoom:50%;" />

  忽略下面的error,不知道为什么,重新打开的时候就这样了,但还能正常运行......

- 广播豆子类型,当豆子被吃掉的时候

  ```cpp
  void AFoodie::Consume()
  {
  	UGameplayStatics::PlaySound2D(this, ConsumptionSound);
  	FoodieEatenEvent.Broadcast(FoodieType); // 广播类型
  	Destroy();
  }
  ```

  

[^1]: https://mp.weixin.qq.com/s/Vliuv3jfUWU_1VvWSBx70w

