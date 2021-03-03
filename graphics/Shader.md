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
