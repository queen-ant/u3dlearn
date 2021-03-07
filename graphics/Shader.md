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
{"DisableBatching"} //关闭批处理
{"ForceNoShadowCasting"} //禁止投射阴影
{"IgnoreProjector"} //忽略projector
{"CanUseSpriteAtlas"} //是否用于精灵图
{"PreviewType"} //面板预览类型

//上述标签仅可以在SubShader 中声明，而不可以在Pass块中声明
```
- RenderSetup（渲染状态设置）

这些指令可以设置显卡的各种状态
```
Cull 剔除。值：Back ｜ Front｜ Off，剔除背面/正面/关闭剔除。
ZTest 深度测试模式。值：Less Greater｜ LEqual｜ GEqual｜ Equal｜ NotEqual｜ Always，设置深度测试时使用的函数
ZWrite 是否开启深度写入。值：On ｜ Off，开启/关闭。
Blend 混合模式。值：SrcFactor DstFactor，开启并设置混合模式
```
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

将方向矢量从object space变换到world space

- inline float4 UnityWorldToClipPos( in float3 pos )

将位置矢量从world space变换到clip space

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
```
sampler2D _MainTex;
float4 _MainTex_ST;
```
- TRANSFORM_TEX
```
#define TRANSFORM_TEX(tex,name) (tex.xy * name##_ST.xy + name##_ST.zw)

o.uv.xy = TRANSFORM_TEX(v.texcoord,_MainTex);
```
TRANSFORM_TEX即应用缩放和偏移到纹理坐标

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
