指南 - CUDA-光栅化器
========================

这个项目的截止日期是**10月16日星期二午夜**。

**概述：**
在这个项目中，你将使用CUDA实现一个简化的光栅化图形管线，类似于OpenGL管线。你将实现顶点着色、图元组装、光栅化、片段着色和帧缓冲区。有关光栅化图形管线的更多信息可以在课堂幻灯片和CIS 560讲义中找到。

提供的基础代码包括glTF加载器(tinygltfloader)以及大部分I/O和簿记代码。它还包括一些你可能会发现有用的函数，如下所述。核心光栅化管线留给你来实现。

如果你不想使用这个基础代码，你不必使用它。你也可以随意更改基础代码的任何部分。
**这是你自己的项目。**

**建议：**
你保存的每张图像应该自动获得不同的文件名。不要全部删除！为了你的README，保留一些图像，这样你就可以选择一些来记录你的进度。

**重要：**
- 如果你不是CGGT/DMD专业的，你可以用GPU计算项目代替这个项目。你必须在继续之前得到Ottavio的预先批准！
- 你可以选择用Vulkan重写这个项目来获得大量额外分数。警告：这是一个非常困难的任务。

### 内容

* `src/` C++/CUDA源文件。
* `util/` C++实用工具文件。
* `gltfs/` 示例glTF测试文件
  * `gltfs/triangle/triangle.gltf`（仅1个三角形，从这个开始）
  * `gltfs/box/box.gltf`（8个顶点，12个三角形，从这个开始）
  * `gltfs/cow/cow.gltf`
  * `gltfs/duck/duck.gltf`（有一个漫反射纹理）
  * `gltfs/checkerboard/checkerboard.gltf`（有一个漫反射纹理，可用于测试透视正确插值）
  * `gltfs/CesiumMilkTruck/CesiumMilkTruck.gltf`（有几个纹理）
  * `gltfs/flower/flower.gltf`（从大多数角度看，模型有很多层）
  * `gltfs/2_cylinder_engine/2_cylinder_engine.gltf`（相对复杂的模型，没有纹理，需要重新缩放以显示在屏幕中央）
* `renders/` 测试实现渲染结果，duck.gltf。
* `external/` 第三方库的包含文件和静态库。

### 运行代码

主函数需要一个glTF模型文件。用一个作为参数调用程序：`cis565_rasterizer gltfs/duck/duck.gltf`。
（在Visual Studio中，`../gltfs/duck/duck.gltf`。）

如果你使用Visual Studio，可以在"项目属性"的"调试 > 命令参数"部分设置这个。请注意，这个值对于每种不同的配置类型都是不同的。你也可以为所有配置设置参数以简化这一点。确保你的路径正确；查看控制台是否有错误。

你也可以从命令行启动构建的程序来输入参数。

## 要求

**如有任何疑问，请在邮件列表中询问。**

在这个项目中，你会得到以下代码：

* 一个小型glTF加载器，用于加载glTF格式模型并将它们转换为OpenGL风格的索引和顶点属性数据缓冲区。
    * [glTF](https://github.com/KhronosGroup/glTF)是一种标准模型格式（Patrick是主要贡献者之一）。
    除非你想支持三角形以外的图元，否则不需要担心其细节，因为这些都已经为你完成了。
* 管线某些部分的结构体。
* 片段缓冲区到帧缓冲区的复制。
* CUDA-GL互操作。
* 使用鼠标的简单交互式相机。

你需要实现以下特性/管线阶段：

* 顶点着色。（`rasterize.cu`中的`_vertexTransformAndAssembly`）
* 图元组装，支持从索引和顶点数据缓冲区读取三角形。（代码已提供，只需取消注释）（`rasterize.cu`中的`_primitiveAssembly`）
* 光栅化。（创建你自己的函数，在`rasterize`中调用它）
* 片段着色。（创建你自己的函数，在`rasterize`中调用它）
* 用于存储和深度测试片段的深度缓冲区。（已为你提供了`int * dev_depth`，你始终可以更改为你自己的版本）
* 片段到深度缓冲区的写入（**使用**原子操作避免竞争）。
* （片段着色器）简单的光照方案，如Lambert或Blinn-Phong。（`rasterize.cu`中的`render`）

更多指导见下文。

你还需要实现至少2.0"分数"值的额外功能。
（括号中给出了分数值）：

* (1.0) 在一个功能中使用共享内存。想法和建议：
   * 寻找那些你可能需要在单个线程中多次访问相同"块"内存的区域
   * 后处理，将片段缓冲区分为"瓦片" - SSAO？Bloom？卡通着色？
   * 共享内存统一变量 - 蒙皮矩阵数组？光源数组？
   * 你能否将共享内存用于某些基于纹理的着色器？
* (2.0) [基于瓦片的管线](https://github.com/CIS565-Fall-2015/cis565-fall-2015.github.io/blob/master/lectures/10-Mobile-Graphics.pptx?raw=true)
* 额外的管线阶段。
   * (1.0) 曲面细分着色器。
   * (1.0) 几何着色器，能够为每个输入图元输出可变数量的图元，使用流压缩优化（允许使用thrust）。
   * (0.5 **如果不做几何着色器**) 背面剔除，使用流压缩优化（允许使用thrust）。
   * (1.0) 变换反馈。
   * (0.5) 混合（写入帧缓冲区时）。
* (1.0) 实例化：多次绘制一组顶点数据，每次运行顶点着色器时使用不同的ID。
* (0.5) 图元上点之间的正确颜色插值。
* (1.0) UV纹理映射，具有双线性纹理过滤和透视正确的纹理坐标。
* 支持光栅化额外的图元：
   * (0.5) 线或线带。
   * (0.5) 点。
   * 对于光栅化线和点，你可以从切换模式开始，该模式将你的管线从显示三角形切换到显示线框或点云。
* 抗锯齿
   * (0.5) SSAA - 超级采样抗锯齿
   * (1.0) [MSAA](https://mynameismjp.wordpress.com/2012/10/24/msaa-overview/) - 多重采样抗锯齿，并与前者进行性能比较
* (1.0) 遮挡查询。
* (1.0) 使用k缓冲区的顺序无关透明度。

这个额外功能列表并不全面。如果你有特别想要实现的想法，请**先联系我们**。

**重要：**
对于每个额外功能，请提供以下简要分析：

* 功能的简明概述。
* 添加该功能的性能影响（更慢或更快）。
  * 性能瓶颈在哪里？
  * 性能改进在哪里？
* 如果你做了一些事情来加速该功能，你做了什么以及为什么？
* 这个功能如何在你当前的实现之外进行优化？

## 基础代码导览

你将主要在以下文件中工作：`rasterize.cu`。
需要完成的区域标有`TODO`注释。有用的参考函数标有`CHECKITOUT`注释。

* `src/rasterize.cu`包含核心光栅化管线。
  * 包含一些预先制作的结构体供你使用，但标有TODO的结构体也是简单光栅化器所需的。与基础代码的任何部分一样，你可以根据需要修改或替换这些结构体。
  * `PrimitiveDevBufPointers`新加载的属性缓冲区、纹理指针等，来自glTF模型。
    一切都是基本数据类型或已经复制到设备内存中。
  * `VertexOut`组装的顶点，具有从位置到纹理坐标的各种属性。
  * `Primitive`组装的图元，带有顶点
  * `Fragment`通过深度/剪裁/模板测试的最终片段，等待着色

* `src/rasterizeTools.h`包含各种有用的工具
  * 包括许多与重心坐标相关的函数，这些函数在实现基于扫描线的光栅化时可能有用。

* `util/utilityCore.hpp`作为有用函数的大杂烩。

## 光栅化管线

下面描述了可能的管线。给出了伪类型签名。并非所有伪代码数组在实践中都一定会存在。

### 初次尝试管线

这描述了*一种可能的*图形管线的最小版本，类似于现代硬件（DX/OpenGL）。你的不必完全匹配。首先，尝试按这里描述的那样编写最少量的代码。在实现每个管线步骤后验证一些输出。这将减少必要的调试时间。

首先用一些简单的模型（`box.gltf`）进行测试。

* 用某个默认值清除片段缓冲区。
* 顶点着色：
  * `VertexIn[n] vs_input -> VertexOut[n] vs_output`
  * 最小的顶点着色器不会应用任何变换 - 它直接在标准化设备坐标（每个维度-1到1）中绘制世界位置。
* 图元组装。
  * `VertexOut[n] vs_output -> Triangle[t] primitives`
  * 实际上已提供代码，只需取消注释。（因为你可能需要阅读glTF设置代码才能自己完全实现这一点）
* 光栅化。
  * `Triangle[t] primitives -> Fragment[m] rasterized`
  * 扫描线实现开始时很简单。
  * 在三角形之间并行化。现在，在每个线程中循环遍历片段缓冲区中的每个像素。
  * 请注意，你不会有任何实际分配的大小为`m`的数组。
* 片段到深度缓冲区。
  * `Fragment[m] rasterized -> Fragment[width][height] depthbuffer`
    * `depthbuffer`用于存储和深度测试片段。
  * 会导致竞争条件 - 在它正常工作之前不要费心修复这些！
  * 如果你为每个片段（包括那些被遮挡的片段）从光栅化内核调用片段着色，则实际上可以在片段着色内部/之后完成。在片段着色之前这样做可能更快（为什么？）但意味着片段着色器不能改变深度。
* 片段着色。
  * `Fragment[width][height] depthbuffer ->`
  * 一个超级简单的测试片段着色器：为每个片段输出相同的颜色。
    * 也尝试显示各种调试视图（法线等）。
* 片段到帧缓冲区的写入。
  * `-> vec3[width][height] framebuffer`
  * 简单地将片段着色器结果保存到帧缓冲区中（以便在屏幕上显示）。

### 实用管线

* 用一些默认值清除片段和深度缓冲区。
  * 你应该能够将默认值传递给清除函数，以便设置清除颜色（背景）、清除深度等。
* 顶点着色：
  * `VertexIn[n] vs_input -> VertexOut[n] vs_output`
  * 应用一些顶点变换（例如使用`glm::lookAt`和`glm::perspective`的模型-视图-投影矩阵）。
* 图元组装。
  * `VertexOut[n] vs_output -> Triangle[t] primitives`
  * 如上所述。
  * 其他图元类型是可选的。
* 光栅化。
  * `Triangle[t] primitives -> Fragment[m] rasterized`
  * 你可以选择使用基于共享内存的瓦片光栅化方法，这应该有较低的全局内存带宽。它也会改变管线的其他部分 - 只有在你有一个工作的"实用"管线后才尝试这个，既为了理智也为了比较。
  * 在三角形之间并行化，但现在避免循环遍历所有像素：
    * 在光栅化三角形时，只扫描三角形周围的盒子（`getAABBForTriangle`）。
* 片段到深度缓冲区。
  * `Fragment[m] rasterized -> Fragment[width][height] depthbuffer`
    * `depthbuffer`用于存储和深度测试片段。
  * 这可以在片段着色之前完成，这可以防止片段着色器改变片段的深度。
    * 这种顺序导致一个优化：它允许你在复杂的片段着色器代码中花费执行时间之前进行深度测试！
    * 如果你想能够改变片段的深度，你需要做一个适应。例如，你可以添加一个单独的着色器阶段，该阶段在光栅化期间发生，可以改变深度。或者，你可以从光栅化步骤调用片段着色器 - 但要注意性能会差得多 - 由于每个线程的可变运行长度，占用率会很低。
  * 处理竞争条件！由于多个图元将片段写入深度缓冲区中的同一个片段，必须通过使用CUDA原子操作来避免竞争。
    * *方法1：*在线程比较旧片段和新片段深度（并可能写入新片段）期间锁定深度缓冲区中的位置。这在所有情况下都应该有效，但会更慢。参见下面关于实现此功能的部分。
    * *方法2：*将深度值转换为定点`int`，并使用`atomicMin`将其存储到`int`类型的深度缓冲区`intdepth`中。之后，存储在`intdepth[i]`中的值（通常）是应该存储到`fragment`深度缓冲区中的片段的值。
      * 这可能会导致一些罕见的竞争条件（例如，跨块）。
    * `flower.gltf`测试文件适合测试竞争条件。
* 片段着色。
  * `Fragment[width][height] depthbuffer ->`
  * 添加着色方法，如Lambert或Blinn-Phong。灯光可以由内核参数定义（如GLSL uniforms）。
* 片段到帧缓冲区的写入。
  * `-> vec3[width][height] framebuffer`
  * 简单地将颜色从深度缓冲区复制到帧缓冲区中（以便在屏幕上显示）。

这是管线步骤的建议序列，但你可以根据需要更改此序列的顺序或合并整个内核。例如，如果你认为这样做有好处，你可以选择合并顶点着色器和图元组装内核，或将透视变换合并到另一个内核中。没有必然正确的内核序列，你可以选择任何有效的序列。请在你的README中记录你选择的序列及原因。

## 资源

### CUDA互斥锁

改编自[这个StackOverflow问题](http://stackoverflow.com/questions/21341495/cuda-mutex-and-atomiccas)。

```cpp
__global__ void kernelFunction(...) {
    // 获取互斥锁的指针，现在应该是0。
    unsigned int *mutex = ...;

    // 循环等待，直到此线程能够执行其临界区。
    bool isSet;
    do {
        isSet = (atomicCAS(mutex, 0, 1) == 0);
        if (isSet) {
            // 临界区代码放在这里。
            // 临界区必须在等待循环内；
            // 如果它在之后，将发生死锁。
        }
        if (isSet) {
            mutex = 0;
        }
    } while (!isSet);
}
```

### 链接

以下资源可能对这个项目有用。

* 线条光栅化幻灯片，MIT EECS 6.837，Teller和Durand
  * [幻灯片](http://groups.csail.mit.edu/graphics/classes/6.837/F02/lectures/6.837-7_Line.pdf)
* 在GPU上的高性能软件光栅化
  * [论文 (HPG 2011)](http://www.tml.tkk.fi/~samuli/publications/laine2011hpg_paper.pdf)
  * [代码](http://code.google.com/p/cudaraster/)
  * 请注意，为了参考论文而查看此代码是可以的，但我们很可能不会批准任何将这些代码实际合并到你的项目中的请求。
  * [幻灯片](http://bps11.idav.ucdavis.edu/talks/08-gpuSoftwareRasterLaineAndPantaleoni-BPS2011.pdf)
* Direct3D 10系统 (SIGGRAPH 2006) - 对那些有兴趣做几何着色器和变换反馈的人
  * [论文](http://dl.acm.org/citation.cfm?id=1141947)
  * [论文，通过Penn Libraries代理](http://proxy.library.upenn.edu:2247/citation.cfm?id=1141947)
* 使用k-Buffer在GPU上实现多片段效果 - 对于那些想要使用k-buffer实现顺序无关透明度的人
  * [论文](http://www.inf.ufrgs.br/~comba/papers/2007/kbuffer_preprint.pdf)
* FreePipe：用于高效多片段效果的可编程并行渲染架构 (I3D 2010)
  * [论文](https://sites.google.com/site/hmcen0921/cudarasterizer)
* 用Javascript编写软件光栅化器
  * [第1部分](http://simonstechblog.blogspot.com/2012/04/software-rasterizer-part-1.html)
  * [第2部分](http://simonstechblog.blogspot.com/2012/04/software-rasterizer-part-2.html)
* OpenGL如何工作：500行代码中的软件渲染
  * [Wiki](https://github.com/ssloy/tinyrenderer/wiki)

## 第三方代码政策

* 使用任何第三方代码必须通过在我们的Google Group上询问来获得批准。
* 如果获得批准，所有学生都可以使用它。通常，我们批准使用不是项目核心部分的第三方代码。例如，对于路径追踪器，我们会批准使用第三方库来加载模型，但不会批准复制和粘贴用于折射的CUDA函数。
* 第三方代码**必须**在README.md中注明出处。
* 未经批准使用第三方代码，包括使用其他学生的代码，是违反学术诚信的行为，至少会导致你在本学期获得F。

## README

* 项目的简要描述和你实现的具体功能。
* 至少一张你的项目运行的截图。
* 你的项目运行的30秒或更长的视频。
* 性能分析（如下所述）。

### 性能分析

性能分析是你将使用在课堂上学到的技能，研究如何使你的CUDA程序更有效率的地方。你必须对你的代码进行至少一项实验，以调查对性能的积极或消极影响。

我们鼓励你在调整方面发挥创意。考虑你的代码中可能被视为瓶颈的地方，并尝试改进它们。

提供你的优化摘要（不超过一页），以及表格和或图表，以直观地解释任何性能差异。

* 包括几个不同模型在每个管线阶段花费的时间的细分。建议使用饼图或100%堆叠条形图。
* 对于优化步骤（如背面剔除），包括性能比较以显示其有效性。

## 提交

如果你修改了任何`CMakeLists.txt`文件（除了`SOURCE_FILES`列表外），请明确提及。
注意Google Group上讨论的任何构建问题。

打开GitHub pull request，以便我们可以看到你已经完成。
标题应为"Project 4: YOUR NAME"。
你的pull request评论部分的模板如下，你可以进行一些复制和粘贴：

* [Repo Link](https://link-to-your-repo)
* （简要）提及你已完成的功能。特别是那些你想要强调的额外功能
    * 功能0
    * 功能1
    * ...
* 对项目本身的反馈，如果有的话。 