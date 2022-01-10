# `Actor`范围内随机生成   Learn Unreal Engine (with C++)

[SpaceshipBattle · fanxingin/UE4项目 - 码云 - 开源中国 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects/tree/master/SpaceshipBattle)

## `Actor`范围内随机生成

1. 新建`box`组件

   ```cpp
   SpawnArea = CreateDefaultSubobject<UBoxComponent>(TEXT("SpawnArea"));
   	RootComponent = SpawnArea;
   ```

2. 获取随机生成位置

   ```cpp
   FVector AEnemySpawner::GetGenerateLocation()
   {
   	float Distance = 0;
   	FVector Location;
   	
   	while (Distance< MinimumDistanceToPlayer)
   	{
   		//在盒子中产生的随机的点
   		Location = UKismetMathLibrary::RandomPointInBoundingBox(SpawnArea->Bounds.Origin, SpawnArea->Bounds.BoxExtent);
   		Distance = (Location - SpaceShip->GetActorLocation()).Size();
   	}
   	return Location;
   }
   ```

3. 在指定位置生成`Actor`

   ```cpp
   FActorSpawnParameters SpawnParameters;
   		//           生成敌人
   		GetWorld()->SpawnActor<AEnemy>(Enemy, GetGenerateLocation(), FRotator::ZeroRotator, SpawnParameters);
   ```

   

   ![扫码_搜索联合传播样式-标准色版](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201101446013.png)

