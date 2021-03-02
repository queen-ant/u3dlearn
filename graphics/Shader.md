# 渲染过程（概念）
## 应用阶段
- 布置场景数据
- 粗粒度剔除
- 设置渲染状态

**最终输出渲染图元，图元（primitive）即点、线、三角面。**

## 几何阶段
主要工作是把顶点信息从local space转换到screen space

- 顶点shader（Vertex Shader） 可编程，必须实现
- 曲面细分shader（Tessellation Shader） 可编程，选择实现
- 几何shader（Geometry Shader） 可编程，选择实现
- 裁剪（Clipping） 不可编程、可配置
- 屏幕映射（Screen Mapping） 不可编程、不可配置

最终输出顶点信息

## 光栅化阶段
- 三角形设置（Triangle Setup） 不可编程、不可配置
- 三角形遍历（Triangle Traversal） 不可编程、不可配置
- 片元shader（Fragment Shader） 可编程，选择实现
- 逐片元操作(Per-Fragment Operations) 不可编程、可配置

最终产生像素

# 渲染过程（硬件）
## CPU
- 将顶点数据（图元列表）装入内存，然后从内存装入显存
- 设置渲染状态
- 向GPU发送draw call

GPU收到draw call之后工作
## GPU
- 几何阶段
- 光栅化阶段
