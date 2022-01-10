# 禁止摄像机跟随`Actor`旋转 Learn Unreal Engine (with C++)

[SpaceshipBattle · fanxingin/UE4项目 - 码云 - 开源中国 (gitee.com)](https://gitee.com/fanxingin/Unreal-Projects/tree/master/SpaceshipBattle)

如果直接将摄像机绑定在根组件上,在根组件旋转时,摄像机也会跟着旋转

那么如何让摄像机不跟随根组件旋转,只跟着根组件移动

## 禁止摄像机跟随根组件旋转

![image-20220109190348436](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201091903465.png)

1. 将`SpringArm`作为`Camera`的根组件

2. 设置`SpringArm`

   ![image-20220109190730654](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201091907688.png)





这样摄像机就不会跟随`Actor`旋转了,我们就固定了视角

![扫码_搜索联合传播样式-标准色版](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201091750564.png)

