# 控制`Actor`朝向,运动  Learn Unreal Engine (with C++)

控制`Actor`的朝向,以及`Actor`的运动

[SpaceshipBattle · fanxingin/UE4项目 - 码云 - 开源中国 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects/tree/master/SpaceshipBattle)

## 控制`Actor`朝向鼠标

1. 设置鼠标在游戏中可见

   - 获取玩家控制器
   - 鼠标可见设置为`true`

   ```cpp
   PC = Cast<APlayerController>(GetController());
   PC->bShowMouseCursor = true;
   ```

2. 获取鼠标与主角之间的角度

   - 获取角度
   - 设置角度

   ```cpp
   void ASpaceShip::LookAtCursor()
   {
   	FVector MouseLocation;
   	FVector MouseDirection;
   	//获取鼠标位置
   	PC->DeprojectMousePositionToWorld(MouseLocation,MouseDirection);
   	//获取当前位置到目标位置所需要旋转的角度
   	//只在XY平面旋转
   	FVector TargetLoaction = FVector(MouseLocation.X, MouseLocation.Y, GetActorLocation().Z);
   	FRotator Rotator = UKismetMathLibrary::FindLookAtRotation(GetActorLocation(), TargetLoaction);
   	SetActorRotation(Rotator);
   }
   ```

3. 这样就使主角始终朝向鼠标

## 控制`Actor`运动

1. 在设置中映射活动与按键

   - 连续运动选择**轴映射**, 例如：连续的前进
   - 不连续的运动选择**操作映射**，例如：跳跃，拾取，开火

   ![image-20220108235032147](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201082350218.png)

2. 输入运动

   - 相当于将输入先存储
   - 让后将存储的向量设置给`Actor`
   - 然后再`Tick()`中调用`Move()`

   ```cpp
   void ASpaceShip::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
   {
   	Super::SetupPlayerInputComponent(PlayerInputComponent);
   	PlayerInputComponent->BindAxis("MoveUp", this, &ASpaceShip::MoveUp);
   	PlayerInputComponent->BindAxis("MoveRight", this, &ASpaceShip::MoveRight);
   }
   ```

   ```cpp
   void ASpaceShip::MoveUp(float Value)
   {
   	//MovementInput<--------------------(方向)*(值)
   	//Movementinput---->ConsumeMovementInputVector
   	AddMovementInput(FVector::ForwardVector,Value);
   }
   ```

   ```cpp
   void ASpaceShip::Move(float time)
   {
   	//给对象加向量        获取移动向量          *time为了防止速度过大穿越  检测碰撞           
   	AddActorWorldOffset(ConsumeMovementInputVector()* Speed * time, true);
   	/*也可以使用这种方法获取时间 需要头文件
   	 *FApp::GetDeltaTime()
   	 */
   }
   ```

   

