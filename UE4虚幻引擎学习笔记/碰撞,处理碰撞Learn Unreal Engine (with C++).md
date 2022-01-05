# 碰撞,处理碰撞,发射     Learn Unreal Engine (with C++)

本文使用打砖块游戏举例

## 碰撞,处理碰撞

碰撞就相当于一个`Actor`进入另一个`Box`中,用这个思路就可以处理碰撞了

`OnComponentBeginOverlap`

当某些内容开始重叠此组件时调用的事件，例如玩家进入触发器。

**委托 事件 **[^1]



`AddDynamic( UserObject, FuncName )`

用于在动态组播委托上调用AddDynamic()的辅助宏。自动生成函数命名字符串。

当碰撞时

```cpp
UFUNCTION()
		void OnOverlapBegin(class UPrimitiveComponent* OverlappedComp, class AActor* OtherActor,
			class UPrimitiveComponent* OtherComp, int32 OtherBodyIndexType, bool bFromSweep,
			const FHitResult& SweepResult);
```

```cpp
void ABrick::BeginPlay()
{
	Super::BeginPlay();
	Box_Collision->OnComponentBeginOverlap.AddDynamic(this, &ABrick::OnOverlapBegin);
}
```

```cpp
/** 当某对象进入球体组件时调用 */
UFUNCTION()
void OnOverlapBegin(class UPrimitiveComponent* OverlappedComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

/** 当某对象离开球体组件时调用 */
UFUNCTION()
void OnOverlapEnd(class UPrimitiveComponent* OverlappedComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
```

## 发射

```cpp
GetBall()->AddForce(FVector(0.0f, 0.0f, 1000.0f), FName(), true);
SM_Ball->AddImpulse(FVector(140.0f, 0.0f, 130.0f), FName(), true);
```

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201051935265.png" alt="image-20220105193538982" style="zoom:50%;" />

`UPrimitiveComponent::AddImpulse`[^2]

给一个刚体增加一个冲量。好一时瞬间爆发

`UPrimitiveComponent::AddForce`

对单个刚体施加一个力



[^1]: [仅使用C++ | 虚幻引擎文档 (unrealengine.com)](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ClassCreation/CodeOnly/)
[^2]: [UPrimitiveComponent::AddImpulse | Unreal Engine Documentation](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Components/UPrimitiveComponent/AddImpulse/)

