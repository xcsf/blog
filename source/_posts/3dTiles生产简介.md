---
title: 3dTiles生产简介
date: 2020-06-13 17:42:30
tags: Cesium
categories:
  - GIS
  - Cesium
  - 3dTiles
---

### 3dTiles 格式简介

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍.png)

由上图可知:

3D Tiles 是由，瓦片集数据和瓦片数据组成。

**瓦片集数据：**一个 JSON 文件存储，通常为 tileset.json。

**瓦片数据：**最终在 cesium 中展示的数据都保存在瓦片数据中。

### 一、瓦片集数据

一个简单的例子：

通过这个例子可以直观的了解 3D Tiles 格式数据。

文件目录：

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍1.png)

tileset.json 文件：

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍2.png)

该切片集切片组织方式最为简单，其只有 3 个瓦片数据。根节点为最简化版本，其 content 指向精细程度最低的瓦片 dragon_low.b3dm。其 children 包含次精细程度瓦片，依次到最精细层。运行时根据 geometricError 参数进行瓦片的选择渲染。

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍3.png)

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍4.png)

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍5.png)

asset(必需):

瓦片集的元数据对象。其中 asset.version(必需)属性定义了 3D Tiles 版本。

geometricError(必需):

根据运行时引入的最大屏幕空间误差(SSE)，决定哪个层次的细节应该呈现。。每个 tileset 和每个 tile 都有一个几何误差属性。为非负数，以米为单位，为切片的原始几何的简化来表示的误差值。如：根瓦片是最简化的版本，有最大的几何误差，而源几何数据(叶子瓦片)则具有接近 0 的几何误差。**一般可以直接用瓦片集外包区域体对角线表示。**

root(必需):

根瓦片。每个 Tile 中的属性如下：

boundingVolume(必需):

切片内容包围框。只有 box,sphere,region 属性。

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍6.png)

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍7.png)![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍8.png)![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍9.png)

box:定义右手笛卡尔坐标系(x,y,z),z 向上，第一行(前三个)元素为边界框中心坐标。第二行为 x 轴方向的半长。第三行为 y 轴方向半长。第四行为 z 轴方向半长；

sphere:前三个值为球体中心坐标值，最后一个为球体半径(米)；

region:定义了在 84 坐标系下具有纬度、经度、和高度坐标的边界区域，[西、南、东、北、最小高度、最大高度。

refine:

根瓦片必需，子瓦片默认继承。可以选择子切片是以”ADD”的方式还是”REPLACE”的方式进行渲染。

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍10.png)

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍11.png)

content:

通过 content 属性中的 URI 引用与 tile 关联的实际可呈现的内容。如果是相对路径则是相对 tilesetJSON 文件。除此之外 content 也包含一个 boundingVolume(非必须)属性，其与外层 tile. boundingVolume 不同处在于 content. boundingVolume 是一个紧密贴合的边界框。

children:

子瓦片数组，对于叶子瓦片，此属性可能未定义或者数组长度为 0；子切片内容由其父切片边界范围完全包围。并有更小的 geometricError。

一个复杂例子:

文件目录：

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍12.png)

tileset.json 文件

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍13.png)

这里引用外部瓦片集”lab_b_0.json”，**当指向外部切片集时 tile.children 必须为空或者未定义。**

引用不能形成循环。

transform：支持局部坐标系统时可定义。默认为单位矩阵。( 前端操作瓦片可利用该属性，如炸开效果。)我们展示需求不需要支持地理坐标系，应该不太需要这个参数。

lab_b_0.json 文件:

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍14.png)

这是一个由多个瓦片构成的瓦片集，可以看到该瓦片集根节点**无 content**，包含了 11 个 children。这与前面一个简单的例子不同。这是因为其生成瓦片方式的不同，这里是通过无层次结构的树状结构来组织。由此可知 3dtile 可以使用多种不同类型的空间数据结构组织瓦片。而其运行时引擎是通用的，可以呈现 tileset 定义的任何树。下图是常用树状结构：

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍15.png)

对比超图生成切片所支持的有，四叉树和八叉树结构。我们先尝试简单的四叉树结构来组织切片。

### 简单的瓦片生成方法

想象我们拿到了一个模型，里面有若干个图元(一个顶点集组成)构成了多个对象。我们要做的是使用四叉树的结构将空间划分下去。起始空为整个模型的外包围框，一生四，四生八…… 这样空间被我们分成了一个个小的瓦片，这里我们要定义**瓦片边长**来控制每个瓦片的大小，根据瓦片的范围去选择模型中得到图元，最后瓦片空间里存在的若干个模型图元即是瓦片的内容。如果每个瓦片过大，里面包含的对象就过多，在加载这一个瓦片时则会发生卡顿。如果瓦片过小，则瓦片文件则会过于碎片，会增加前端请求量和计算量。通过这个步骤，我们可以得到一个 tile 的**索引树**。同时也将模型按照空间分成了**多个部分**(瓦片内容)，同时经过计算得出每个**瓦片的包围盒**，**其父瓦片包围盒必须包围子瓦片的包围盒。**

通过上面的处理，我们已经把 tile 组织了起来，必需的参数还差一个 geometricError，上文写到，使用对角线长作为参数，计算简单。如果显示效果有问题，这个参数有待调整。

这样对于切片集，所必需的的 asset、geometricError、refine、root.boundingVolume、root. geometricError 属性都可以计算出来，生成对应的 json 文件。

### LOD 数据生成

到此，还有一个问题，就是我们常说的 LOD，经过上面处理我们仅仅是将一个模型分块(瓦片)，如果不做 LOD 即代表原始模型被分成小块传输并加载显示。所以在瓦片化后还需要在叶子瓦片下挂接多级精细程度的模型，就像第一个简单的例子那样，同一个模型分为三个精细层，通过 geometricError 去控制选择。所以，geometricError 如果选择对角线长作为参数，会出现一个问题，比如，简化 10%的对象和原始对象对角线长几乎一样，这样参数很可能会失效，初步考虑需要在简化面的过程中去提取出这个参数。我们可以先搁置这个问题，先采用简单的对角线长来计算，最后根据展示效果再来调整。

对于瓦片 LOD 的处理，这里相对复杂，寻找相关图形算法库自带实现对比效果应该是快速的方法。如果要自己实现减面算法，单纯的实现并不难，但是如果考虑到效率以及简化限度的控制比较麻烦比如，一个正方体他的 lod 不应该变形成其他的形状，对于 BIM 数据而言许多对象应该是存在类似问题，如果远看一个立方体变成一个四面体则很影响效果。

CGAL 中应该是存在解决相关问题的程序包，不过还需要尝试。

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍16.png)

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍17.png)

### 瓦片格式转换

最后需要将每个瓦片内容生成 3dtile 的瓦片的格式，即 b3dm、i3dm 等等。我们应该是使用常用的 b3dm 格式。对于该格式生成开源已经有少量工具(官方 obj2gltf 工具)可以借助，这一步涉及到属性数据及对象区分(比如，后续点击展示信息需要用到)。总体来说将模型在瓦片化之后，将每个瓦片内容转换成 b3dm 导出，存储路径即为 tile.content.uri 的值。具体格式内容这里不多描述，后续可以参考标准做转换。

### 简单生产流程实现

1. 完成 obj->gltf->glb->b3dm 格式转换。

2. 拆分 obj 中的对象并将每个对象生成对应的 b3dm 瓦片文件。同时需要按照所期望的加载顺序对瓦片进行排序。如：先加载外壳瓦片展示模型整体情况，再加载内部瓦片。

3. 根据排序，组织 b3dm 瓦片计算生成对应瓦片集 tileset.json 文件。

4. 将不同 LOD 的 OBJ 合成至同一瓦片集生成 tileset.json 文件

5. 渲染加载效果：可以看到每个瓦片对象，根据第 3 步，定义的tileset.json，按顺序渲染出来。

![3dTiles生产介绍](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/3dTiles生产介绍18.gif)
