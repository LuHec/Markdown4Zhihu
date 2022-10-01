#  一篇搞定Unity Shader入门精要




# 第二章：Shader和渲染管线



## 2.1 什么是Shader

- GPU流水线上高度可编程阶段。
- 有特定类型的着色器，如顶点着色器，片元着色器。
- 依靠着色器可以控制流水线中的渲染细节。



## 2.2 什么是顶点、片元

- 顶点就是点，包含了空间坐标信息。
- 图元是由顶点组成的。一个顶点，一条线段，一个三角形或者多边形都可以成为图元。
- 片元是在图元经过光栅化阶段后，被分割成一个个像素大小的基本单位。
- 像素是最终呈现在屏幕上的包含RGBA值的图像最小单元。



## 2.3 渲染流水线

- 计算机从一系列的顶点数据，纹理等信息出发，将这些信息转换为图像，这个工作通常由GPU和CPU共同完成。


- 渲染流水线是概念流水线。




### 2.3.1 渲染流程的三个阶段

每个阶段都包含子流水线阶段。

![image-20220930161401466](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161401466.png)

##### 1.应用阶段：

通常由CPU实现。

这一阶段由三个主要任务：

-   准备场景数据
    
-   粗粒度剔除(可以剔除不可见物体，提升渲染性能)
    
-   输出渲染图元。
    

**渲染图元**：可以是点，线，三角面等。渲染图元会传递到几何阶段。

##### 2.几何阶段：

通常在GPU上进行，决定绘制的图元，怎么绘制，在哪里绘制。

##### 3.光栅化阶段：

通常在GPU上进行，使用上个阶段传递的数据来产生像素，渲染最终图像。



### 2.3.2 CPU和GPU的通信——应用阶段

![image-20220930161416817](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161416817.png)

#### 1.把数据加载到显存中

渲染数据从硬盘加载到系统内存(RAM)，网格和纹理等数据被加载到显存(VRAM)。因为显卡对于显存的访问速度更快，且对于RAM无直接访问权限。

加载到显存后就可以移除RAM内部数据，但是一些数据需要保留给CPU访问。从硬盘加载到RAM非常耗时。

![image-20220930161426628](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161426628.png)

#### 2.设置渲染状态：

通过CPU设置渲染状态，指导GPU如何进行渲染工作。

渲染状态定义了场景中的网格是怎么被渲染的，如片元着色器，顶点着色器，光源属性，材质等。

当没有更改渲染状态的时候，所有网格都使用一种渲染状态，看起来像是同一种材质。

#### 3.调用Draw Call:

Draw Call是一个命令，发起方为CPU，接收方GPU。一次DC命令指向一个需要被渲染的图元列表。

GPU会根据渲染状态和所有输入的顶点数据进行计算，生成像素。这就是GPU流水线。

![image-20220930161434273](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161434273.png)



### 2.3.3 GPU流水线

GPU渲染过程就是GPU流水线。

对几何阶段和光栅化阶段开发者无法拥有绝对控制权，实现载体为GPU。

几何阶段和光栅化阶段可以分为若干的流水线阶段，由GPU实现，**每个阶段都提供了不同的可配置性或可编程性**。

![image-20220930161441683](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161441683.png)



### 2.3.4 几何阶段

#### 1.顶点着色器(Vertex Shader)

可以完全编程。

是GPU流水线的第一个过程，输入来自于CPU。处理单位是顶点。

顶点着色器不能判断各个顶点间的关系，也不能创建和销毁顶点。因为这样的独立性，GPU可以利用本身的特性并行化处理每一个顶点，这一阶段处理速度很快。

这个阶段主要完成的任务是：

- **坐标变换**
- **逐顶点光照**：

![image-20220930161451976](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161451976.png)

**坐标变换**：

可以对顶点坐标进行变换，改变顶点的位置。可以通过改变顶点位置来模拟水面和布料。

**顶点着色器最基本的工作**：

将顶点坐标从模型空间转换为齐次裁剪空间。

**NDC**：归一化的设备坐标

![image-20220930161601664](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161601664.png)

#### 2.裁切(Clipping)

这一步不可以编程。

由于场景可能会非常大，摄像机不会覆盖所有物体。在摄像机外的物体会被裁切。

![image-20220930161612337](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161612337.png)

图元和相机视野的关系有

-   全部在外面
    
-   全部在里面
    
-   部分在外面
    
    比如一条线段的一个顶点在内部，一个在外部，那么就会在线段和视野边界的交接处生成一个新的顶点来替代外部的顶点。
    

#### 3.屏幕映射(Screen Mapping)

这一步输入的坐标依然是三维坐标。屏幕映射会将坐标的X,Y值转换到**屏幕坐标系(Screen Coordinates)**下。屏幕坐标决定了该像素点到屏幕的边缘有多远。

而对于Z轴的分量，不会做任何处理。但是仍然会被保留，并和屏幕坐标系构成**窗口坐标系(Window Coordinates)**。这些值会传递到光栅化阶段。

![image-20220930161622714](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161622714.png)

注：OpenGL和DirectX的坐标系并不相同。

![image-20220930161641341](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161641341.png)



### 2.3.5 光栅化阶段

**光栅化就是将顶点数据转换为片元的过程。**

上一个阶段传递了屏幕坐标系下的顶点位置以及额外信息：深度值(Z坐标)，法线方向，视角方向等。

光栅化阶段两个重要目标

-   计算图元覆盖了哪些像素
    
-   为像素计算颜色
    

#### 1.三角形遍历(Triangle Traversal)

三角形遍历阶段会通过几何阶段计算的信息来判断一个三角网格覆盖了哪些像素，并使用三角形的顶点信息对覆盖区域的像素进行插值。

如果一个像素能被三角网格覆盖，就会生成一个**片元(fragment)**。这就叫三角形遍历，也被称作**扫描变换(Scan Conversion)**。

![image-20220930161651540](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161651540.png)

片元是包含很多状态的集合，这些状态用来计算像素的最终颜色。状态包括了（不限于）屏幕坐标、深度信息、顶点信息（法线、纹理坐标等）

#### 2.片元着色器(Fragment Shader)

是另一个重要的可编程阶段。

片元着色器的输入是三角形遍历中插值得到的数据。它的输出是一个或者多个颜色。

这一阶段可以完成很多重要渲染技术，如纹理采样。

片元着色器只能影响单个片元。

![image-20220930161708053](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161708053.png)

#### 3.逐片元操作(Pre-Fargment Operations)

主要任务:

-   决定每个片元可见性。需要进行很多测试，如深度测试和模板测试。
    
-   如果通过测试，就将片元的颜色值和颜色缓冲区的颜色进行合并。
    

这个阶段高度可配置。

只有通过了测试才能和缓冲区颜色混合并写入颜色缓冲区：

![image-20220930161737557](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930161737557.png)

**模板测试(Stencil Test)和深度测试(Depth Test)**

- **模板测试：**

GPU首先会读取模板缓冲区中该片元的模板值，然后和参考值比较。如果没有通过开发者自定的比较函数，那么就舍去该片元。

不管片元有没有通过测试，都可以根据模板测试或者深度测试的结果来修改模板缓冲区，也是由开发者自定义。

此测试还可以用于渲染阴影，轮廓等。

- **深度测试：**

基本的流程与模板测试差不多。一般来说，如果深度值比较缓冲区中较小，就会丢弃该片元。我们想保留距离相机较近的物体，舍弃较远的或是被遮挡的。

与模板测试不同，如果深度测试不通过，就没有资格修改深度缓冲区中的值。如果通过，开发者可以选择是否覆盖原来的深度值。这是通过开启/关闭深度写入来实现的。

**透明效果与这两个测试关系非常密切**

![image-20220915153652175](file:///C:/Users/41613/AppData/Roaming/Typora/typora-user-images/image-20220915153652175.png?lastModify=1664524986)

如果通过了测试，那么片元就来到**合并**功能面前。

**为什么需要合并?**

渲染是一个接着一个物体绘制到画面上的，颜色缓冲区存储了上一次渲染的结果。此时是覆盖之前的结果还是作其他处理，这就是合并需要解决的问题。

**混合操作：**

GPU会通过混合函数将缓冲区颜色和片元颜色进行混合，类似PS的图层混合模式，图层混合模式决定了该图层和下一图层的混合效果。

-   对于不透明物体，可以关闭混合操作，这样片元着色器得到的颜色就会覆盖掉缓冲区的颜色。
    
-   对于半透明物体，就需要开启混合操作，让这个物体看起来是透明的。
    

![image-20220915160411301](file:///C:/Users/41613/AppData/Roaming/Typora/typora-user-images/image-20220915160411301.png?lastModify=1664524986)

一般来说，GPU都会在片元着色器执行前就进行测试，这样就防止了计算完颜色后却是不需要的信息。

但是提前可能会造成一些问题，如透明测试等。因此GPU在遇到冲突时会禁用预先测试。这样会造成性能下降。

![image-20220915161016596](file:///C:/Users/41613/AppData/Roaming/Typora/typora-user-images/image-20220915161016596.png?lastModify=1664524986)

#### 双重缓冲

为了避免用户看到正在光栅化的图元，GPU会采用**双重缓冲(Double Buffer)**。当场景已经渲染到了**后置缓冲区(Black Buffer)**中，就会和**前置缓冲区(Front Buffer)**进行内容交换，保证了画面的连续。






# 第三章：Unity Shader

## 3.1 shader lab

### 3.1.1 什么是shaderlab

Unity Shader是Unity提供的高层级渲染抽象层。在Unity中，所有的Unity Shader都通过ShaderLab来编写。ShaderLab是Unity提供编写Unity Shader的一种说明性语言。

![image-20220918012327737](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918012327737.png)



### 3.1.2 Unity Shader结构

```js
Shader "MyShader"
{
    Properties
    {
        //properties needs
	}
    
    SubShader
    {
        //true Shader code will be write here
        //Surface Shder				表面着色器
        //Vertex/Fragment Shader	顶点/片元着色器 
        //Fixed Function Shader		固定函数着色器
	}
    SubShader
    {
        
	}
}
```



#### Name and Proerties

文件的第一行需要指定名字。通过添加'/'可以控制在材质面板中出现的位置。

```js
Shader "Custom/NewSurfaceShader"
```

![image-20220918013715896](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918013715896.png)

`Proerties`中包含了一系列的属性。

属性的名字(Name)通常以下划线开始，如`_Color`。显示在材质面板上的名称(display name)，如`"Color"`。还要指定它的类型(PropertyType)如`Color`，以及默认值如`(1,1,1,1)`。

```js
Properties
{
    _Color ("Color", Color) = (1,1,1,1)
    _MainTex ("Albedo (RGB)", 2D) = "white" {}
    _Glossiness ("Smoothness", Range(0,1)) = 0.5
    _Metallic ("Metallic", Range(0,1)) = 0.0
}
```

![image-20220918014257699](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918014257699.png)





#### SubShader

一个Unity Shader文件至少包含一个SubShader语义块。加载时，unity会扫描所有的SubShader语义块，并选择第一个能在目标平台上运行的SubShader。如果都不支持，会使用Fallback指定的Unity Shader。

不同性能的显卡适用不同的SubShader数，性能更高的显卡能够使用更多的SubShader。

```js
SubShader
{
    //可选的标签
    [Tags]
    
    //可选的状态
    [RenderSetUp]
    
    //一个Pass定义了一个完整的渲染流程
    Pass
    {
        //true Shader code will be write here
        //Surface Shder				表面着色器
        //Vertex/Fragment Shader	顶点/片元着色器 
        //Fixed Function Shader		固定函数着色器
    }
}
```



##### 状态设置

ShaderLab常见的渲染状态设置选项：

![image-20220918182918087](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918182918087.png)

如果在SubShader块中设置了状态，就会应用到全部的Pass。可以在Pass中单独设置状态。



##### 标签(Tages)

标签的作用是告诉Unity渲染引擎要怎样以及何时渲染这个对象。

标签**仅可**在Subshader中声明，**不可**在Pass中声明。

结构如下:

```js
Tags {"TagName1" = "Value1" "TagName2" = "Value2"}
```



#### Pass

结构如下

```js
Pass
{
	[Name]
    [Tags]
    [RenderSetUp]
    //Other code
}
```

可以在定义该Pass的名称

```js
Name "MyPassName"
```

通过这个名称，可以使用ShaderLab的UsePass命令来使用其他Unity Shader中的Pass

```js
UsePass "MyShader/MYPASSNAME"
```

需要注意，Pass的名称会被Unity转换成大写的表示，因此在使用UsePass命令的时候需要将Pass名称全部大写。

Pass也可以使用标签，不过不同于SubShader。

![image-20220918184214898](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918184214898.png)

一些特殊Pass

- **UsePass**
- **GrabPass**



#### Fallback

在SubShader语义块后，可以添加Fallback指令。如果所有的SubShader在这块显卡上都无法运行，就会使用这个Fallback指定的最低级shader。

```js
Fallback "name"
//for example
Fallback "VertexLit"
//or
Fallback Off
//means do noting
```

在渲染阴影纹理时，Unity会寻找一个阴影投射Pass。通常Fallback使用的内置Shader包含了一个这样的通用Pass。



### 3.1.3 着色器

#### 表面着色器

**Surface Shader**是Unity创造的着色器代码类型，定义在**Pass**中。需要的代码量很少,效果比较差。

本质上是将开发者提供的表面着色器转换为顶点/片元着色器。是对顶点/片元着色器的高层抽象。可以交由Unity处理光照细节。



#### 顶点/片元着色器

顶点/片元着色器需要定义在CGPROGRAM和ENDCG之间，同时要写在Pass语义块，同样使用GL/HLSL编写。

我们需要自定义每个Pass的Shader代码，可以控制渲染实现的细节。

```js
Pass
{
    CGPROGRAM

    #include "UnityCG.cginc"

    #pragma vertex vert
    #pragma fragment frag

    uniform float4 _Color;

    struct a2v
    {
        float4 vertex : POSITION;
        float3 normal : NORMAL;
        float4 texcoord : TEXCOORD0;
    };

    struct v2f
    {
        float4 pos : SV_POSITION;
        fixed3 color : COLOR0;
    };

    v2f vert(a2v v)
    {
        v2f o;
        o.pos = UnityObjectToClipPos(v.vertex);
        o.color = v.normal * 0.5 + fixed3(.5, .5, .5);
        return o;
    }

    fixed4 frag(v2f i) : SV_TARGET
    {
        fixed3 c = i.color;
        // c *= _Color.rgb;
        return fixed4(c, 1.0);
    }

    ENDCG
}
```



#### 固定函数着色器

一些老旧设备不支持可编程管线着色器，需要使用固定函数着色器，这种方法已经逐渐被淘汰。



### 3.1.4 怎么选择Unity Shader

- 使用可编程管线着色器，即表面着色器或者顶点/片元着色器。
- 表面着色器与光源相关，但是影响性能。
- 如果光照数目少，使用顶点/片元着色器。
- 有很多自定义渲染效果，顶点/片元着色器。




# 第四章：数学知识

## 4.1笛卡尔坐标系

### 4.1.1 二维笛卡尔坐标系



### 4.1.2 三维笛卡尔坐标系



### 4.1.3 左右手系

- 左右手法则判断旋转正方向
- 左右手系可以互相转换，一般将z轴取反

- 当两个坐标系具有相同**旋向性(handedness)**就可以旋转重合。



### 4.1.4 Unity使用坐标系

- 除了观察空间以外皆为左手系。
- **right，up，forward**分别对应**x，y，z**。
- z值对应于深度值。深度值越大，z值越小（右手系）距离摄像机越远。





## 4.2点和矢量

### 4.2.1 点

- 是n维空间的一个位置，没有大小，宽度的概念。在笛卡尔坐标系，可以以 **P = (Px, Py)**表示二维的点， **P = (Px, Py, Pz)**表示三维空间的点。



### 4.2.2 矢量

矢量是值n维空间中一种包含了模（magnitude）和方向（direction）的有向线段。如速度（velocity）。标量（scalar）没有方向只有大小，如距离（distance）。

- 矢量的模就是适量的长度。
- 矢量的方向描述了矢量在空间中的指向。

矢量表示方法与点类似。如v=(x,y,z)来表示三维矢量。

- 标量一般使用小写字母表示
- 矢量一般用粗体小写表示，或者头顶箭头
- 矩阵使用大写粗体表示

通常，矢量被用于表示相对于某个点的**偏移（displacement）**，也就是说他是一个相对量。只要矢量的模和方向保持不变，无论放在哪里都是一个矢量。



### 4.2.3 点和矢量的区别

点是一个没有大小之分的空间中的位置，矢量是一个有模和方向但没有位置的量。但是两者在表示上非常相似。

矢量可以描述相对位置，此时矢量的尾是一个位置，头又是另一个位置。如果把矢量的为尾固定在坐标原点，矢量的表示就与点重合了。

![image-20220918224058876](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918224058876.png)

可以认为，任何一个点都可以表示成一个从原点出发的矢量。

将表示方向的矢量称作**方向矢量**（空间直线的方向用一个与该直线平行的非零向量来表示，该向量称为这条直线的一个方向向量）。



### 4.2.4 矢量运算

#### 矢量的乘法/除法

#### 矢量的加法

**a**+**b**就是将a的头指向b的尾，并画一条从**a**的尾指向**b**的头的矢量。如果进行了一个**a**的位移，又进行了一个**b**的位移，就等同于进行了一个**a**+**b**的位移。

减法由减向量指向被减向量。

在图形学中矢量通常用于描述位置偏移，可以利用矢量的加法和减法来计算一点相对于另一点的位移。



#### 矢量的模

矢量的模是一个标量，可以理解为矢量在空间中的长度。

![image-20220918230348010](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918230348010.png)



#### 单位矢量

很多情况下只关注矢量的方向而不是模，如计算顶点法线方向和光源方向。在这种情况下就需要计算单位矢量（unit vector），模长为1的矢量。转换过程就叫做归一化（normalization）。

![image-20220918231935045](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220918231935045.png)



#### 矢量的点积

- 点积就是将两个矢量对应分量相乘后取和，最终的结果是一个标量。


![image-20220920010249062](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920010249062.png)



- 矢量点积满足交换律。




- 点积的其中一个几何意义就是投影。

  **b**在平行**a**的线上的投影：

![image-20220920010623836](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920010623836.png)



- 投影的值可能是负数。正负与a和b的方向有关，当夹角大于90度的时候是负数。因此也可以用来判断物体的前后关系。


![image-20220920012215688](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920012215688.png)



在Shader的计算中，有一些性质可以用来帮助计算：

- 性质一：点积可以结合标量乘法

  ![image-20220920012415166](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920012415166.png)

- 性质二：点积可以解和矢量加法和减法

  ![image-20220920012453447](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920012453447.png)

- 性质三：一个矢量与本身进行点积的结果，是该矢量的模的平方

  因此可以直接使用点积来求模的平方，而不需要使用模的公式。当只需要比较向量大小的时候，就可以使用点积来求。

  ![image-20220920012526199](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920012526199.png)



点积的第二种求法：

![image-20220920012708593](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920012708593.png)

![image-20220920012722682](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920012722682.png)



通过点乘的结果反向求得夹角。

![image-20220920013136269](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920013136269.png)





#### 矢量叉积

叉积最常见的应用就是计算垂直一个平面/三角形的矢量，判断三角面片的朝向。

- 两个矢量的叉积可以如此计算：


![image-20220920013311373](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920013311373.png)



- 两个矢量的叉积会得到一个同时**垂直**于这两个矢量的新矢量。新矢量的模可以如此计算：


![image-20220920013658982](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220920013658982.png)

- 叉积不满足**交换律**，**结合律**，但是满足**反交换律**。



<img src="https://www.zhihu.com/equation?tex=\mid a\times b \mid=\mid a \mid  \mid b \mid\sin\theta = -\mid b \times a \mid
" alt="\mid a\times b \mid=\mid a \mid  \mid b \mid\sin\theta = -\mid b \times a \mid
" class="ee_img tr_noresize" eeimg="1">

- 可以使用**平行四边形**来解释：

  ![image-20220921102532427](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921102532427.png)

- 使用左右手来判断新向量的方向。将两向量首位相接，如果是左手系就使用左手，将四指与两向量旋转方向对齐，大拇指方向就是新产生的向量方向。

  ![image-20220514232355647](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220514232355647.png)



## 4.3 矩阵

在三维数学中，通常会使用矩阵来进行变换。一个矩阵可以将一个坐标空间转换到另一个坐标空间。



### 4.3.1 矩阵的定义

一个由 m X n 个标量组成的长方形数组。

![image-20220921115743034](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921115743034.png)





### 4.3.2 和矢量联系起来

矢量可以看做 n x 1 的列矩阵或者 1 x n 的行矩阵。

可以让矢量像一个矩阵一样参与矩阵运算。

![image-20220921115859597](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921115859597.png)

![image-20220921115959269](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921115959269.png)



### 4.3.3 矩阵运算

#### 矩阵标量乘法

![image-20220921120333700](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921120333700.png)



#### 矩阵和矩阵的乘法

现有矩阵`r x n`矩阵`A`和`n x c`矩阵`B`，其结果是一个 `r x c` 矩阵。

<img src="https://www.zhihu.com/equation?tex=新矩阵A\times B =C\\
C_{ij}=\sum_{k=1}^{n}A_{ik}\times B_{kj}
" alt="新矩阵A\times B =C\\
C_{ij}=\sum_{k=1}^{n}A_{ik}\times B_{kj}
" class="ee_img tr_noresize" eeimg="1">


矩阵乘法满足的运算规律：

- 矩阵乘法不满足交换律。左右矩阵交换位置后对应了不同的线性变换。

- 矩阵乘法满足结合律。

  ![image-20220921122647467](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921122647467.png)



### 4.3.4 特殊的矩阵

#### 方块矩阵

即方阵， n = m。

一些特殊运算只有方阵才满足。如对角矩阵（除对角元素外所有元素都为0）

![image-20220921122823718](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921122823718.png)



#### 单位矩阵

是一个特殊的对角矩阵，使用 ***I***（identity matrix）表示。任何矩阵和它相乘得到的还是原矩阵。

![image-20220921123115238](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921123115238.png)

![image-20220921123029444](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921123029444.png)



#### 转置矩阵

行变列，列变行就是转置。

将矩阵转置过后就是转置矩阵。可以从行矩阵转成列矩阵，反过来也一样。

![image-20220921123231717](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921123231717.png)

性质：

- 矩阵转置的转置等于原矩阵。

  ![image-20220921123325157](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921123325157.png)

- 矩阵相乘的转置，等于反向相乘矩阵的转置

![image-20220921123411480](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921123411480.png)



#### 逆矩阵

![image-20220921123535299](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921123535299.png)



如果一个矩阵有逆矩阵，那么称之为**可逆的（invertible）**或者**非奇异的（nonsingular）**。相反则称之为**noninverible** 或者 **singular**。



逆矩阵具有几何意义，矩阵**M**可以表示一个对向量**v**变换，逆矩阵可以还原这个变换。

如果一个矩阵不是一个降维矩阵（即线性无关），没有丢失信息，那就是一个可逆矩阵。

![image-20220921124718568](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124718568.png)



性质：

- 如果矩阵可逆

  ![image-20220921124012392](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124012392.png)

- 单位矩阵的逆是它本身。

- 转置矩阵的逆是逆矩阵的转置

  ![image-20220921124048610](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124048610.png)

- 矩阵相乘的转置是矩阵转置反向相乘

  ![image-20220921124122461](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124122461.png)

  ![image-20220921124137082](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124137082.png)



#### 正交矩阵

如果一个矩阵**M**与他的逆相乘是一个单位矩阵，那么称之为正交的（orthognal），是一个正交矩阵（orthognal matrix）。

![image-20220921124638361](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124638361.png)

可以得出结论。

![image-20220921124629193](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921124629193.png)



**如何判断正交矩阵**：

如果一个矩阵是正交矩阵，那么可以得到：

![image-20220921125337425](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921125337425.png)![image-20220921125347534](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921125347534.png)



性质：

- 矩阵的每一行，即c1,c2,c3都是单位矢量，因为这样他们与自己的点积才能是1。
- 矩阵的每一行，即c1,c2,c3都要相互垂直，因为这样他们之间的点积才能是0；
- 上述两条对于矩阵每一列都适用。

满足以上两个条件，就说明是一个正交矩阵。

**正交基**：

如果坐标空间的基矢量之间相互垂直，那么就称为正交基。

**标准正交基：**

如果正交基的矢量都为单位矢量，则称为标准正交基。

当一个矩阵是标准正交基的时候，就可以用矩阵的转置来求得逆变换。



#### 对称矩阵

该矩阵的转置就是自身。

<img src="https://www.zhihu.com/equation?tex=M^T=M
" alt="M^T=M
" class="ee_img tr_noresize" eeimg="1">
![image-20220921131757466](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921131757466.png)

#### 行矩阵还是列矩阵

行矩阵和列矩阵具有完全不同的意义，对于一个线性变换来说，列矩阵放在列矢量的左边，行矩阵放在行矢量的右边。

在Unity中，采用列矢量来计算。



## 4.4 矩阵变换

变换指的是把一些数据，点、方向矢量、颜色等，通过某种方式进行转换的过程。

### 4.4.1 线性变换（linear transform）

指的是能够保持矢量加和标量乘的变换。

![image-20220921220142842](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921220142842.png)

- **缩放（scale）**就是一种线性变换。如**f(x)= 2x**可以表示一个矢量的模被放大两倍。


- **旋转（rotation）**也是一种线性变换。

  

### 4.4.2 齐次坐标

#### 什么是齐次坐标

由于三阶方阵不能表示平移，我们就将其扩展到四阶矩阵。为此，需要将原来的三维矢量转换为四位矢量，也就是我们说的齐次坐标。

齐次坐标只是为了方便计算而使用的一种表示方式。



#### 分解齐次坐标

齐次坐标可以表现为以下形式：

![image-20220921220844070](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921220844070.png)

其中左上角的矩阵依然表示旋转和缩放，**0**是零矩阵，右下角的标量就是1。



#### 平移矩阵

（关于**w**看这个链接）

https://blog.51cto.com/u_15273495/3431380

可以使用矩阵乘法对一个点进行平移变换：

![image-20220921220955290](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921220955290.png)





对一个方向矢量进行变换，结果依然是它自身。很容易理解，矢量没有位置属性，可以位于任何位置。

![image-20220921223010229](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921223010229.png)



#### 缩放矩阵

可以对一个模型沿空间三轴进行缩放。

![image-20220921223052683](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921223052683.png)

- 当三个缩放系数相同，就称之为**统一缩放（uniform scale）**。统一缩放不会改变角度和比例信息，而非统一缩放会。




- 缩放矩阵的逆矩阵，即对角矩阵的逆：

  ![image-20220921223522148](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921223522148.png)

  缩放矩阵一般不是正交矩阵。

  



#### 旋转矩阵

绕x轴旋转：

![image-20220921224557528](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921224557528.png)

绕y轴旋转

![image-20220921224611147](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921224611147.png)

绕z轴旋转

![image-20220921224622975](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921224622975.png)



性质：

- 旋转矩阵是正交矩阵，且多个旋转矩阵相乘依然是旋转矩阵。

- 旋转矩阵的逆矩阵是一个旋转相反角度的矩阵。






#### 复合变换

一般来说，约定的变换顺序就是先缩放，再旋转，最后平移。

![image-20220921230102905](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220921230102905.png)

变换顺序至关重要，不同的顺序可能会导致完全不同的结果。





## 4.5坐标空间

![image-20220922173057221](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922173057221.png)

顶点着色器最基本的功能就是把模型的顶点坐标从模型空间转换到齐次裁剪坐标空间中。

渲染游戏的过程可以理解为将一个个顶点经过处理最终转化到屏幕上的过程。



### 4.5.1 坐标空间的变换

坐标空间会形成一个**层次结构**，每个坐标空间都是另一个坐标空间的**子空间**。

对坐标空间的变换实际上是在**父空间和子空间**之间对点和矢量进行变换。



一个子空间的坐标可以表示为(a,b,c)。可以将其理解为沿着各个轴分别移动a,b,c的距离。

则在父空间中，这个坐标表示为（O为原点坐标）：

![image-20220922000849358](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922000849358.png)

使用齐次坐标得到矩阵：

![image-20220922000937213](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922000937213.png)

- 当求出了一个子到父的坐标变换矩阵，就能通过该矩阵的逆矩阵反推子坐标空间的原点和坐标方向。


- 从坐标空间A到坐标空间B和坐标空间B到坐标空间A是两个互逆的变换。


- 一个从**模型空间**到**世界空间**变换矩阵就表示了**模型空间的基**在**世界空间的坐标**。对该矩阵的各列归一化后，就能得到模型空间的基在世界空间中的单位矢量表示。


- p空间的基坐标在c空间下表示，对应的就是这个变换矩阵的每一列。




对方向矢量的变换，可以忽略第四列和第四行，因为方向矢量是不受平移影响的。因此在计算的时候可以截取齐次矩阵中的三阶方阵。

在shader中，常常会看到截取变换矩阵的前三行前三列来对**法线方向、光照方向**来进行空间变换。

![image-20220922002336282](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922002336282.png)



当一个变换矩阵是正交矩阵时，矩阵的转置等于矩阵的逆。

![image-20220922002909965](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922002909965.png)



### 4.5.2 顶点的坐标空间变换过程

在渲染管线中，一个顶点最开始在模型空间中定义，然后在不同空间中计算颜色，最后将它变换到屏幕空间中，得到真正的屏幕像素坐标。



### 4.5.3 模型空间

也被称作**对象空间（object space）**或者***局部空间（local space）***。是每个模型的独立坐标空间。当移动时，模型空间也会一起移动，旋转时，模型空间也会跟着旋转。



将一个模型放到场景中，就会有一个模型空间时刻跟随它。模型的某一个部分可以通过顶点属性来得到。比如有一个点的位置是（0,2,4），由于顶点变换中往往包含平移变换，因此将其扩展到齐次坐标系下，得到顶点坐标为（0,2,4,1）。

![image-20220922010825844](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922010825844.png)



### 4.5.4 世界空间

**world space**是一个特殊坐标系，它是我们所关心的**最大空间**。**最大**指的是世界空间是相对于所有坐标的**最外层空间**。

世界空间用于描述**绝对位置**，即物体**在世界坐标系中的位置**。



### 4.5.5 观察空间

**view space**也被称为**摄像机空间（camera space）**。摄像机决定了渲染的视角。其坐标可以随意，但在Unity中，**相机空间不同于世界空间**，**它使用右手坐标系**，+x指向右方，+y指向上方，+z指向后方。

屏幕空间和观察空间不同，屏幕空间是三维的。由观察空间转换为屏幕空间，需要一个**投影（projection）**操作。

**view space**也被称为**摄像机空间（camera space）**。摄像机决定了渲染的视角。其坐标可以随意，但在Unity中，**相机空间不同于世界空间**，**它使用右手坐标系**，+x指向右方，+y指向上方，+z指向后方。

屏幕空间和观察空间不同，屏幕空间是三维的。由观察空间转换为屏幕空间，需要一个**投影（projection）**操作。

顶点变换的第二步就是将顶点从世界空间转到观察空间，即**观察变换（view transform）**。

得到顶点在观察空间中的位置，有两种方法。

- 计算观察空间坐标轴在世界空间下的表示，然后构建出到世界空间的变换矩阵（同上）。
- 将观察空间平移，使原点和世界空间对齐。然后进行图中数据的逆操作，即求得逆矩阵。



### 4.5.6 裁剪空间/齐次裁剪空间

**clip space**，也被称作齐次裁剪空间。用于变幻的矩阵叫做**裁剪矩阵（clip matrix），也被称作投影矩阵（projection matrix）**。

完全位于这块空间内部的图元将会被保留，位于这块外的图元将会被剔除，与边界相交的图元则会被裁剪。

#### 视椎体（view frustum）

视椎体是空间中的一块区域，决定了摄像机可以看到的空间，由六个平面包围而成，这些平面也被称为**裁剪平面（clip planes）**。

视椎体有两种类型：

- 正交投影（orthographic projection）

  在正交投影中，所有网格大小都一样，平行线会一直平行。

- 透视投影（near clip plane）

  透视投影模拟了人眼看世界的方式。

![image-20220922112512463](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922112512463.png)



**近裁剪平面（near clip plane）**，**远裁剪平面（far clip plane）**决定了摄像机可以看到的深度范围。

![image-20220922113132590](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922113132590.png)



**通过投影矩阵将顶点变换到裁剪空间中**。                  

- 投影矩阵/裁剪矩阵并不进行真正的投影，只是准备投影。
- 对x、y、z分量进行缩放。使用w分量作为范围，进行裁切。

#### 透视投影

![image-20220922114853046](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922114853046.png)

- 在Unity中，可以通过FOV属性来改变视椎体数值方向的张开角度，而near和far参数可以修改近裁切平面和远裁切平面里摄像机的远近。这样就可以求出视椎体近裁切平面和远裁切平面的高度。


![image-20220922115032170](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922115032170.png)

- Unity中，一个摄像机的横纵比由Game视图的横纵比和ViewPort Rect的W 和 H属性共同决定。假设相机横纵比为Aspect：


![image-20220922115337615](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922115337615.png)



- 这里的投影矩阵建立在unity对坐标系的假定之上，针对的是观察空间的右手坐标系。


- 将一个观察空间的顶点与之相乘，就可以将顶点变换到裁剪空间中。


![image-20220922135829926](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922135829926.png)



如果一个顶点在视椎体内，那么它变换后的坐标必须满足

![image-20220922141230016](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922141230016.png)



以下是经过投影矩阵后，视椎体的变化

![image-20220922144128263](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922144128263.png)

注意到经过裁剪矩阵变换后，空间的旋向性被改变：从右手坐标系变换到了左手坐标系。离相机越远，z值越大。



#### 正交投影

与透视投影一致，都是由相机和Game视图的横纵比共同决定的。

正交投影的视椎体是一个长方形。一样可以求得视椎体近裁剪平面和远裁剪平面的高度：

![image-20220922145200968](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922145200968.png)



现在还缺乏横向信息，一样可通过相机的横纵比得到:

![image-20220922145237806](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922145237806.png)![image-20220922145247998](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922145247998.png)



可以得出裁剪矩阵

![image-20220922145543766](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922145543766.png)

![image-20220922145709460](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922145709460.png)

和透视投影不同的是，最终结果w值依然等于1。

最后，坐标系的旋性也会被改变。

![image-20220922150410034](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922150410034.png)





### 4.5.7 屏幕空间

裁剪后，将视椎体投影到**屏幕空间（screen space）**。经过这一步变换，将会得到真正的像素位置。

- 首先进行标准**齐次除法（homogeneous divisoion）**也被称作**透视除法（perspective division）**。就是用w去除各个轴分量。在OpenGL中，这一步的得到的坐标叫做**归一化的设备坐标（Normalized Device Coordinates, NDC）**。


- 经过这一步，齐次裁剪坐标空间就被转换到NDC中，一个立方体。按照OpenGl的惯例，立方体的范围是[-1, 1]。


![image-20220922163903545](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922163903545.png)



在三维齐次坐标中，对于任何一个齐次坐标，都在它低维度中有一个确定的坐标

反过来说，`每一个低维度坐标`，都对应了`无穷个齐次坐标`，而这些`齐次坐标`组成了`一条过零点的直线`。

只需要除以第三维的分量，就能还原回原来的坐标。

<img src="https://www.zhihu.com/equation?tex=[ka,kb,k]^T对应的是欧式空间中的[a,b]^T
" alt="[ka,kb,k]^T对应的是欧式空间中的[a,b]^T
" class="ee_img tr_noresize" eeimg="1">
![image-20221001005210748](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20221001005210748.png)

因此，在四维齐次坐标的情况下，除以第四维的分量w，就能得到在三维空间中的实际坐标。最终就会形成上述的立方体。



对于正交投影来说，本身的裁剪空间就是一个立方体，且由于经过正交投影矩阵变换后的顶点w分量为1，不会产生任何影响。

![image-20220922164232956](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922164232956.png)



现在根据变换后的x，y坐标来映射输出窗口的像素坐标。在Unity中，屏幕空间左下角的像素坐标是（0,0），右上角像素坐标是（pixelWidth，pixelHeight）。由于当前x和y都已经是[1,1]，因此这个映射过程就是一个缩放的过程。

齐次除法和屏幕映射过程可以总结成下面的公式：

![image-20220922171153338](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922171153338.png)



通常z分量会被用于深度缓冲。



在Unity中，从裁剪空间到屏幕空间的转换是由Unity帮我们完成的。





## 4.9法线变换

**法线（normal）**也被称为法矢量（normal vector）。法线是需要特殊处理的一种方向矢量，变换顶点时，也要变换它的顶点法线，以便后续处理（如片元着色器）中的计算光照等。

- **切线（tangent）**，或**切矢量（tangent vector）**

  切线通过计算两个顶点之间的插值得到。可以使用三阶矩阵来变换（方向矢量不受平移的影响）。切矢量不会受到非统一缩放的影响。

![image-20220922174721997](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922174721997.png)

- **法线**

  法线会受到非统一缩放的影响，不能直接使用变换矩阵。需要使用原变换矩阵的逆转置矩阵：


<img src="https://www.zhihu.com/equation?tex=G =(M^{-1}_{A->B})^T
" alt="G =(M^{-1}_{A->B})^T
" class="ee_img tr_noresize" eeimg="1">

-  **特例**

  当变换矩阵为正交矩阵，法线变换矩阵即为原变换矩阵。


<img src="https://www.zhihu.com/equation?tex=(M^{-1}_{A->B})^{T}=M_{A->B}
" alt="(M^{-1}_{A->B})^{T}=M_{A->B}
" class="ee_img tr_noresize" eeimg="1">






# 第五章：Unity Shader的基本语法结构

## 5.1顶点片元着色器的基本结构：

```js
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Custom/Chapter5-SimpleShader"
{
    SubShader 
    {
        Pass
        {
            //start cg
            CGPROGRAM

            //函数声明，告诉Unity哪个函数包含顶点着色器代码，哪个包含片元着色器
            //#pragma vertex [name]
            #pragma vertex vert
            #pragma fragment frag

            //顶点着色器
            //POSITION 语义，不可省略。把模型顶点坐标填充到参数v中
            //SV_POSITION 告诉Unity顶点着色器的输出是裁剪空间中的顶点坐标
            float4 vert(float4 v : POSITION) : SV_POSITION
            {
                //更新后，unity自带MPV函数 模型 观察 投影矩阵
                return UnityObjectToClipPos (v);
            }

            //片元着色器
            //SV_TARGET 也是HLSL的系统语义，告诉渲染器把用户输出的颜色存储到一个渲染目标中
            //这里输出到默认的帧缓存中
            fixed4 frag() : SV_TARGET
            {
                //返回颜色
                //每个分量都是[0,1],(0,0,0,0)为黑色,(1,1,1,1)白色
                return fixed4(.5, .5, .5, .5);
            }

            ENDCG
            //end cg
        }
    }
}

```



### 5.1.1 语义

**语义（semantics）**就是一个赋给Shader输入输出的字符串，这个字符串表达了这个参数的含义。语义可以让Shader知道从哪里读取数据，并把数据输出到哪里。

- 语义分为有意义语义和无意义语义。
- 接受系统数据的语义有特殊含义，而用户输入无特殊含义。

**SV系统数值（ststem-value）语义**。

- SV_POSITION表示光栅化的变换后的顶点坐标（即齐次裁剪空间中的坐标）。
- SV_TARGET表示告诉渲染器把结果存储到渲染目标。



### 5.1.2 模型数据从哪里来

- 每帧调用Draw Call时，Mesh Render组件会把它负责渲染的模型数据发送给Unity Shader。


- 定义结构体逐顶点获取模型数据。

```js
//application to vertex shader
struct a2v
{
    //用模型空间的顶点坐标填充
    float4 vertex : POSITION;
    //用模型空间的法线方向填充
    float3 normal : NORMAL;
    //用模型的第一套纹理坐标填充
    float4 texcoord : TEXCOORD0;
}
```



### 5.1.3 顶点着色器和片元着色器之间如何通信

- 定义结构体接受返回信息

- 顶点着色器计算后返回结构体
- 片元着色器接收顶点着色器的输出做插值后得到的结果。

```js
//vertex shader to fragment shader
struct v2f
{
    float4 pos : SV_POSITION;
    fixed3 color : COLOR0;
}

v2f vert(v2f v) : SV_POSITION
{
    v2f o;
    o.pos = UnityObjectToClipPos(v.vertex);
    o.color = ...
    return o
}

fixed4 frag(v2f i) : SV_TARGET
{
    return fixed4(i.color, 1.0);
}
```



### 5.1.4 顶点着色器和片元着色器的差别

- 顶点着色器逐顶点，片元着色器逐片元
- 顶点着色器获取顶点信息
- 片元着色器获取顶点插值结果信息。



### 5.1.5 使用属性

通过参数，可以随时调整材质的效果。这些参数需要写在Properties中。

为Shader添加**Properties**：

```js
Properties
{
    _Color ("Color Tint", Color) = (1, 1, 1, 1)
}
```

会在材质面板显示一个颜色拾取器，从而直接控制模型在屏幕上显示的颜色。

![image-20220922234321282](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220922234321282.png)

为了在CG代码中可以访问它，还需要在代码片段中定义一个变量。这个变量的名称和类型必须与Properties语义块中的属性定义相匹配。

```js
CGPROGRAM

#pragma vertex vert
#pragma fragment frag
//
float4 _Color;
```



### 5.1.6 内置包含文件

**包含文件（include file）**，类似c++的头文件。在Unity中，它们的文件后缀是.cginc。

在编写的时候可以使用#include指令把这些文件包含进来。

```js
CGPROGRAM
//...
#include "UnityCG.cginc"
//...
ENDCG
```






# 第六章：标准光照模型

#### 大纲

- 光照

  - 环境光
  - 自发光
  - 漫反射

- 标准光照模型

  - 漫反射
    - 兰伯特
    - 半兰伯特

  - 高光反射
    - Phong
    - Blinn-Phong

- 渲染方式

  - 逐顶点
  - 逐像素

## 6.1模拟光照

模拟真实光照环境来生成一张图像，需要考虑3种物理现象

- 首先，光线从光源射出
- 然后，光线和场景中的物体相交：一些被吸收，一些被散射到其他地方。
- 最后，摄像机吸收了一些光，产生一张图像。

**在本章中，假设漫反射部分没有方向性，光线在所有方向上是平均分布的。同时，只考虑一个特定方向上的高光反射。**



#### 6.1.1 辐照度

[补充网站]: https://observablehq.com/@camargo/lamberts-cosine-law

**辐照度（irradiance）**：垂直于**I**的单位面积上单位时间穿过的能量。

<img src="https://www.zhihu.com/equation?tex=E_{e}=\frac{d\Phi_{e}}{dS} \\
E_{e}表示辐射照度，\Phi_{e}表示辐射通量，S表示面积
" alt="E_{e}=\frac{d\Phi_{e}}{dS} \\
E_{e}表示辐射照度，\Phi_{e}表示辐射通量，S表示面积
" class="ee_img tr_noresize" eeimg="1">
一般来说，光照都不会和平面垂直，因此需要使用光源方向和表面法线之间的夹角余弦值得到照射面积。

<img src="https://www.zhihu.com/equation?tex=cos\theta=\frac{d}{d_0}⇒E_{e}=\frac{d\Phi_{e}cos\theta}{dS} \\
d_0指板的长度
" alt="cos\theta=\frac{d}{d_0}⇒E_{e}=\frac{d\Phi_{e}cos\theta}{dS} \\
d_0指板的长度
" class="ee_img tr_noresize" eeimg="1">
![image-20220923114909761](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923114909761.png)





#### 6.1.2 吸收和散射

**散射（scattering）和吸收（absorption）**

- 散射只会改变光线的方向，而不改变光线的密度和颜色。
- 吸收只会改变光线的密度和颜色，而不会改变光线的方向。

**折射（refraction）或 透射（transmission）**

- 光线在物体表面经过散射后，会散射到内部，称之为折射或透射。

**反射（reflection）**

- 散射到外部，称之为反射。

在折射进入物体内部的光线还会与内部颗粒进行相交，一些光线最后会重新发射出物体表面，而一些则被吸收。

那些重新发射出去的光线将具有和入射光线不同的方向分布和颜色。

![image-20220923120033099](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923120033099.png)



为了区别这两种不同的散射方向，在光照模型中使用不同的部分计算它们：

- **高光反射（specular）**部分表示物体表面是如何反射光线的。
- **漫反射（diffuse）**部分表示有多少光线会被折射、吸收和散射出表面。

根据入射光线的数量和方向，可以计算出射光线的数量和方向，通常用**出射度（exitance）**描述。

辐照度和出射度满足线性关系，它们的比值就是材质的漫反射和高光反射属性。



#### 6.1.3 着色

**着色（shading）**。根据材质属性、光源信息，使用一个等式去计算沿某个观察方向的出射度的过程。这个等式也被称为**光照模型（Lighting Model）**。



#### 6.1.4 BRDF光照模型

 当已知光源位置和方向、视角方向时，我们就需要知道一个表面是如何和光照进行交互的。

**BRDF（Bidirectional reflectance distribution function）**包含了对给定模型表面上的一个点的外观的完整描述。



## 6.2 标准光照模型

#### 6.2.1 环境光

 在真实的光照情况下,一个物体也可能会被**间接光照**所照亮。

![image-20220923122842631](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923122842631.png)

模型做简化，场景中的所有物体都使用相同的环境光,公式如下:

![image-20220923233930336](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923233930336.png)

#### 6.2.2 自发光

光线也可以直接由光源进入摄像机。标准光照模型使用自发光来计算这个部分的贡献度。直接使用改材质的自发光颜色：

![image-20220923234042449](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923234042449.png)



#### 6.2.3 漫反射

漫反射光照用于对那些被物体表面随机散射到各个方向的辐射度进行建模。在漫反射中，视角的位置并不重要，可以认为在任何反射方向上的分布都是一样的。

![image-20220923133838665](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923133838665.png)



##### 1.兰伯特光照模型

漫反射光照符合**兰伯特定律（Lambert's law）**:

<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{diffuse})max(0,\hat{n}\cdot \hat{I})
" alt="c_{diffuse}=(c_{light}\cdot m_{diffuse})max(0,\hat{n}\cdot \hat{I})
" class="ee_img tr_noresize" eeimg="1">

- **n**是表面法线
- **I**是指向光源的单位矢量
- **m**diffuse是材质的漫反射颜色
- **c**light是光源颜色。

这个公式很好理解，就是混合了材质的反射颜色和光源颜色。再将其乘以法向量和入射方向的余弦值，就可以得到最终的漫反射强度。



随着n和L之间的夹角θ逐渐增大，区域dA受到的光线照射量会越来越少 （因为很多光线都无法照射到dA表面上了）。当角度超出[-90, 90]时，点积光源就到了物体的背面，因此需要将强度直接归0，防止正表面被来自后方的光照亮。

<img src="https://www.zhihu.com/equation?tex=f(θ) = max(cosθ,0) = max(\hat{L}\cdot \hat{n},0)
" alt="f(θ) = max(cosθ,0) = max(\hat{L}\cdot \hat{n},0)
" class="ee_img tr_noresize" eeimg="1">
![image-20220923235410730](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923235410730.png)

##### 2.半兰伯特光照模型

半兰伯特模型就是在原模型上进行了一个简单的修改（由Valve公司在开发《半条命》时提出）——不限制区间，而是对光照方向和法线进行缩放以及偏移：

<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{diffuse})(\alpha(\hat{n}\cdot \hat{I})+\beta)
" alt="c_{diffuse}=(c_{light}\cdot m_{diffuse})(\alpha(\hat{n}\cdot \hat{I})+\beta)
" class="ee_img tr_noresize" eeimg="1">
在通常情况下，缩放度和偏移量都为0.5。

<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{diffuse})(0.5(\hat{n}\cdot \hat{I})+0.5)
" alt="c_{diffuse}=(c_{light}\cdot m_{diffuse})(0.5(\hat{n}\cdot \hat{I})+0.5)
" class="ee_img tr_noresize" eeimg="1">
这样，当点积结果小于0时，就可以将点积的结果映射到[0,1]之间。



#### 6.2.4 高光反射

这里的高光反射是一种经验模型，它可以用于计算完全镜片反射方向被反射的光线，可以让物体看起来有光泽，如金属材质。

![image-20220923133838665](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220923133838665.png)

计算高光反射需要知道：

- 表面法线n
- 视角方向v
- 光源方向I
- 反射方向r

反射方向可以通过表面法线和光源方向求得。


<img src="https://www.zhihu.com/equation?tex=r=2(\hat{n} \cdot \hat{I})\hat{n}-\hat{I}
" alt="r=2(\hat{n} \cdot \hat{I})\hat{n}-\hat{I}
" class="ee_img tr_noresize" eeimg="1">



##### 1.Phong光照模型

- **m**gloss，是材质的**光泽度（gloss）**，也被称为**反光度(shininess)**。用于控制高光区域的“亮点”有多宽。它的值越大，亮点就越小。
- **m**specular 是材质的高光反射颜色，它用于控制该材质对于高光反射的强度和颜色。
-  **c**light 则是光源的颜色和强度。同样，这里也需要防止**(v·r)**的结果为负数。


<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{r})^{m_{gloss}}
" alt="c_{diffuse}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{r})^{m_{gloss}}
" class="ee_img tr_noresize" eeimg="1">


##### 2.Blinn-Phong模型

这种模型的基本思想就是避免计算反射方向**r**。为此，引入了一个新的单位矢量**h**，通过对**单位矢量v和I**取平均后再归一化得到，即：

<img src="https://www.zhihu.com/equation?tex=\hat{h}=\frac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}
" alt="\hat{h}=\frac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}
" class="ee_img tr_noresize" eeimg="1">
然后使用n与h之间的夹角进行计算：

![image-20220924002002120](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220924002002120.png)

Blinn模型公式如下：

![image-20220924002017790](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220924002017790.png)

##### 3.两种模型对比

- 一般Blinn更快。
- 两者都是经验模型。



## 6.3 逐像素还是逐顶点

- 在顶点着色器中计算，称为**逐顶点光照（per-vertex lighting）**。


- 在片元着色器中计算，称为**逐像素光照（per-pixel lighting）**。




##### 1.逐顶点光照

也被称为**高洛德着色（Gouraud shading）**。在逐顶点光照中，**对每个顶点计算光照**，然后在渲染图元内部进行线性插值，最后输出像素颜色。由于顶点数量小于像素，因此逐顶点光照的计算量往往小于逐像素光照。但是逐顶点光照依赖于线性插值，因此当光照模型中有非线性计算时，逐顶点光照就会出现问题。

而且由于逐顶点光照会在渲染图元内部对顶点颜色进行插值，这会导致渲染图元内部的颜色总是暗与顶点处的最高颜色值，某些情况下会产生明显的棱角现象。



##### 2.逐像素光照

在逐像素光照中，**以每个像素为基础**，得到他的法线（通过顶点法线插值或者法线纹理采样得到），然后计算光照模型。这种在面片之间对顶点法线进行插值的技术称为**Phong着色（Phong shading）**，也称为Phong插值或法线插值着色技术。



##### 3.总结

比如画一个三角形，顶点为A/B/C，由于光源的影响使得A亮BC暗。 在顶点着色器中计算光效再传给片元着色器，则顶点之间的片元的颜色是OpenGL插值的结果。 在片元着色器中计算光效，则能保证所有片元颜色均是由光源计算得到的。



## 6.2Unity中的环境光和自发光

在标准光照模型中，环境光和自发光的计算是最简单的。

Unity中，场景的环境光可以在Window->Lighting->Environment中调节

![image-20220924005638154](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220924005638154.png)

在Shader中，可以通过内置变量**UNITY_LIGHTMODEL_AMBIENT**得到环境光的颜色和强度信息。



计算自发光非常简单，只需要在片元着色器输出最后的颜色之前，把材质的自发光颜色添加到输出颜色上即可。



## 6.3实现漫反射光照模型

### 逐顶点光照漫反射模型

- 套用兰伯特公式


<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{diffuse})max(0,\hat{n}\cdot \hat{I})
  " alt="c_{diffuse}=(c_{light}\cdot m_{diffuse})max(0,\hat{n}\cdot \hat{I})
  " class="ee_img tr_noresize" eeimg="1">

  

  ```js
   fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));
  ```

  

- 在顶点着色器中计算

```js
v2f vert(a2v v)
{
    v2f o;
    //变换到裁剪空间
    o.pos = UnityObjectToClipPos(v.vertex);

    //获取环境光照
    fixed ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

    //变换法线到世界空间
    fixed3 worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));

    //获取光源方向
    fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);

    //计算反射光
    fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));

    //和环境光融合
    o.color = ambient + diffuse;

    return o;
} 
```



### 逐片元光照漫反射模型

- 在片元着色器中计算

  ```js
  fixed4 frag(v2f i) : SV_TARGET
  {
      //获取环境光
      fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
      //获取法线的世界坐标
      fixed3 worldNormal = normalize(i.worldNormal);
      //获取光源方向
      fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
      //计算模型
      fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb*saturate(dot(worldNormal, worldLightDir));
      fixed3 color = ambient + diffuse;
      return fixed4(color, 1.0);
  }
  ```



### 实现半兰伯特模型

- 在片元着色器中计算

  ```js
  fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb*(0.5*dot(worldNormal, worldLightDir) + 0.5);
  ```

  

### 对比

- 正面：

  逐顶点的Lambert模型,逐像素的Lambert模型以及Half-Lambert模型:

  ![image-20221001005556642](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20221001005556642.png) 

- 背面

  Half-Lambert模型，逐像素的Lambert模型以及逐顶点的Lambert模型：

​		![image-20220930210112585](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220930210112585.png)





## 6.4实现高光反射模型

### Phong光照模型

- 套用公式

<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{r})^{m_{gloss}}
  " alt="c_{diffuse}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{r})^{m_{gloss}}
  " class="ee_img tr_noresize" eeimg="1">

  ```js
  fixed3 specilar = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
  ```

- 逐顶点和逐片元操作与上面一致。



### Blinn模型

- 套用公式

  - 计算中值向量：


<img src="https://www.zhihu.com/equation?tex=\hat{h}=\frac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}
  " alt="\hat{h}=\frac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}
  " class="ee_img tr_noresize" eeimg="1">

  ```
   fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
   fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
   
   fixed3 halfDir = normalize(worldLightDir + viewDir);
  ```

  - 计算高光反射

<img src="https://www.zhihu.com/equation?tex=c_{specular}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{h})^{m_{gloss}}
    " alt="c_{specular}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{h})^{m_{gloss}}
    " class="ee_img tr_noresize" eeimg="1">

    ```js
    fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
    ```

    

- 计算

  ```js
  fixed4 frag(v2f i) : SV_TARGET
  {
      float3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
  
      fixed3 worldNormal = normalize(i.worldNormal);
      fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
  
      fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
  
      fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
  	
      //获取中值向量
      fixed3 halfDir = normalize(worldLightDir + viewDir);
  	
      //计算
      fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
  
      return fixed4(ambient + diffuse + specular, 1.0);
  }
  ```



### 对比

从左到右依次为vertex Phong，pixel Phong， Blinn-Phong。

![image-20220927215116100](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220927215116100.png)



## 6.5公式总结

### 漫反射

- 兰伯特模型：

<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{diffuse})max(0,\hat{n}\cdot \hat{I})
  " alt="c_{diffuse}=(c_{light}\cdot m_{diffuse})max(0,\hat{n}\cdot \hat{I})
  " class="ee_img tr_noresize" eeimg="1">

  ```js
   fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));
  ```

- 半兰伯特模型

  ```js
  fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb*(0.5*dot(worldNormal, worldLightDir) + 0.5);
  ```



### 高光反射

- Phong光照模型

<img src="https://www.zhihu.com/equation?tex=c_{diffuse}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{r})^{m_{gloss}}
  " alt="c_{diffuse}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{r})^{m_{gloss}}
  " class="ee_img tr_noresize" eeimg="1">

  ```js
  fixed3 specilar = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
  ```

- Blinn-phong模型

  - 计算中值向量：


<img src="https://www.zhihu.com/equation?tex=\hat{h}=\frac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}
  " alt="\hat{h}=\frac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}
  " class="ee_img tr_noresize" eeimg="1">

  ```
  fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
  fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
  
  fixed3 halfDir = normalize(worldLightDir + viewDir);
  ```

  - 计算高光反射

<img src="https://www.zhihu.com/equation?tex=c_{specular}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{h})^{m_{gloss}}
    " alt="c_{specular}=(c_{light}\cdot m_{specular})max(0,\hat{n}\cdot \hat{h})^{m_{gloss}}
    " class="ee_img tr_noresize" eeimg="1">

    ```js
    fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
    ```

    


# 第七章：纹理材质

#### 大纲

- 单张纹理
- 实现纹理凹凸（看起来有凹凸感）
- 实现纹理渐变（动漫渲染等）
- 实现纹理遮罩（美术人员高度可控物体表面质感）

## 7.1基础纹理要点

- **纹理映射(texture mapping)**：使用一张图片来控制模型的外观。将一张图“粘”在模型表面，**逐纹素（texel）**（纹理元素）地控制模型颜色。

- **纹理映射坐标（texture-mapping coordinates）**：又被称作 **UV**坐标，使用二维变量(u,v)来表示。u是横轴坐标，v是纵轴坐标。一般会将将纹理坐标归一化

![image-20220925005853199](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220925005853199.png)

- **纹理采样**：通过输入的顶点坐标，在纹理中采样得到结果。输入坐标可能不在[0, 1]范围内。

  - ```c++
    tex2D(_MainTex, i.uv);
    ```

- **平铺模式（Wrap Mode）**：纹理的平铺模式决定了引擎在遇到不在[0, 1]范围内的纹理坐标时如何进行纹理采样。

  - **Repeat**：当UV超过[0,1]范围后，整数部分就会被舍弃，而直接采用小数部分。这样的结果是纹理会不断重复。
  - **Clamp**：会将UV限制在[0, 1]之间，大于1取1，小于0取0。

- **过滤模式（Filter Mode）**

  - **Point**：最近邻（nearest neighbor）滤波，实现像素风
  - **Bilinear**：线性滤波。线性插值，看起来会模糊。
  - **Trilinear**：与Bilinear差唯一的区别是Trilinear会混合多级渐远纹理。

- **多级渐远纹理（mipmapping）**

  在Unity中，可以勾选如下选项开启**多级渐远纹理（mipmapping）**技术：

  ![image-20220925202455359](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220925202455359-1664549392023-3.png)



## 7.2单张纹理

#### 7.2.1 创建纹理

##### 1.在属性中声明材质

```
Properties
{
    _MainTex ("Main Tex", 2D) = "white" {}
}
```

##### 2.声明变量

- sampler2D为材质类型

- 需要多声明一个带有_ST后缀的float4变量。在Unity中，使用**纹理名ST**声明某个纹理的属性。其中**ST**是**scale**和**Transform**的缩写。可以得到纹理的缩放和偏移值。
  - **_MainTex_ST.xy**存储的是缩放值
  - **_MainTex_ST.zw**存储的是偏移值

```
sampler2D _MainTex;
float4 _MainTex_ST;
```

##### 3.使用TRANSFORM_TEX(tex, name)宏计算纹理缩放

```JS
//宏效果等同于
//o.uv = v.texcoord.xy * _MainTex_ST.xy  + _MainTex_ST.zw;
o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
```

##### 4.计算漫反射

- 进行纹理采样得到颜色,并和物体颜色混合，得到漫反射颜色。

  ```c++
  fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
  fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
  ```

![image-20220925015507044](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220925015507044.png)



## 7.3凹凸映射

### 7.3.1 原理

使用高度纹理来计算法线，计算后得到不同的颜色，使得纹理看起来有凹凸质感。

- **高度纹理（height map）**来进行**表面位移（displacement）**，然后得到一个修改后的法线值。也被称为**高度映射（height mapping）**。
- **法线纹理（normal map）**直接存储表面法线，又被称为**法线映射（normal mapping）**。

本章提到的多个纹理计算流程基本一致。



### 7.3.2 高度纹理

高度图中存储的是**强度值（idtensity）**，颜色越浅说明该位置表面越往外凸起。

- 直观
- 计算复杂，需要由像素的灰度值计算而得。

![image-20221001010133599](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20221001010133599.png)



### 7.3.3 法线纹理

法线纹理存储的是表面法线方向。

- 法线的分量范围在[-1, 1]，而像素分量范围[0, 1]，因此需要做一个映射：

<img src="https://www.zhihu.com/equation?tex=pixel=\frac{normal + 1}{2}
  " alt="pixel=\frac{normal + 1}{2}
  " class="ee_img tr_noresize" eeimg="1">

- 采样后需要对结果进行一次反映射，得到原先的法线方向。

<img src="https://www.zhihu.com/equation?tex=normal=pixel\times 2-1
  " alt="normal=pixel\times 2-1
  " class="ee_img tr_noresize" eeimg="1">




有两种存储法线信息的方法：

- 模型空间的法线纹理
- 切线空间的法线纹理



**切线空间储存方式**：

- 法线为z轴
- 切线为x轴
- **副切线（bitangent，b）**为y轴。

副切线：由法线和切线叉积得到。



**模型空间存储法线：**

- 直接使用模型空间法线信息。
- 统一坐标系，边缘插值结果平滑。
- 绝对法线信息，只能对应一个模型。

**切线空间存储法线：**

- 使用切线空间法线信息。
- 每个顶点都有独立坐标系。

- 切线空间法线，不受模型影响。



### 7.3.4 在切线空间下计算切线空间法线纹理

##### 1.纹理法线贴图_BumpMap

- 用**“bump”**作为它的默认值。“bump”是Unity内置法线纹理。
- **_BumpScale**用来控制凹凸程度，当它为0时，意味着法线纹理不会对光照产生任何影响。

```js
Properties
{
    _Color ("Color Tint", Color) = (1, 1, 1, 1)
    _MainTex ("Main Tex", 2D) = "white" {}
    _BumpMap ("Normal Map", 2D) = "bump" {}
    _BumpScale ("Bump Scale", Float) = 1.0
    _Specular ("Specular", Color) = (1, 1, 1, 1)
    _Gloss ("Gloss", Range(8.0, 256)) = 20
}
```

##### 2.声明变量

- 可以采用和纹理贴图一样的缩放，不需要单独设置。

```js
sampler2D _MainTex;
float4 _MainTex_ST;
sampler2D _BumpMap;
float _MainTexScale;
```

##### 3.结构体

- **tangent：**存储切线

```js
struct a2v
{
    float4 vertex : POSITION;
    float3 normal : NORMAL;
    float4 tangent : TANGENT;
    float4 texcoord : TEXCOORD0;
};
```

##### 4.计算副切线，获取变换矩阵

- 叉乘得到副切线

  ```js
  float3 binormal = cross(normalize(v.normal), normalize(v.tangent.xyz)) * v.tangent.w;
  ```

- 切线空间基向量构成正交矩阵，因此法线变换矩阵就是切线空间的变换矩阵

  ```
  float3x3 rotation = float3x3(v.tangent.xyz, binormal, v.normal);
  ```

- 也可以使用**TANGENT_SPACE_ROTATION**宏来获取到切线空间的旋转矩阵。

  ```js
  TANGENT_SPACE_ROTATION
  //float3 binormal = cross(normalize(v.normal), normalize(v.tangent.xyz)) * v.tangent.w;
  //float3x3 rotation = float3x3(v.tangent.xyz, binormal, v.normal);
  ```

  - 定义

    ```c++
    #define TANGENT_SPACE_ROTATION \
        float3 binormal = cross( normalize(v.normal), normalize(v.tangent.xyz) ) * v.tangent.w; \
        float3x3 rotation = float3x3( v.tangent.xyz, binormal, v.normal )
    ```

    

- 将需要计算的变量变换到切线空间

  ```js
  o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
  o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;
  ```

##### 5.解映射，计算光照模型

- 解映射。

  ```js
  // tangentNormal.xy = (packedNormal.xy * 2 - 1) * _BumpScale;
  // tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
  tangentNormal = UnpackNormal(packedNormal);
  ```

  需要将法线纹理的类型设置为**Normal map**才能使用函数。

  ![image-20220926021818683](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220926021818683.png)

- 计算切线空间的顶点法线

  ```js
  tangentNormal.xy *= _MainTexScale;
  tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
  ```

  

![image-20220926024524114](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220926024524114.png)



### 7.3.5 在世界空间下计算

##### 1.定义结构体

- 由于一个插值寄存器只能存储float4大小的变量，对于矩阵这样的变量可以进行拆分。
- 实际上法线变换矩阵只需要存三阶即可。但为了充分利用插值寄存器的空间，多出来的一维可以用来存储顶点的世界坐标。

```js
struct v2f
{
    float4 pos : SV_POSITION;
    float4 uv : TEXCOORD0;
    float4 TtoW0 : TEXCOORD1;
    float4 TtoW1 : TEXCOORD2;
    float4 TtoW2 : TEXCOORD3;
};
```

##### 2.修改顶点着色器，计算切线空间到世界空间的变换矩阵

首先转换各个向量的坐标空间，然后存储进寄存器中。

```js
float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w;

//存储切线空间到世界空间的变换矩阵，以及顶点在世界坐标的表示。
o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);
o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);
```

##### 3.修改片元着色器

和切线空间没有太大的区别，计算得到世界空间的纹理法向量后，计算光照即可。

```js
float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);

fixed3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos));
fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv.zw));
bump.xy *= _BumpScale;
bump.z = sqrt(1.0 - saturate(dot(bump.xy, bump.xy)));

bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));
```



## 7.4 渐变纹理

有时需要更加灵活地控制光照结果，因此可以使用**渐变纹理**来控制**漫反射光照**的结果。

**Valve**首先使用了一种基于**冷到暖色调（cool-to-warm tones）**的着色技术。

[(PDF) A Non-Photorealistic Lighting Model For Automatic Technical Illustration (researchgate.net)](https://www.researchgate.net/publication/2611462_A_Non-Photorealistic_Lighting_Model_For_Automatic_Technical_Illustration)

![image-20220926235702616](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220926235702616.png)



这种方式可以自由控制物体的漫反射关照。不同的渐变纹理有不同的特性。

中间这张图由黑色逐渐向浅灰色靠拢，且中间的分界线部分微微发红，类似《军团要塞2》中人物渲染使用的渐变纹理。

![image-20220927000312790](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220927000312790.png)

右边的渐变纹理通常用语卡通风格的渲染。这种渐变纹理中的色调通常是突变的，即没有平滑过渡，以此来模拟卡通中的阴影色块。



### 7.4.1 实现

##### 1.声明渐变纹理

- ```
  _RampTex ("Ramp Tex", 2D) = "white" {}
  ```

- ```
  sampler2D _RampTex;
  float4 _RampTex_ST;
  ```

##### 2.纹理采样

```js
fixed3 diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert)).rgb * _Color.rgb;
fixed3 diffuse = _LightColor0.rgb * diffuseColor;
```

不同的渐变纹理效果：

![image-20220927013116779](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220927013116779.png)

##### 3.注意点

- 渐变纹理的Wrap mode需要改为Clamp。不然可能会因为浮点数精度的原因导致没有被限制在区间内（比如1.000001），被截取整数部分，只剩下小数（前文提到的Default会截取超出范围的数的小数部分，并保留小数部分）。导致顶点颜色变成黑色。

  ![image-20221001010625031](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20221001010625031.png)





## 7.5 遮罩纹理

**遮罩纹理（mask texture）**能保护某些区域，使他们免于修改。

- 为了得到不同强度的反光，可以使用遮罩纹理来控制光照。
- 也可以通过遮罩纹理来表现需要混合多张纹理的材质，如草地，石子，裸露土地的纹理。

使用纹理的一般流程是：

- 通过采样得到遮罩纹理的纹素值。
- 使用其中某个或者几个通道的值（如texel.r）来与某种表面属性进行相乘。
- 当通道值为0的时候，混合后得到0，就能不进行计算，从而保护区域。

遮罩纹理可以让美术人员更加精准地控制模型表面的各种性质。



### 7.5.1 实践

##### 1.准备

- 添加遮罩纹理和遮罩系数。

```js
_SpecularMask ("SpecularMask", 2D) = "white" {}
_SpecularScale ("Specular Scale", Float) = 1.0
```

```js
sampler2D _SpecularMask;
float _SpecularScale;
```



##### 2.片元着色器

- 对遮罩纹理的**r通道**进行采样，并进行缩放。
- 将高光反射的结果乘以得到的遮罩纹理系数。如果该像素计算得到的系数为0，那么就不会反光。

```js
fixed specularMask = tex2D(_SpecularMask, i.uv).r * _SpecularScale;
//遮罩限制高光
fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(tangentNormal, halfDir)), _Gloss) * specularMask;

return fixed4(ambinet + diffuse + specular, 1.0);
```

**需要说明的是**，我们使用的这张遮罩纹理其实有很多空间被浪费了——它的rgb分量存储的都是同一个值。在实际的游戏制作中，我们往往会充分利用遮罩纹理的每一个颜色通道来存储不同的表面属性。



##### 3.对比

左为使用遮罩层后的结果

![image-20220928205325968](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220928205325968.png)



### 7.5.2 其他遮罩纹理

![image-20221001001657327](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20221001001657327.png)








# 第八章 透明效果

#### 大纲

- 透明度测试
- 透明度混合
- 混合等式和参数
- 不同参数效果



## 8.1 透明效果

透明是游戏中经常要使用的一种效果。在实时渲染中要实现透明效果，通常会在渲染模型时控制它的**透明通道（Alpha Channel）**。



在Unity中，通常使用两种方法实现透明效果：

- **透明度测试（Alpha Test）**：当片元的透明度不满足条件（一般是小于某个阈值），该片元就会被舍弃。被舍弃的片元不会再进行任何处理。这种方法无法得到真正的半透明效果。
- **透明度混合（Alpha Blending）**：需要关闭深度写入。会使用当前片元的透明度作为混合因子，与已经存储在颜色缓冲中的颜色值进行混合，得到新的颜色。这种方法可以得到真正的半透明效果。



名词：

- 深度缓冲（depth buffer，或者叫z-buffer）：根据深度缓存中的值来判断该片元距离摄像机的距离，如果深度值更小，且开启深度写则更新缓冲区。

- 深度写入（z-write）：使用当前深度值更新缓冲区。





### 8.2 渲染顺序

对于两个半透明物体：

半透明物体之间也是要符合一定渲染顺序的。

渲染引擎一般会先对物体排序，再渲染：

- 先渲染所有不透明物体，并开启它们的深度测试和深度写入。
- 按距离排序所有半透明物体，然后从后往前渲染半透明物体，开启深度测试，关闭深度写入。

当物体间的前后顺序混乱时，又会出现问题。

![image-20220927221858643](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220927221858643-1664603614322-6.png)

### Unity Shader的渲染顺序

使用渲染队列（render queue）。

可以使用SubShader的Queue来决定我们的模型归于哪个渲染队列。

![image-20220927222620823](https://raw.githubusercontent.com/LuHec/Markdown4Zhihu/master/Data/一篇搞定UnityShader入门精要/image-20220927222620823.jpg)









