# ![](README.assets/JTRPLogoLow.png)Jason Ma Toon Render Pipeline (JTRP)


这是我业余时间开发的基于**Unity HDRP**并支持**DX12 RayTracing**的卡通渲染工具集，定位于实时生产影视级质量卡通渲染CG。

 

**更多细节请关注专栏，此RM不做过多介绍且不常更新：https://www.zhihu.com/people/blackcat1312/posts**

B站：https://space.bilibili.com/42463206

技术交流群：1046752881

 

## 三渲二入门

![JTRP教程](README.assets/JTRP教程.jpg)

https://www.bilibili.com/video/BV1AA411A7RR/

视频中你会学到：

版本控制
- 为何使用Git：多人协作、版本控制
- Github账户、仓库、LFS、存储和单文件大小限制
- SourceTree教程：
  - https://zhuanlan.zhihu.com/p/212302462
  - https://zhuanlan.zhihu.com/p/254909901
- Clone、修改、暂存、推送、回滚、忽略
- 或直接下载Zip：https://github.com/Jason-Ma-233/JasonMaToonRenderPipeline

前置知识
- 3D 美术、DCC 基础
- Unity 安装、语言包、基本知识
- Unity MMD：https://www.bilibili.com/video/BV1Db411e74e
- Blender：
  -  PMX导入插件：https://github.com/GiveMeAllYourCats/cats-blender-plugin
  -  模型处理：拆分材质、脸单独材质、脸部球面化法线（可选）

JTRP
- 三渲二概述：什么是PBR / NPR / 卡通渲染 / 三渲二、三渲二的常见特征与典型案例
- UTS：https://github.com/unity3d-jp/UnityChanToonShaderVer2_Project/blob/release/urp/2.2.3/Documentation~/index.md
- 参数概览
- Outline
  -  传统法线外扩
  -  P+ 4 Unity：https://www.psoft.co.jp/jp/product/pencil/unity/
  -  P+在线文档：https://docs.psoft.co.jp/pus400w/jp/latest/
- 使用JTRP进行角色卡通渲染
  -  阴影色
  -  光线追踪阴影
  -  头发阴影
  -  脸+头发球形阴影
  -  头发高光
  -  边缘光
- Timeline：角色动画、相机动画、表情动画、ABC
- 实时物理：https://assetstore.unity.com/packages/tools/physics/magica-cloth-160144
- 后处理：光线追踪GI / AO / SSR、Bloom、LUT、ToneMapping……
- 渲染输出

## DXR PBR + NPR Sample

![image-20210111010551810](README.assets/image-20210111010551810.png)![image-20210111010608857](README.assets/image-20210111010608857.png)

Video：https://www.bilibili.com/video/BV1Tr4y1F7Pv

 

## Light Weight ShaderGUI

![image-20210110034731796](README.assets/image-20210110034731796.png)

LWGUI是一般ShaderGUI的替代方案，为了写最少的代码并保持灵活易用而编写。所有功能基于Custom Drawer，只需在Shader Property前加上Attribute即可实现各种自定义ShaderGUI。使用时无需写一行ShaderGUI，写Shader的同时进行排版，不同Shader互不相干。Shader末尾需要添加`CustomEditor "JTRP.ShaderDrawer.LWGUI"`。

LWGUI内置于JTRP，你可以在JTRP的[Lit Shader](https://github.com/Jason-Ma-233/JasonMaToonRenderPipeline/blob/master/Assets/JTRP/Runtime/Material/Lit/MyLitShader.shader)找到更多使用示例。

### Function List

```c#
/// 创建一个折叠组
/// group：group key，不提供则使用shader property name
/// keyword：_为忽略，不填和__为属性名大写 + _ON
/// style：0 默认关闭；1 默认打开；2 默认关闭无toggle；3 默认打开无toggle
Main(string group = "", string keyWord = "", float style = 0)
    
/// 在折叠组内以默认形式绘制属性
/// group：父折叠组的group key，支持后缀KWEnum或SubToggle的KeyWord以根据enum显示
Sub(string group)

/// n为显示的name，k为对应KeyWord，最多5组，float值为当前激活的KeyWord index（0-4）
KWEnum(string group, string n1, string k1, ... string n5, string k5)

/// 以单行显示Texture，支持额外属性
/// extraPropName：需要显示的额外属性名称
Tex(string group = "", string extraPropName = "")
    
/// <summary>
/// 将一张4*256的Ramp贴图绘制为Gradient
/// </summary>
RampDrawer(string group, string defaultFileName = “JTRP_RampMap”)
    
/// 支持并排最多4个颜色，支持HDR/HSV
/// parameter：填入HSV则将当前颜色转换为HSV颜色传入Shader，无需则填"_"
/// color：可选额外颜色的property name
/// 注意：更改参数需要手动刷新Drawer实例，在shader中随意输入字符引发报错再撤销以刷新Drawer实例
Color(string group, string parameter, string color2, string color3, string color4)
    
/// 以SubToggle形式显示float
/// keyword：_为忽略，不填和__为属性名大写 + _ON
SubToggle(string group, string keyWord = "")
    
/// 同内置PowerSlider，非线性Range
SubPowerSlider(string group, float power = 1)
    
/// 同内置Header，仅与LWGUI共同使用
Title(string group, string header)

/// 绘制float以更改Render Queue
[Queue]

```

其中函数名带Sub的一般只支持在折叠组下显示，不带Sub的group参数填“_”以在折叠组外显示，另外Decorator与内置Drawer不兼容，比如`[Header(string)]`只应在不使用Attribute或使用内置Attribute的Property上使用，而在使用LWGUI的Property上应使用`[Title(group, string)]`，否则显示可能会出错。

## 平滑法线导入工具（ModelOutlineImporter）（Legacy）

![](README.assets/Snipaste_2020-04-14_22-30-12.png)

一般的Backface Outline由于是沿法线挤出，会在硬表面模型上产生断裂，为了解决这些问题我开发了平滑的描边法线导入工具。将需要导入平滑法线的模型名称加上后缀名：“**_ol**”，即可自动应用平滑算法后将法线导入原模型的**UV8**。Lit的Outline需与此工具配套使用，详情可以参考[此文章](https://zhuanlan.zhihu.com/p/107664564)。

## Reference

Resources used by this repository, if you think I violated your interests, please contact me (jasonma0012@foxmail.com).

| Name                    | Author      | Link                                                         | Content                                 |
| ----------------------- | ----------- | ------------------------------------------------------------ | --------------------------------------- |
| Japanese School         | SbbUtutuya  | [SbbUtutuya - Asset Store (unity.com)](https://assetstore.unity.com/publishers/5437) | Some models and low-resolution textures |
| Chitanda Eru            | @itou_nko   | [🐗亥と卯🐰 (@itou_nko) / Twitter](https://twitter.com/itou_nko) | Character model and textures            |
| UnityChanToonShaderVer2 | unity3d-jp  | [unity3d-jp/UnityChanToonShaderVer2_Project: UnityChanToonShaderVer2 Project / v.2.0.7 Release (github.com)](https://github.com/unity3d-jp/UnityChanToonShaderVer2_Project) | Shaders                                 |
| Toony Colors Pro        | Jean Moreno | [Toony Colors Pro 2 \| VFX 着色器 \| Unity Asset Store](https://assetstore.unity.com/packages/vfx/shaders/toony-colors-pro-2-8105) | Editor > Ramp Utility                   |

