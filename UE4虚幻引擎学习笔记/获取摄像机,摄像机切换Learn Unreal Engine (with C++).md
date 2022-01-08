# 获取摄像机,摄像机切换Learn Unreal Engine (with C++)

摄像机应该是使用最普遍的组件了

## 获取摄像机,摄像机切换

1. 新建C++类(以CameraActor为父类)

2. 将摄像机在地图中放置

3. 头文件声明

   ```cpp
   	virtual void BeginPlay() override;
   
   	UPROPERTY(EditAnywhere, BlueprintReadWrite)
   		UBoxComponent* OverlapVolume; // 盒体组件,用于检测人物碰撞
   
   	UPROPERTY(EditAnywhere, BlueprintReadWrite)
   		TSubclassOf<ACameraActor> CameraToFind; //待寻找的摄像机
   
   	UPROPERTY(EditAnywhere, BlueprintReadWrite)
   		float CameraBlendTime; // 切换时间
   
   	virtual void NotifyActorBeginOverlap(AActor* OtherActor) override;
   	virtual void NotifyActorEndOverlap(AActor* OtherActor) override;
   ```

   

4. 实现

   ```cpp
   void ABlendTriggerVolume::NotifyActorBeginOverlap(AActor* OtherActor)
   {
   	Super::NotifyActorBeginOverlap(OtherActor);
   
   	//如果是人物转换就能成功,反之nullptr
   	if(AStaticCameraCharacter* PlayerCharacterCheck = Cast<AStaticCameraCharacter>(OtherActor))
   	{
   		//得到玩家角色的玩家控制器,同上
   		if(APlayerController* PlayerCharacterController = Cast<APlayerController>(PlayerCharacterCheck->GetController()))
   		{
   			TArray<AActor*> FoundActors;
   			//获取所有对象的数组
   			UGameplayStatics::GetAllActorsOfClass(GetWorld(), CameraToFind, FoundActors);
   			//摄像机绑定
   			PlayerCharacterController->SetViewTargetWithBlend(FoundActors[0], CameraBlendTime, 
   				EViewTargetBlendFunction::VTBlend_Linear);
   		}
   	}
   }
   ```

   ```cpp
   void ABlendTriggerVolume::NotifyActorEndOverlap(AActor* OtherActor)
   {
   	Super::NotifyActorEndOverlap(OtherActor);
   
   	if (AStaticCameraCharacter* PlayerCharacterCheck = Cast<AStaticCameraCharacter>(OtherActor))
   	{
   		if(APlayerController* PlayerCharacterController = Cast<APlayerController>(PlayerCharacterCheck->GetController()))
   		{
   			PlayerCharacterController->SetViewTargetWithBlend(PlayerCharacterController->GetPawn(), CameraBlendTime, EViewTargetBlendFunction::VTBlend_Linear);
   
   		}
   	}
   }
   ```

   

5. 之后我们基于摄像机类创建蓝图类,并设置相关内容

   在蓝图细节中找到如下,并设置

   ![image-20220106233812235](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201062338269.png)

