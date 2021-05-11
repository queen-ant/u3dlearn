参考：https://github.com/candycat1992/Unity_Shaders_Book

* [渲染过程（概念）](#渲染过程概念)
  * [应用阶段](#应用阶段)
  * [几何阶段](#几何阶段)
  * [光栅化阶段](#光栅化阶段)
* [渲染过程（硬件）](#渲染过程硬件)
  * [CPU](#cpu)
  * [GPU](#gpu)
* [Unity Shader模板](#unity-shader模板)
* [ShaderLab](#shaderlab)
  * [结构](#结构)
    * [名字](#名字)
    * [Properties](#properties)
    * [SubShader](#subshader)
    * [FallBack](#fallback)
* [矩阵相关英文名称](#矩阵相关英文名称)
* [基本数据类型](#基本数据类型)
  * [整数数据类型](#整数数据类型)
  * [纹理/采样器类型](#纹理采样器类型)
  * [精度、硬件支持和性能](#精度硬件支持和性能)
* [语义](#语义)
  * [从应用阶段 输入模型数据 给顶点着色器时 Unity支持的常用语义](#从应用阶段-输入模型数据-给顶点着色器时-unity支持的常用语义)
  * [从顶点着色器（的输出） 输入数据 给片元着色器时 Unity使用的常用语义](#从顶点着色器的输出-输入数据-给片元着色器时-unity使用的常用语义)
  * [片元着色器 输出 时Unity支持的常用语义](#片元着色器-输出-时unity支持的常用语义)
* [跨平台差异](#跨平台差异)
  * [屏幕坐标不同](#屏幕坐标不同)
  * [Shader语法有差异](#shader语法有差异)
  * [Shader语义有差异](#shader语义有差异)
* [Shader编写规范](#shader编写规范)
  * [避免不必要的计算](#避免不必要的计算)
  * [慎用分支和循环语句](#慎用分支和循环语句)
* [基础光照](#基础光照)
  * [环境光](#环境光)
  * [漫反射（diffuse）](#漫反射diffuse)
  * [高光反射(Specular)](#高光反射specular)
* [UnityCG\.cginc](#unitycgcginc)
* [基础纹理](#基础纹理)
  * [纹理坐标UV](#纹理坐标uv)
  * [纹理采样tex2D](#纹理采样tex2d)
  * [反射率计算](#反射率计算)
  * [\_TexName\_ST属性](#_texname_st属性)
  * [纹理属性](#纹理属性)
    * [Wrap Mode](#wrap-mode)
    * [Filter Mode](#filter-mode)
    * [Generate Mip Maps](#generate-mip-maps)
  * [凹凸映射（法线贴图）](#凹凸映射法线贴图)
  * [渐变纹理](#渐变纹理)
  * [遮罩纹理](#遮罩纹理)
* [透明效果](#透明效果)
  * [Queue](#queue)
  * [透明度测试](#透明度测试)
  * [透明度混合](#透明度混合)
    * [Blend](#blend)
      * [支持的操作有：](#支持的操作有)
      * [支持的因子有：](#支持的因子有)
      * [常见的混合类型](#常见的混合类型)
  * [开启深度写入的透明度混合](#开启深度写入的透明度混合)
  * [双面渲染](#双面渲染)
    * [透明度测试的双面渲染](#透明度测试的双面渲染)
    * [透明度混合的双面渲染](#透明度混合的双面渲染)

# 渲染过程（概念）
## 应用阶段
- 布置场景数据
- 粗粒度剔除
- 设置渲染状态

**最终输出渲染图元，图元（primitive）即点、线、三角面。**

## 几何阶段
主要工作是把顶点信息从local space转换到screen space

- 顶点shader（Vertex Shader） *可编程，必须实现*

工作：顶点变换（变换到裁剪空间）、顶点着色（逐顶点光照）

- 曲面细分shader（Tessellation Shader） *可编程，选择实现*

工作：细分图元

- 几何shader（Geometry Shader） *可编程，选择实现*

工作：逐图元操作或生成更多图元

- 裁剪（Clipping） *不可编程、可配置*

工作：在裁剪空间内对视椎体内的场景进行裁剪，通过透视除法得到归一化的设备坐标（Normalized Device Coordinates ，NDC）

- 屏幕映射（Screen Mapping） *不可编程、不可配置*

工作：将顶点从裁剪空间映射到屏幕空间

最终输出顶点信息

## 光栅化阶段
- 三角形设置（Triangle Setup） *不可编程、不可配置*

工作：根据顶点信息进行插值计算出三角形的边

- 三角形遍历（Triangle Traversal） *不可编程、不可配置*

工作：也被称为扫描变换（Scan Conversion）。查找被三角形覆盖的像素，根据顶点信息（如深度信息）对被覆盖的像素进行插值，生成片元，最终输出片元序列。片元状态包括了（但不限于）像素的屏幕坐标、深度信息，以及其他从几何阶段输出的顶点信息，例如法线、纹理坐标等。

- 片元shader（Fragment Shader） *可编程，选择实现*

工作：计算片元的着色，输出是一个或者多个颜色值。这一阶段可以完成很多重要的渲染技术，其中最重要的技术之一就是纹理采样。

- 逐片元操作(Per-Fragment Operations) *不可编程、可配置*

工作：模板测试、深度测试，决定片元的可见性。混合，输出颜色缓冲区作为最终效果，包含双重缓冲技术。

**逐片元操作的顺序不是唯一的。在Unity给出的渲染流水线中，我们可以发现它给出的深度测试是在片元着色器之前。这种将深度测试提前执行的技术通常也被称为Early-Z技术。**

最终产生像素，构成屏幕图像

# 渲染过程（硬件）
## CPU
- 将顶点数据（图元列表）装入内存，然后从内存装入显存
- 设置渲染状态
- 向GPU发送draw call

GPU收到draw call之后工作
## GPU
- 几何阶段
- 光栅化阶段

# Unity Shader模板

# ShaderLab
## 结构
### 名字

```C
Shader "MyShader/Name"{}
```

### Properties

```C
Shader "MyShader/Name"
{
  Properties
  {
    // Numbers and Sliders
    _Int ("Int", Int) = 2
    _Float ("Float", Float) = 1.5
    _Range("Range", Range(0.0, 5.0)) = 3.0
    // Colors and Vectors
    _Color ("Color", Color) = (1,1,1,1)
    _Vector ("Vector", Vector) = (2, 3, 6, 1)
    // Textures
    _2D ("2D", 2D) = "" {}
    _Cube ("Cube", Cube) = "white" {}
    _3D ("3D", 3D) = "black" {}
  }
  FallBack "Diffuse"
}
```

### SubShader
- Tag

Tag是键值对，键值都是字符串类型

Tags { "TagName1" = "Value1" "TagName2" = "Value2" }

类型：
```C
{"Queue"} //控制渲染队列
{"RenderType"} //对着色器分类
{"DisableBatching"} // true|false，关闭批处理
{"ForceNoShadowCasting"} // true|false，禁止投射阴影
{"IgnoreProjector"} // true|false，忽略projector
{"CanUseSpriteAtlas"} // true|false，是否用于精灵图
{"PreviewType"} //面板预览类型

//上述标签仅可以在SubShader 中声明，而不可以在Pass块中声明
```
详情：

[Queue](#queue)

"RenderType"：定义渲染类型。预制的值有这些

    "Opaque"：绝大部分不透明的物体都使用这个； 
    
    "Transparent"：绝大部分透明的物体、包括粒子特效都使用这个； 
    
    "Background"：天空盒都使用这个； 
    
    "Overlay"：GUI、镜头光晕都使用这个；

- RenderSetup（渲染状态设置）

这些指令可以设置显卡的各种状态
```
Cull 剔除。值：Back ｜ Front｜ Off，剔除背面/正面/关闭剔除。
ZTest 深度测试模式。值：Less Greater｜ LEqual｜ GEqual｜ Equal｜ NotEqual｜ Always，设置深度测试时使用的函数
ZWrite 是否开启深度写入。值：On ｜ Off，开启/关闭。
Blend 混合模式。值：SrcFactor DstFactor，开启并设置混合模式
ColorMask RGB | A | 0 | 其他任何R、G、B、A的组合，用于指定渲染结果的输出通道，当ColorMask设为0时，意味着该Pass不写入任何颜色通道，即不会输出任何颜色。
```
详情：

[Blend](#blend)

- Pass

一样有Name，Tag，RenderSetup

**特殊Pass，UsePass "MyShader/MYPASSNAME"可以调用其他shader中的Pass，因为Unity会把Pass的名字全部转换为大写，所以使用UsePass时要使用大写名字。**

**特殊Pass还有GrabPass，该Pass负责抓取屏幕并将结果存储在一张纹理中，以用于后续的Pass处理。**

对于RenderSetup，除了上面提到的状态设置外，在Pass中我们还可以使用固定管线的着色器命令。

对于Tag，在Pass中不能使用上面提到的类型，有独特的类型：
```C
{"LightMode"} //设置Pass在渲染流水线中的角色
{"RequirOption"} //达到某种条件后才渲染该Pass
```
详情：

[LightMode](#lightMode)

### FallBack

当所有SubShader都不能支持时，使用FallBack给定的shader
```C
FallBack "name"
//关闭 FallBack Off
```

# 矩阵相关英文名称
- model space、object space、local space

- world space

顶点从model space到world space叫model transform

- view space、camera space

从观察空间到屏幕空间的转换需要经过一个投影操作 （projection）

从world space到view space叫view transform

- clip space

projection matrix、clip matrix，把view space转换到clip space

- screen space

homogeneous division、perspective division，clip space中的坐标{x,y,z,w}经过齐次除法后的坐标为{x/w,y/w,z/w,1}，这一步得到的坐标叫做归一化的设备坐标 （Normalized Device Coordinates，NDC）

屏幕坐标x0 = (x/w+1)\*pWidth/2，y0 = (y/w+1)\*pHeight/2。pWidth、pHeight为屏幕像素宽、高。

# 基本数据类型

着色器中的大多数计算是对浮点数（在 C# 等常规编程语言中为 float）进行的。浮点类型有几种变体：float、half 和 fixed（以及它们的矢量/矩阵变体，比如 half3 和 float4x4）。这些类型的精度不同（因此性能或功耗也不同）：

- 高精度：float

最高精度浮点值；一般是 32 位（就像常规编程语言中的 float）。

**完整的 float 精度通常用于世界空间位置、纹理坐标或涉及复杂函数（如三角函数或幂/取幂）的标量计算。**

- 中等精度：half

中等精度浮点值；通常为 16 位（范围为 –60000 至 +60000，精度约为 3 位小数）。

**半精度对于短矢量、方向、对象空间位置、高动态范围颜色非常有用。**

- 低精度：fixed

最低精度的定点值。通常是 11 位，范围从 –2.0 到 +2.0，精度为 1/256。

**固定精度对于常规颜色（通常存储在常规纹理中）以及对它们执行简单运算非常有用。**

## 整数数据类型

整数（int 数据类型）通常用作循环计数器或数组索引。为此，它们通常可以在各种平台上正常工作。

根据平台的不同，GPU 可能不支持整数类型。例如，Direct3D 9 和 OpenGL ES 2.0 GPU 仅对浮点数据进行运算，并且可以使用相当复杂的浮点数学指令来模拟简单的整数表达式（涉及位运算或逻辑运算）。

Direct3D 11、OpenGL ES 3、Metal 和其他现代平台都对整数数据类型有适当的支持，因此使用位移位和位屏蔽可以按预期工作。

## 纹理/采样器类型
通常按照如下方式在 HLSL 代码中声明纹理：
```
sampler2D _MainTex;
samplerCUBE _Cubemap;
```
对于移动平台，这些将转换为“低精度采样器”，即预期纹理应具有低精度数据。如果您知道纹理包含 HDR 颜色，则可能需要使用半精度采样器：
```
sampler2D_half _MainTex;
samplerCUBE_half _Cubemap;
```
或者，如果纹理包含完整浮点精度数据（例如深度纹理），请使用完整精度采样器：
```
sampler2D_float _MainTex;
samplerCUBE_float _Cubemap;
```

## 精度、硬件支持和性能
**PC的GPU始终为高精度**。也就是说，对于所有 PC (Windows/Mac/Linux) GPU，在着色器中编写 float、half 还是 fixed 数据类型都无关紧要。这些 GPU 将始终以 32 位浮点精度来计算所有数据。

仅当目标平台是移动端 GPU 时，half 和 fixed 类型才变得重要，在这种情况下，这些类型主要面临功耗（有时候是性能）约束。请记住，要确认是否遇到精度/数值问题，必须在移动设备上测试着色器。

fixed精度实际上只在一些较旧的移动平台上有用，**在大多数现代的GPU上，它们内部把fixed和half当成同等精度来对待。**

# 语义

## 从应用阶段 输入模型数据 给顶点着色器时 Unity支持的常用语义

- POSITION（float4），模型空间中的顶点位置

- TANGENT（float4）、NORMAL（float3）的范围是[-1,1]

- COLOR（fixed4）的范围是[0,1]

- TEXCOORDn ，如TEXCOORD0、TEXCOORD1

该顶点的纹理坐标，TEXCOORD0表示第一组纹理坐标，依此类推。通常是float2或float4类型

TEXCOORDn 中n 的数目是和Shader Model有关
```
struct appdata_full {
    float4 vertex : POSITION;
    float4 tangent : TANGENT;
    float3 normal : NORMAL;
    float4 texcoord : TEXCOORD0;
    float4 texcoord1 : TEXCOORD1;
    float4 texcoord2 : TEXCOORD2;
    float4 texcoord3 : TEXCOORD3;
    fixed4 color : COLOR;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};
```
**只有normal是float3，其他都是4**

## 从顶点着色器（的输出） 输入数据 给片元着色器时 Unity使用的常用语义

- SV_POSITION，裁剪空间中的顶点坐标，结构体中必须包含一个用该语义修饰的变量
- COLOR0、COLOR1，通常用于输出第一、二组顶点颜色，但不是必需的
- TEXCOORD0~TEXCOORD7，通常用于输出纹理坐标，但不是必需的
```
struct v2f {
  float4 pos: SV_POSITION;
  float3 worldNormal: TEXCOORD0;
  float3 worldPosition: TEXCOORD1;
  float2 uv:TEXCOORD2;
  fixed3 color: COLOR0;
};
```
## 片元着色器 输出 时Unity支持的常用语义

- SV_Target，输出值将会存储到渲染目标（render target）中。

# 跨平台差异

## 屏幕坐标不同
OpenGL原点在左下角，DirectX在左上角

## Shader语法有差异

- OpenGL初始化可以缺省，DirectX不行
```
float4 v = float4(0.0); //OpenGL
float4 v = float4(0.0, 0.0, 0.0, 0.0); //DirectX
```
- out修饰符的参数在DirectX中需要初始化
```
void vert (inout appdata_full v, out Input o) {
    // 使用Unity内置的UNITY_INITIALIZE_OUTPUT宏对输出结构体o进行初始化
    UNITY_INITIALIZE_OUTPUT(Input,o);
    // … 
}
```

- DirectX 9 / 11也不支持在顶点着色器中使用tex2D函数。

tex2D是一个对纹理进行采样的函数，如果我们的确需要在顶点着色器中访问纹理，需要使用tex2Dlod函数来替代。

## Shader语义有差异

- 使用SV_POSITION 来描述顶点着色器输出的顶点位置。

一些Shader使用了POSITION 语义，但这些Shader**无法在索尼PS4平台上或使用了细分着色器的情况下正常工作。**

- 使用SV_Target 来描述片元着色器的输出颜色。

一些Shader使用了COLOR或者COLOR0语义，**同样的，这些Shader无法在索尼PS4上正常工作。**

# Shader编写规范

## 避免不必要的计算

不同的Shader Target、不同的着色器阶段，我们可使用的临时寄存器和指令数目都是不同的。

Unity支持的Shader Target：
- #pragma target 2.0

默认的Shader Target等级。相当于Direct3D 9上的Shader Model 2.0，**不支持对顶点纹理的采样，不支持显式的LOD纹理采样等**

- #pragma target 3.0

相当于Direct3D 9上的Shader Model 3.0，**支持对顶点纹理的采样等**

- #pragma target 4.0

相当于Direct3D 10上的Shader Model 4.0，**支持几何着色器等**

- #pragma target 5.0

相当于Direct3D 11上的Shader Model 5.0
```
Shader Model是由微软提出的一套规范，通俗地理解就是它们决定了Shader中各个特性（feature）的能力（capability）。

这些特性和能力体现在Shader能使用的运算指令数目、寄存器个数等各个方面。

Shader Model等级越高，Shader的能力就越大。
```

## 慎用分支和循环语句

- 应该尽量把计算向流水线上端移动

例如把放在片元着色器中的计算放到顶点着色器中，或者直接在CPU中进行预计算，再把结果传递给Shader

- 分支判断语句中使用的条件变量最好是常数，即在Shader运行过程中不会发生变化。
- 每个分支中包含的操作指令数尽可能少。
- 分支的嵌套层数尽可能少。

# 基础光照

BRDF（Bidirectional Reflectance Distribution Function）分类：
- 经验模型
- 基于物理的BRDF模型

渲染方式分类：
- 逐顶点光照，Phong着色。在顶点着色器中进行光照计算。
- 逐像素光照，高洛德着色（Gouraud shading）。在片元着色器中进行光照计算。

**如果要实现逐像素光照，需要在顶点着色器中计算世界坐标下的法线和顶点坐标传递给片元着色器**

## 环境光
`fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;`

## 漫反射（diffuse）

- 兰伯特（Lambert）模型

光源颜色\*漫反射颜色\*Max(0,Dot(表面法线，顶点到光源方向))

- 半兰伯特模型

光源颜色\*漫反射颜色\*(0.5+0.5\*Dot(表面法线，顶点到光源方向))

## 高光反射(Specular)

- Phong模型

光源颜色\*高光反射颜色\*(Max(0,Dot(顶点到摄像机方向，反射方向)))^光泽度

- Blinn-Phong模型

光源颜色\*高光反射颜色\*(Max(0,Dot(表面法线，normalized(顶点到摄像机方向+顶点到光源方向))))^光泽度

**Blinn模型的夹角大小是Phong模型的一半**

# UnityCG.cginc

- \_WorldSpaceCameraPos

摄像机world space坐标

- unity_ObjectToWorld、unity_WorldToObject

object space到world space的变换矩阵
`float3 worldPos = mul(unity_ObjectToWorld, localPos).xyz;`

- inline float3 UnityWorldSpaceViewDir( in float3 worldPos )

返回world space中点到camera的方向，**无normalize**

- inline float3 ObjSpaceViewDir( in float4 v )

返回object space中点到camera的方向，**无normalize**

- inline float3 UnityWorldSpaceLightDir( in float3 worldPos )

**仅可用于前向渲染中**。返回world space中从该点到光源的光照方向，已处理不同的光源类型。**无normalize**。

- inline float3 ObjSpaceLightDir( in float4 v )

**仅可用于前向渲染中**。返回object space中从该点到光源的光照方向，已处理不同的光源类型。**无normalize**。

- inline float3 UnityObjectToWorldNormal( in float3 norm )

将法线从object space变换到world space

- inline float3 UnityObjectToWorldDir( in float3 dir )、UnityWorldToObjectDir

将方向矢量从object space变换到world space，**输入float4会忽略w分量**

- inline float4 UnityWorldToClipPos( in float3 pos )、UnityObjectToClipPos

将位置矢量从world space/object space变换到clip space

- #define TRANSFORM_TEX(tex,name) (tex.xy \* name##\_ST.xy + name##\_ST.zw)

对顶点纹理坐标以缩放和偏移量进行变换得到uv坐标

- inline fixed3 UnpackNormal(fixed4 packednormal)

从采样中求解法线，需要将纹理类型设置成Normal Map

```
inline fixed3 UnpackNormal(fixed4 packednormal)
{
#if defined(UNITY_NO_DXT5nm)
    return packednormal.xyz * 2 - 1;
#else
    return UnpackNormalmapRGorAG(packednormal);
#endif
}

fixed3 tNormal = UnpackNormal(tex2D(_BumpTex,i.uv));
```

# 基础纹理

## 纹理坐标UV
可以使用float4来储存两张纹理的坐标，uv.xy、uv.zw。

主纹理和凹凸纹理一般共用一个坐标。

插值寄存器最大支持float4，储存3x3矩阵可用3个float4值，充分利用空间可以把第四个值w储存其他数据。

## 纹理采样tex2D

fixed4 tex2D(sampler2D Tex,float2 uv)

输入纹理和纹理坐标，返回对纹理的采样（rgba值）

## 反射率计算

`fixed3 albedo = tex2D(_MainTex,i.uv).rgb*_Color.rgb;`

运用于漫反射和环境光

`fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;`

`fixed3 diffuse = _LightColor0.rgb * albedo*saturate(dot(i.wNormal,lightDir));`

## \_TexName\_ST属性

在纹理名后加上\_ST，用于储存纹理坐标的缩放和偏移，类型为float4。直接声明即可。

**其中偏移的取值范围是[-1,1]，如0.1则为偏移该轴方向上纹理长度1/10的距离**
```
sampler2D _MainTex;
float4 _MainTex_ST;
```
- TRANSFORM_TEX
```
#define TRANSFORM_TEX(tex,name) (tex.xy * name##_ST.xy + name##_ST.zw)

o.uv.xy = TRANSFORM_TEX(v.texcoord,_MainTex);
```
TRANSFORM_TEX即应用缩放和偏移到texcoord坐标，texcoord是物体的顶点属性，所以跟纹理（_MainTex）的变换是相反的。如Material文件面板上Tiling为2是缩小两倍，Offset为正是纹理左移。

## 纹理属性

### Wrap Mode

决定了当纹理坐标超过[0, 1]范围后将会如何被平铺

- Repeat，超过[0, 1]则截取小数部分，效果是重复
- Clamp，大于1则为1，小于0则为0

### Filter Mode

决定了当纹理由于变换而产生拉伸时将会采用哪种滤波模式

Point、 Bilinear、Trilinear，质量依次提升。会影响多级渐远纹理（Mip Maps）的效果。

### Generate Mip Maps

开启多级渐远纹理技术，没有开启则Filter Mode中Bilinear、Trilinear效果一样。

## 凹凸映射（法线贴图）

分模型空间、切线空间、灰度图（高度图）。一般使用切线空间。

对模型空间/切线空间下的法线贴图采样，得到的是将[-1,1]映射到[0,1]的值，p=(n+1)/2，所以要得到法线，则n=2p-1。

tex2D采样只能得到二维坐标xy，由于切线空间中法线总是正方向，所以z坐标可以直接由xy求得。

- inline fixed3 UnpackNormal(fixed4 packednormal)

从采样中求解法线，需要将纹理类型设置成Normal Map

```
inline fixed3 UnpackNormal(fixed4 packednormal)
{
#if defined(UNITY_NO_DXT5nm)
    return packednormal.xyz * 2 - 1;
#else
    return UnpackNormalmapRGorAG(packednormal);
#endif
}

fixed3 tNormal = UnpackNormal(tex2D(_BumpTex,i.uv));
```
- 副切线的计算需要乘v.tangent.w，因为OpenGL和DirectX的uv坐标v轴是相反的

`fixed3 wBiTangent = cross(wNormal,wTangent.xyz)*v.tangent.w;`

- 切线空间的好处

1、切线空间记录的是相对法线信息，所以可以使用在不同的网格上

2、可进行uv动画

3、可压缩（只需储存xy）

- 切线空间到世界空间的变换矩阵
```
fixed3 wTangent = UnityObjectToWorldDir(v.tangent.xyz);
fixed3 wNormal = UnityObjectToWorldNormal(v.normal);
fixed3 wBiTangent = cross(wNormal,wTangent.xyz)*v.tangent.w;

//一个float4储存矩阵的一行
o.TanToWrd0 = float4(wTangent.x,wBiTangent.x,wNormal.x,wPos.x);
o.TanToWrd1 = float4(wTangent.y,wBiTangent.y,wNormal.y,wPos.y);
o.TanToWrd2 = float4(wTangent.z,wBiTangent.z,wNormal.z,wPos.z);
```

## 渐变纹理

可用于计算漫反射，获得插画风格的渲染效果。

结合半兰伯特模型，对渐变纹理以坐标fixed2(halfLambert, halfLambert)进行采样。渐变纹理实际上是一维坐标（v轴上值相同），所以uv两个坐标都使用halfLambert。

```
fixed halfLambert  = 0.5 * dot(worldNormal, worldLightDir) + 0.5;
fixed3 diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert)).rgb * _Color.rgb;
fixed3 diffuse = _LightColor0.rgb * diffuseColor;
```

当使用fixed2(halfLambert, halfLambert)对渐变纹理进行采样时，虽然理论上halfLambert的值在[0, 1]之间，但由于浮点精度问题可能会有1.00001这样的值出现。如果我们使用的是Repeat模式，此时就会舍弃整数部分，只保留小数部分，得到的值就是0.000 01，导致错误。所以**渐变纹理的Wrap Mode应设置为Clamp**。

## 遮罩纹理

用于对纹理进行像素级的控制。对需要控制的效果乘上遮罩纹理的采样即可。

遮罩纹理也可以利用rgba通道来储存多种数据，实现高自由度的贴图效果。

# 透明效果

先渲染不透明物体，然后再从后往前渲染透明物体。

## Queue

索引号越小，渲染顺序越靠前。

- Background 索引号1000

最先渲染的队列，如天空盒

- Geometry 索引号2000

默认队列，不透明物体使用此队列

- AlphaTest 索引号2450

需要透明度测试的物体使用此队列

- Transparent 索引号3000

需要透明度测试混合的物体使用此队列

- Overlay 索引号4000

最后渲染的队列，可用于实现叠加效果，如镜头光晕

## 透明度测试

还需的Tags：{"IgnoreProjector"="True" "RenderMode"="TransparentCutout"}

要么可见要么不可见。

只需在片元shader中最前面使用clip。对含有透明度的纹理进行采样，将texColor.a与设定的_Cutoff比较，小于则剔除。

```
// Alpha test
    clip (texColor.a - _Cutoff);
// Equal to 
//  if ((texColor.a - _Cutoff) < 0.0) {
//      discard;
//  }
```

## 透明度混合

还需的Tags：{"IgnoreProjector"="True" "RenderMode"="Transparent"}

**需要关闭深度写入(ZWrite Off)，但是不会关闭深度测试**，所以先渲染不透明物体再渲染透明物体也能正常遮挡。

设置好Blend之后只需在片元shader中返回a通道的值即可：
```
return fixed4(ambient + diffuse, texColor.a * _AlphaScale);
```

### Blend

在Unity中，当我们使用Blend（Blend Off命令除外）命令时，除了设置混合状态外也开启了混合。

但是，在其他图形API中我们是需要手动开启的。

例如在OpenGL中，我们需要使用glEnable(GL_BLEND)来开启混合。但在Unity中，它已经在背后为我们做了这些工作。

- Blend Off

关闭混合

- Blend ScrFactor DstFactor

对rgb通道和a通道使用相同的因子

DstColor = ScrFactor\*SrcColor + DstFactor\*DstColor

- Blend ScrFactor DstFactor, ScrFactorA DstFactorA

对rgb通道和a通道使用不相同的因子。**注意要一个逗号隔开**。

DstColor.rgb = ScrFactor\*SrcColor.rgb + DstFactor\*DstColor.rgb

DstColor.a = ScrFactorA\*SrcColor.a + DstFactorA\*DstColor.a

- Blend BlendOperation

设置混合操作。

#### 支持的操作有：

- Add 默认操作
- Sub 默认操作Add中加变成减
- RevSub 调换Sub中的相减顺序，即用混合后的目标颜色减去混合后的源颜色
- Min 取rgba四个通道中源颜色和目标颜色的较小值，无需因子计算
- Max 取rgba四个通道中源颜色和目标颜色的较大值，无需因子计算

#### 支持的因子有：
- One 1
- Zero 0
- SrcColor 源颜色，包含四个通道
- SrcAlpha 源颜色a通道
- DstColor 目标颜色，包含四个通道
- DstAlpha 目标颜色a通道
- OneMinusSrcColor 1-源颜色
- OneMinusSrcAlpha 1-源颜色a通道
- OneMinusDstColor 1-目标颜色
- OneMinusDstAlpha 1-目标颜色a通道

#### 常见的混合类型
```
// 正常（Normal），即透明度混合
Blend SrcAlpha OneMinusSrcAlpha
// 柔和相加（Soft Additive）
Blend OneMinusDstColor One
// 正片叠底（Multiply），即相乘
Blend DstColor Zero
// 两倍相乘（2x Multiply）
Blend DstColor SrcColor
// 变暗（Darken）
BlendOp Min
Blend One One
// 变亮（Lighten）
BlendOp Max
Blend One One
// 滤色（Screen）
Blend OneMinusDstColor One
// 等同于
Blend One OneMinusSrcColor
// 线性减淡（Linear Dodge）
Blend One One
```
## 开启深度写入的透明度混合

一种方案是使用两个Pass 来渲染模型：**第一个Pass开启深度写入，但不输出颜色**，它的目的仅仅是为了把该模型的深度值写入深度缓冲中；

第二个Pass进行正常的透明度混合，由于上一个Pass已经得到了逐像素的正确的深度信息，该Pass就可以按照像素级别的深度排序结果进行透明渲染。

但这种方法的**缺点在于，多使用一个Pass会对性能造成一定的影响**。

在开启Blend的Pass之前添加一个开启深度写入的Pass，然后将ColorMask设为0。
```
// Extra pass that renders to depth buffer only
Pass {
  Tags{"LightMode"="ForwardBase"}
  ZWrite On
  ColorMask 0
}
```

## 双面渲染

用Cull指令来控制需要剔除哪个面的渲染图元。

### 透明度测试的双面渲染

在Pass中设置Cull Off即可

### 透明度混合的双面渲染

使用两个Pass，第一个Pass剔除Front，第二个Pass剔除Back。
```
SubShader {
    Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
    Pass {
        Tags { "LightMode"="ForwardBase" }
        // First pass renders only back faces 
        Cull Front
        // 片元shader代码
    }
    Pass {
        Tags { "LightMode"="ForwardBase" }
        // Second pass renders only front faces 
        Cull Back
        // 片元shader代码，与第一个Pass一样
    }
}
```

# 渲染路径
全局默认渲染路径的设置：

Unity 2018：Edit → Project Settings → Player → Other Settings → Rendering Path

Unity 2019：Edit → Project Settings → Graphics → Tier Settings → Rendering Path

摄像机中可以设置渲染路径覆盖默认设置：

Unity 2018：Rendering Path → Use Player Settings 是默认设置

Unity 2019：Rendering Path → Use Graphics Settings 是默认设置

## LightMode
- Always 不管使用哪种渲染路径，该Pass总是会被渲染，但不会计算任何光照
- ForwardBase 用于前向渲染 。该Pass会计算环境光、最重要的平行光、逐顶点/SH光源和Lightmaps
- ForwardAdd 用于前向渲染 。该Pass会计算额外的逐像素光源，每个Pass对应一个光源
- Deferred 用于延迟渲染 。该Pass会渲染G缓冲（G-buffer）
- ShadowCaster 把物体的深度信息渲染到阴影映射纹理（shadowmap）或一张深度纹理中
- 旧版：
```C#
PrepassBase //用于遗留的延迟渲染 。该Pass会渲染法线和高光反射的指数部分
PrepassFinal //用于遗留的延迟渲染 。该Pass通过合并纹理、光照和自发光来渲染得到最后的颜色
Vertex、VertexLMRGBM和VertexLM //用于遗留的顶点照明渲染
```
## 前向渲染
顶点照明渲染路径仅仅是前向渲染路径的一个子集，因此已经被抛弃。

三个缓冲区：颜色缓冲、深度缓冲（Z buffer）、模板缓冲（Stencil Buffer）。帧缓冲是以上三种缓冲结合起来的统称。

在前向渲染中，先利用深度缓冲中的深度值进行**深度测试**决定一个片元是否可见，若可见则进行光照计算，并更新帧缓冲；否则剔除。

有三种光照处理模式：逐顶点处理、逐像素处理，球谐函数 （Spherical Harmonics，SH）处理 

在前向渲染中，
- 场景中最亮的平行光（方向光）总是按逐像素处理。
- **渲染模式被设置成Important的光源，会按逐像素处理。**
- **渲染模式被设置成Not Important的光源，会按逐顶点或者SH处理。**
- 如果根据以上规则得到的逐像素光源数量小于Quality Setting 中的逐像素光源数量(Pixel Light Count)，会有更多的光源以逐像素的方式进行渲染。
- 最多有4个光源按逐顶点处理。

### Base Pass
渲染设置
```
Tags{"LightMode"="ForwardBase"}
#pragma multi_compile_fwdbase
```
光照计算
```
一个逐像素的最亮平行光，以及所有逐顶点和SH光源。
```
可实现并访问的光照效果
```
光照纹理（lightmap）
环境光
自发光
阴影（平行光阴影）
```
Base Pass一般只定义一个，但是也可以定义多次，例如需要双面渲染等情况。只有一个Base Pass时，一个Base Pass仅会执行一次。

### Additional Pass
渲染设置
```
Tags{"LightMode"="ForwardAdd"}
//如果我们没有开启和设置混合模式，那么Additional Pass的渲染结果会覆盖掉之前的渲染结果
//通常情况下，我们选择的混合模式是Blend One One，线性减淡
Blend One One
#pragma multi_compile_fwdadd
```
光照计算
```
其他影响物体的逐像素光源，每个逐像素光源执行一次该Pass

一个Additional Pass会根据影响该物体的其他逐像素光源的数目被多次调用，即每个逐像素光源会执行一次Additional Pass
```
光照效果
```
默认不支持阴影
```
Additional Pass中渲染的光源在默认情况下是没有阴影效果的，即便我们在它的Light组件中设置了有阴影的Shadow Type。

但我们可以在Additional Pass中使用 __#pragma multi_compile_fwdadd_fullshadows__ 代替#pragma multi_compile_fwdadd编译指令，为点光源和聚光灯开启阴影效果。

但这需要Unity在内部使用更多的Shader变种。

### 可使用的变量和函数
- 变量
```C#
float4 _LightColor0
//该Pass处理的逐像素光源的颜色，已经是颜色和强度相乘后的结果

float4 _WorldSpaceLightPos0
//_WorldSpaceLightPos0.xyz是该Pass处理的逐像素光源的位置。
//如果该光源是平行光，那么_WorldSpaceLightPos0.w是0，其他光源类型w值为1

float4×4 _LightMatrix0
//从世界空间到光源空间的变换矩阵。可以用于采样cookie和光强衰减（attenuation）纹理

float4 unity_4LightPosX0
float4 unity_4LightPosY0
float4 unity_4LightPosZ0
//仅用于Base Pass。前4个非重要的点光源在世界空间中的位置

float4 unity_4LightAtten0
//仅用于Base Pass。存储了前4个非重要的点光源的衰减因子

half4[4] unity_LightColor
//仅用于Base Pass。存储了前4个非重要的点光源的颜色

```
- 函数
```C#
float3 WorldSpaceLightDir (float4 v)
//仅可用于前向渲染中 。输入一个模型空间中的顶点位置，返回世界空间中从该点到光源的光照方向。
//内部实现使用了UnityWorldSpaceLightDir函数
//没有被归一化

float3 UnityWorldSpaceLightDir (float4 v)
//仅可用于前向渲染中 。输入一个世界空间中的顶点位置，返回世界空间中从该点到光源的光照方向。
//没有被归一化

float3 ObjSpaceLightDir (float4 v)
//仅可用于前向渲染中 。输入一个模型空间中的顶点位置，返回模型空间中从该点到光源的光照方向。
//没有被归一化

float3 Shade4PointLights (...)
//仅可用于前向渲染中。计算四个点光源的光照，它的参数是已经打包进矢量的光照数据，通常就是上述的内置变量，
//如unity_4LightPosX0, unity_4LightPosY0, unity_4LightPosZ0、unity_LightColor和unity_4LightAtten0等。
//前向渲染通常会使用这个函数来计算逐顶点光照
```
## 延迟渲染

除了前向渲染中使用的颜色缓冲和深度缓冲外，延迟渲染还会利用额外的缓冲区，这些缓冲区也被统称为**G缓冲（G-buffer）**，其中G是英文**Geometry**的缩写。

G缓冲区存储了我们所关心的表面（通常指的是离摄像机最近的表面）的其他信息，例如该表面的法线、位置、用于光照计算的材质属性等。

**延迟渲染必须包含两个Pass。**

在第一个Pass中，我们**不进行任何光照计算，而是仅仅计算哪些片元是可见的**。

这主要是通过深度缓冲技术来实现，**当发现一个片元是可见的，我们就把它的相关信息存储到G缓冲区中**。

**对于每个物体来说，该Pass仅会执行一次**。

在第二个Pass中，我们**利用G缓冲区的各个片元信息，例如表面法线、视角方向、漫反射系数等，进行真正的光照计算，并更新到帧缓冲中**。

- 优点
```
可以看出，延迟渲染使用的Pass数目通常就是两个，这跟场景中包含的光源数目是没有关系的。信息全部统一存储到G缓冲中，然后统一计算。

换句话说，延迟渲染的效率不依赖于场景的复杂度，而是和我们使用的屏幕空间的大小有关，与物体接受光照的像素数成正比。

对于延迟渲染路径来说，它最适合在场景中光源数目很多、如果使用前向渲染会造成性能瓶颈的情况下使用。
```
而且，**延迟渲染路径中的每个光源都可以按逐像素的方式处理**。

- 缺点：
```
不支持真正的抗锯齿（anti-aliasing）功能。

不能处理半透明物体。

对显卡有一定要求。如果要使用延迟渲染的话，显卡必须支持MRT（Multiple Render Targets）、Shader Mode 3.0及以上、深度渲染纹理以及双面的模板缓冲。
（2006 年以后制造的大多数 PC 显卡都支持延迟着色。所有至少运行 OpenGL ES 3.0 的移动设备都支持延迟着色。）
```
**注意：使用正交投影 (Orthographic projection) 时不支持延迟渲染。如果摄像机的投影模式设置为正交模式，则摄像机将回退到前向渲染**。

### G缓冲区中默认包含的渲染纹理（Render Texture，RT）

RT的通道使用情况在中括号内。

- RT0：ARGB32格式：漫射颜色[RGB]，遮挡[A] 。
- RT1：ARGB32格式：高光反射（镜面反射）颜色[RGB] ，粗糙度[A] (即高光反射指数部分)。
- RT2：ARGB2101010格式：世界空间法线[RGB] ，A通道未使用。
- RT3：ARGB2101010(非HDR) 或 ARGBHalf(HDR)格式：自发光 + lightmap + 反射探针（reflection probes）
- 深度+模板缓冲区。

因此，默认的 G 缓冲区布局为 160 位/像素（非 HDR）或 192 位/像素 (HDR)。

**如果混合光照模式为 Shadowmask 或 Distance Shadowmask，则使用第五个RT**：

- RT4，ARGB32格式：光照遮挡值 (RGBA)。

此时，G 缓冲区布局为 192 位/像素（非 HDR）或 224 位/像素 (HDR)。

**如果硬件不支持五个并发渲染目标，则使用阴影遮罩的对象将回退到前向渲染路径**。

当摄像机不使用 HDR 时，自发光 + lightmap (RT3) 采用对数编码，因此提供的动态范围高于 ARGB32 纹理通常可能提供的范围。

请注意，当摄像机使用 HDR 渲染时，不会为自发光 + lightmap (RT3) 创建单独的渲染目标；而是将摄像机渲染到的渲染目标（即传递给图像效果的渲染目标）用作 RT3。

**当在第二个Pass中计算光照时，默认情况下仅可以使用Unity内置的Standard光照模型**。

如果我们想要使用其他的光照模型，就需要替换掉原有的Internal-DeferredShading.shader文件。(见 https://docs.unity3d.com/cn/2019.4/Manual/RenderTech-DeferredShading.html 末尾)

方法是将内置着色器中的 Internal-DeferredShading.shader 文件的修改版本放入“Assets”文件夹中名为“Resources”的文件夹内。然后打开 Graphics 设置（菜单：Edit > Project Settings，然后单击 Graphics 类别）。将“Deferred”下拉选单改为“Custom Shader”。然后，更改当前使用的着色器对应的着色器 (Shader) 选项。

### 可使用的变量和函数
- 变量
```C#
float4 _LightColor
//光源颜色

float4×4 _LightMatrix0
//从世界空间到光源空间的变换矩阵。可以用于采样cookie和光强衰减纹理
```

## 光源类型
**5个基本属性：位置、方向、颜色、强度、衰减**

- 平行光

只有方向没有位置，没有光照衰减

- 点光源

半径（Range）

- 聚光灯

高度（Range）、锥顶角度（Spot Angle）

- 面光源

烘焙时发挥作用

**光源的强度和颜色以及光源到物体的距离将会影响前向渲染中光源重要度的排序**

### 不同光源的宏
UnityCG.cginc内置的UnityWorldSpaceLightDir如下
```C++
inline float3 UnityWorldSpaceLightDir( in float3 worldPos )
{
    #ifndef USING_LIGHT_MULTI_COMPILE
        return _WorldSpaceLightPos0.xyz - worldPos * _WorldSpaceLightPos0.w;
    #else
        #ifndef USING_DIRECTIONAL_LIGHT
        return _WorldSpaceLightPos0.xyz - worldPos;
        #else
        return _WorldSpaceLightPos0.xyz;
        #endif
    #endif
}
```
判断逐像素光源的类型通过判断内置宏是否定义，使用#ifdef指令或#ifndef指令。

使用预编译指令#pragma multi_compile_fwdbase(或#pragma multi_compile_fwdadd)的情况下，如果判断得知是平行光的话，光源方向可以直接由 **_WorldSpaceLightPos0.xyz** 得到；如果是点光源或聚光灯，那么 **_WorldSpaceLightPos0.xyz** 表示的是世界空间下的光源位置。

## 光照衰减

Unity在内部使用一张名为_LightTexture0的纹理，其中_LightTexture0对角线上的纹理颜色值，表明了在光源空间中不同位置的点的衰减值，由此来获取光源衰减。

为了对_LightTexture0纹理采样得到给定点到该光源的衰减值，我们首先需要得到该点在**光源空间**中的位置，这是通过_LightMatrix0变换矩阵得到的。
```C++
float3 lightCoord = mul(_LightMatrix0, float4(i.worldPosition, 1)).xyz;
```
然后，使用这个坐标的模的平方对衰减纹理进行对角线上的采样，再使用宏UNITY_ATTEN_CHANNEL来得到衰减纹理中衰减值所在的分量，以得到最终的衰减值：
```C++
fixed atten = tex2D(_LightTexture0, dot(lightCoord,  lightCoord).rr).UNITY_ATTEN_CHANNEL;
```

## 阴影

### Shadow Map
计算阴影区域最常使用的是一种名为Shadow Map的技术。首先把摄像机的位置放在与光源重合的位置上，那么场景中该光源的阴影区域就是那些摄像机看不到的地方。

在前向渲染路径中，如果场景中最重要的平行光开启了阴影，Unity就会为该光源计算它的**阴影映射纹理（shadowmap）**。

shadowmap本质上也是一张深度图，它记录了从该光源的位置出发、能看到的场景中距离它最近的表面位置（深度信息）。

计算shadowmap一种方法是，先把摄像机放置到光源的位置上，然后按正常的渲染流程，即调用Base Pass和Additional Pass来更新深度信息，得到阴影映射纹理。但这种方法会对性能造成一定的浪费，因为我们实际上仅仅需要深度信息而已。

因此，Unity选择使用一个额外的Pass来专门更新光源的shadowmap，**这个Pass就是LightMode标签被设置为ShadowCaster的Pass**。

Unity首先把摄像机放置到光源的位置上，然后调用该Pass，通过对顶点变换后得到**光源空间**下的位置，并据此来输出深度信息到shadowmap中。

当开启了光源的阴影效果后，底层渲染引擎首先会在当前渲染物体的Unity Shader中找到LightMode为ShadowCaster的Pass。如果没有，它就会在**Fallback指定的Unity Shader中继续寻找**（除透明度混合以外的Unity内建Shader都带这个Pass）。如果仍然没有找到，该物体就无法向其他物体投射阴影（但它仍然可以接收来自其他物体的阴影）。

**内建Shader的源码可在 https://unity3d.com/get-unity/download/archive 下载，选择Built in shaders即可**。

### Screenspace Shadow Map
屏幕空间的阴影映射技术 （Screenspace Shadow Map），本是延迟渲染中产生阴影的方法，需要显卡支持MRT，有些移动平台不支持这种特性。

Unity会在ShadowCaster的Pass中，根据光源的阴影映射纹理和摄像机的深度纹理来得到**屏幕空间的阴影图**，表明某些片元**虽然是可见的，但是却处于该光源的阴影中**。通过这样的方式，阴影图就包含了**屏幕空间中所有有阴影的区域**。

如果想要一个物体接收来自其他物体的阴影，只需要在Shader中对阴影图进行采样。由于阴影图是屏幕空间下的，因此，我们首先需要把表面坐标**从模型空间变换到屏幕空间中，然后使用这个坐标对阴影图进行采样**，把采样结果和最后的光照结果**相乘**来产生阴影效果。

### 阴影实现

在Unity中，我们可以选择是否让一个物体投射或接收阴影。这是**通过设置Mesh Renderer组件中的Cast Shadows和Receive Shadows属性**来实现的。

#### Cast Shadows
- 开启（On）或关闭（Off）
- Two Sided，在默认情况下，在计算光源的shadowmap时会剔除掉物体的背面。设置为Two Sided则允许对物体的所有面都计算阴影信息。

#### Receive Shadows

开启了Receive Shadows并不能直接实现接受阴影的效果，还需要在Shader中对阴影颜色进行混合。

**需要使用AutoLight.cginc中三个内置宏SHADOW_COORDS、SHADOW_TANSFER、SHADOW_ATTENUATION**。

**需要保证：a2v结构体中的顶点坐标变量名必须是vertex ，顶点着色器的输入结构体a2v必须命名为v ，且v2f中的顶点位置变量必须命名pos**

- **顶点着色器的输出结构体v2f**中添加SHADOW_COORDS：
```C++
struct v2f {
    float4 pos : SV_POSITION;
    float3 worldNormal : TEXCOORD0;
    float3 worldPos : TEXCOORD1;
    SHADOW_COORDS(2)  //注意无;号，参数数字是TEXCOORD序号
};
```
- 顶点着色器返回之前添加TRANSFER_SHADOW ：
```C++
v2f vert(a2v v) {
    v2f o;  
    ...     
    // Pass shadow coordinates to pixel shader
    TRANSFER_SHADOW(o);
    return o;
}
```
- 在片元着色器中计算阴影值SHADOW_ATTENUATION ：
```C++
// Use shadow coordinates to sample shadow map
fixed shadow = SHADOW_ATTENUATION(i);
```
**使用UNITY_LIGHT_ ATTENUATION可以同时计算光照衰减和阴影**
```C++
fixed4 frag(v2f i) : SV_Target {
    ...
    UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos); //使用三个参数，结果变量名（无需外部声明）、v2f输入、世界空间坐标
    return fixed4(ambient + (diffuse + specular) * atten, 1.0);
}
```

### 透明物体的阴影

- 透明度测试

把FallBack设置为Transparent/Cutout/VertexLit，并将Cast Shadows设置为Two Sided即可。

需要注意的是 ，由于Transparent/Cutout/VertexLit中计算透明度测试时，使用了名为_Cutoff的属性来进行透明度测试，因此，这要求我们的Shader中也**必须提供名为_Cutoff的属性**。否则，同样无法得到正确的阴影结果。

- 透明度混合

一般情况下不会投射阴影，但是通过把它们的Fallback设置为VertexLit、Diffuse这些不透明物体使用的Unity Shader可以强制投射阴影。

# 高级纹理
## 立方体纹理
立方体纹理在Shader属性中类型为Cube，CG代码中类型为samplerCUBE

立方体纹理的采样使用texCUBE函数，用一个矢量进行采样。将以立方体纹理的中心为起点，沿矢量方向与立方体相交的点为采样点。

**创建立方体纹理有三种方法**：
- 直接由一些特殊布局的纹理创建

提供一张具有特殊布局的纹理，例如类似立方体展开图的交叉布局、全景布局等。然后，把该纹理的Texture Shape设置为Cube即可。

此方法可以对纹理数据进行压缩，而且可以支持边缘修正、光滑反射（glossy reflection）和HDR等功能。

- 手动创建一个Cubemap资源（位于Legacy），再把6张图赋给它

是Unity 5之前的版本中使用的方法。

- 由脚本生成

Camera.RenderToCubemap(Cubemap)函数可以把Camera观察到的场景图像存储到6张图像中，从而创建出该位置上对应的立方体纹理，写入到Cubemap参数中。

### 天空盒子
天空盒子（Skybox）可使用运用了立方体纹理的材质。

Window → Rendering → Lighting Settings → Skybox可设置Skybox使用的Material。Shader选择使用Skybox/Cubemap。

### 反射和折射

由光路的可逆性，对观察方向矢量viewDir的相反矢量-viewDir进行反射或折射计算，得到的结果就是采样所需的矢量。
```C++
reflectDir = reflect(-viewDir, normal);
fixed3 reflection = texCUBE(_Cubemap, reflectDir).rgb;

//第三个参数:= 入射介质折射率/折射介质折射率，光线与法线夹角大小与折射率的大小成反比
refractDir = refract(-viewDir, normal, _RefractRatio); //需要归一化
fixed3 refraction = texCUBE(_Cubemap, refractDir).rgb;
```
**使用线性插值lerp与漫反射混合**
```
//float3 lerp(float3 a, float3 b, float w) { return a + w*(b-a)};
//或者写成(1-w)*a+w*b，w=1时为b，w=0时为a
lerp(diffuse, reflection, _ReflectAmount);
```

### 菲涅尔反射

菲涅耳反射描述了一种光学现象，即当光线照射到物体表面上时，一部分发生反射，一部分进入物体内部，发生折射或散射。**被反射的光和入射光之间存在一定的比率关系**，这个比率关系可以通过菲涅耳等式进行计算。

- Schlick 菲涅耳近似等式

F+(1-F)\*(1-dot(v,n))^5，F是一个反射系数，用于控制菲涅耳反射的强度， v是视角方向，n表面法线。

- Empricial 菲涅耳近似等式

saturate(bias+scale\*(1-dot(v,n))^power)，是上式的推广

**菲涅耳反射计算结果一般用于lerp的第三个系数**
```
lerp(diffuse, reflection, saturate(fresnel));
```
**注意一些实现也会直接把fresnel和反射光照相乘后叠加到漫反射光照上**

## 渲染纹理
现代的GPU允许我们**把整个三维场景渲染到一个中间缓冲中**，即渲染目标纹理（Render Target Texture，RTT），而不是传统的帧缓冲或后备缓冲（back buffer）。

多重渲染目标 （Multiple Render Target，MRT ，这种技术指的是GPU允许我们把场景同时渲染到多个渲染目标纹理中，而不再需要为每个渲染目标纹理单独渲染完整的场景。**延迟渲染就是使用多重渲染目标的一个应用**。

### 使用方式

- 直接创建Render Texture

可将作Render Texture设置为Camera的Target Texture，则该Camera会把观察到的场景实时渲染到Render Texture中。

- 屏幕后处理

调用GrabPass命令或OnRenderImage函数可获取当前屏幕图像。

Unity会把这个屏幕图像放到一张和屏幕分辨率等同的渲染纹理中，使我们可以在自定义的Pass中进行调用进而处理，从而实现各种屏幕特效。

GrabPass中的字符串为屏幕图像的Render Texture变量名称
```
GrabPass{"_TextureName"}
```
也可以直接使用```GrabPass{}```，此时使用_GrabTexture来访问屏幕图像的Render Texture。但是，当场景中有多个物体都使用了这样的形式来抓取屏幕时，这种方法的性能消耗比较大，因为**对于每一个使用它的物体，Unity都会为它单独进行一次昂贵的屏幕抓取操作。但这种方法可以让每个物体得到不同的屏幕图像**。

使用了自定义命名时，Unity只会在每一帧时为第一个使用名为_TextureName的纹理的物体执行一次抓取屏幕的操作。这种方法更高效，因为不管场景中有多少物体使用了该命令，**每一帧中Unity都只会执行一次抓取工作，但这也意味着所有物体都会使用同一张屏幕图像**。不过，在大多数情况下这已经足够了。

在其他Pass中需要变量声明
```
sampler2D _TextureName;
float4 _TextureName_TexelSize;
```
\_TextureName_TexelSize可以让我们得到该纹理的纹素大小，例如一个大小为256×512的纹理，它的纹素大小为(1/256, 1/512)。

### 两种使用方式对比

从效率上来讲，直接创建Render Texture的效率往往要好于GrabPass，尤其在移动设备上。Render Texture可以自定义渲染纹理的大小，尽管**这种方法需要把部分场景再次渲染一遍**（因为要调用Camera）。

**但我们可以通过调整摄像机的渲染层来减少二次渲染时的场景大小，或使用其他方法来控制摄像机是否需要开启**。

而使用GrabPass获取到的**图像分辨率和显示屏幕是一致的**，这意味着在一些高分辨率的设备上可能会造成严重的带宽影响。

并且GrabPass往往**需要CPU直接读取后备缓冲（back buffer）中的数据，破坏了CPU和GPU之间的并行性**，这是比较耗时的。

**Unity引入了命令缓冲（Command Buffers）来允许我们扩展Unity的渲染流水线**。使用命令缓冲我们也可以得到类似抓屏的效果，它可以在不透明物体渲染后把当前的图像复制到一个临时的渲染目标纹理中，然后在那里进行一些额外的操作，例如模糊等，最后把图像传递给需要使用它的物体进行处理和显示。除此之外，命令缓冲还允许我们实现很多特殊的效果。官方文档： http://docs.unity3d.com/
Manual/Graphics CommandBuffers.html

### 镜子效果
在镜子表面上放置一个Camera，创建Render Texture并设置为Target Texture。然后在Shader中使用该Render Texture进行反转处理即可。
```
o.uv.x = 1-o.ux.x;
```

### 模拟折射扭曲效果
使用GrabPass获取屏幕图像
```
GrabPass{"_RefractionTex"}
```
首先在顶点着色器中使用ComputeGrabScreenPos计算屏幕空间的坐标
```
o.pos = UnityObjectToClipPos(v.vertex);
o.scrPos = ComputeGrabScreenPos(o.pos);
```
使用法线纹理，用切线空间下的法线计算偏移
```
float2 offset = tNormal.xy*_Distortion*_RefractionTex_TexelSize.xy;
i.scrPos.xy += offset;
```
然后做透视除法对_RefractionTex（GrabPass得到的Render Texture）进行采样，即得到扭曲后的屏幕图像
```
fixed3 refractColor = tex2D(_RefractionTex,i.scrPos.xy/i.scrPos.w).rgb;
```
最后用lerp混合即可
```
return fixed4(lerp(reflectColor,refractColor,_RefractAmount),1);
```

## 程序纹理
程序纹理即通过脚本生成纹理并赋予材质

先在脚本中声明需要使用该纹理的材质
```
public Material material = null;
```
再声明纹理
```
private Texture2D m_generatedTexture = null;
```
如果material没有手动赋值则为null，此时尝试通过`renderer.sharedMaterial;`获取该脚本所在的物体上得到相应的材质。在函数_UpdateMaterial中使用material.SetTexture设置纹理。
```
void Start () {
    if (material == null) {
        Renderer renderer = gameObject.GetComponent<Renderer>();
        if (renderer == null) {
            Debug.LogWarning("Cannot find a renderer.");
            return;
        }
        material = renderer.sharedMaterial;
    }
    _UpdateMaterial();
}

private void _UpdateMaterial() {
    if (material != null) {
        m_generatedTexture = _GenerateProceduralTexture();
        material.SetTexture("_MainTex", m_generatedTexture);
    }
}
```
在函数_GenerateProceduralTexture中：

- 使用`Texture2D proceduralTexture = new Texture2D(textureWidth, textureHeight);`实例化一个纹理

- 使用`proceduralTexture.SetPixel(x, y, pixel);`绘制纹理上的像素，pixel类型是Color

- 使用`proceduralTexture.Apply();`写入像素值，只需在所有像素绘制完成后调用一次

### 程序材质
在Unity中，有一类专门使用程序纹理的材质，叫做**程序材质（Procedural Materials）**

程序材质和它使用的程序纹理并不是在Unity中创建的，而是使用了一个名为**Substance Designer**的软件在Unity外部生成的。

Substance Designer是一个非常出色的纹理生成工具，很多3A的游戏项目都使用了由它生成的材质。可以从Unity的资源商店或网络中获取到很多免费或付费的Substance材质。**这些材质都是以.sbsar为后缀的**。

当把这些文件导入Unity后，Unity就会生成一个程序纹理资源 （Procedural Material Asset）。**一个程序纹理资源可以包含一个或多个程序材质**。
