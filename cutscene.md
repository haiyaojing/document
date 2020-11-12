## CutScene

cutscene是对Timeline的一层封装，先了解Timeline和Cinemachine之后，再了解一下我们主要的存取方式，常用轨道即可

#### Timeline

[官网文档链接](https://docs.unity3d.com/cn/2018.4/Manual/TimelineSection.html)看了文档后对与timeline的基本概念、基本操作都应该没问题了。

#### Cinemachine

[官网链接](https://unity.com/cn/unity/features/editor/art-and-design/cinemachine)看完教程或者通过unity的packagemanager下载插件查看Example使用更佳

[别人的博客](https://blog.csdn.net/tanyu159/article/details/88608559)

### 以下是我们项目中的使用情况

核心代码

```
运行时MoonClient
CutScene/
├── MCutSceneAssetMgr.cs	预加载(部分)
├── MCutSceneData.cs	数据定义
├── MCutSceneHelper.cs 辅助类
├── MCutSceneMgr.cs		CutScene控制器，控制资源加载、状态切换
├── MCutsceneObject.cs 预加载封装
├── Shot 切片
│   ├── MAnimationShot.cs	自定义动画切片
│   ├── MBlackCurtainShot.cs 黑幕切片
│   ├── MCameraCullingShot.cs 相机裁剪&LOD切片
│   ├── MCvShot.cs Cv切片
│   ├── MEmotionBubbleShot.cs	气泡切片
│   ├── MEmotionShot.cs	表情切片
│   ├── MEntityOutlineShot.cs	描边切片
│   ├── MEntityScalerShot.cs	实体比例切片
│   ├── MFxParentShot.cs	特效父节点切片(有一些限制)
│   ├── MFxShot.cs	特效切片
│   ├── MHideShot.cs	显隐切片
│   ├── MImageShot.cs	2D图片切片
│   ├── MModelParentShot.cs 模型父节点切片
│   ├── MPostFxShot.cs	后处理切片
│   ├── MPostUberEffectShot.cs 后处理切片
│   ├── MSceneObjShot.cs	场景节点切片
│   ├── MShadowShot.cs 阴影切片
│   ├── MShakeScreenShot.cs 摇晃切片
│   ├── MStoryWordShot.cs 黑幕切片
│   ├── MWeatherShot.cs 天气&时间切片
│   └── MWordBubbleShot.cs 气泡切片
└── Track 轨道
    ├── MAnimationTrack.cs
    ├── MBlackCurtainTrack.cs
    ├── MCvTrack.cs
    ├── MEmotionBubbleTrack.cs
    ├── MEmotionTrack.cs
    ├── MFxParentTrack.cs
    ├── MFxTrack.cs
    ├── MHideTrack.cs
    ├── MImageTrack.cs
    ├── MModelParentTrack.cs
    ├── MPostFxTrack.cs
    ├── MPostUberEffectTrack.cs
    ├── MSceneObjTrack.cs
    ├── MShadowTrack.cs
    ├── MShakeScreenTrack.cs
    ├── MStoryWordTrack.cs
    ├── MWeatherTrack.cs
    └── MWordBubbleTrack.cs


```

常见轨道

1. Track Group(轨道组)

   用途：归类，方便管理![示例](cutscene.assets/tapd_20332331_base64_1577168206_99.png)

   

   

   

