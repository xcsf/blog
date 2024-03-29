---
title: WebGL优化方法
date: 2020-06-12 18:35:58
tags: WebGL
categories:
 - WebGL
 - 笔记
---

> 1. 找到性能瓶颈，尝试降低CPU或者GPU的时钟频率去测试哪个效率低
>
> 2. 纹理受限，可以采取 减少canvas的长宽或者使用低分辨率的纹理测试；webgl 纹理绑定伸展和收缩效果时，gl.NEAREST 是最快的但会产生块状效果，gl.LINER因为是取平均值，会产生模糊
>
> 3. 将Mip映射应用于纹理贴图
>
> 4. 处理webgl丢失上下文的问题
>
> 5. 不要经常切换program，在切换program和在着色器中使用if else语句都需要进行考量
>
> 6. 避免在顶点数组数据中使用常量
>
> 7. 在webgl中，使用drawElements()的gl.TRIANGLE_STRIP 结合退化三角形 比使用drawArrays()的gl.TRIANGLE方式节省内存,并且减少使用drawArrays和drawElements的次数
>
> 8. 顶点组织顺序按照数组排序，不要使用乱序，因为难以命中缓存
>
> 9. 减少使用drawArrays和drawElements的次数
>
> 10. 避免绘制时从GPU读回数据或状态,例如，gl.getError() gl.readPixels(), 影响流水线的实现
>
> 11. 用webgl inspector找出冗余的调用，因为webgl是的状态是跨帧持续的，减少使用改变webgl状态的方法。比如gl.enable(XXX)，只执行一次就行了
>
> 12. 用细节层次简化模型（LOD技术）
>
> 13.  避免在shader中做逻辑判断，比如if else。
>
>     > 有的人可能会很疑惑，为何要这样？这和GPU的基本调度有关系，GPU的基本调度单位叫做wavefront, 就是指一组完全相同的计算指令，在GPU的几个计算单元中并行执行，每一个指令的输入数据不同而已。这样并行度很高，可以极大程度提高性能。但是一旦引入if else，就会把wavefront破坏掉，比如现在有10个计算单元在并发执行，但是碰到if，在5个计算单元中为true, 在5个计算单元中为false, 这样会造成新的计算指令，那么之前的并行运算将无法继续。新的计算指令需要排队等待执行，或者新的指令要转移到新的计算单元上，这个过程涉及到数据的复制转移，会比较耗时，会严重破坏并行度。
>     >
>     > 但是有些场景下，shader中完全不用逻辑判断又不行，那该如何呢？可以考虑使用shader的内置函数，比如step函数，案例如下：
>     >
>     > float a;
>     >
>     > if(b >1){
>     >
>     > a = 1;
>     >
>     > }else{
>     >
>     > a = 0.5;
>     >
>     > }
>     >
>     > 可以优化为：
>     >
>     > float a;
>     >
>     > float temp = step(b, 1);
>     >
>     > a = temp * 0.5 + (1 - temp);
>
> 14. 减少三角形数量
>
>     > 较少三角形的数量大体上可以从以下几个角度入手：
>     >
>     > (1). 空间分割技术：包括八叉树，四叉树做空间分割，将不在当前可视区域物体剔除掉
>     >
>     > (2). 遮挡检测技术：视锥体范围内，有些物体会被前面的物体遮挡，这些被遮挡的物体其实是不需要渲染的。遮挡查询有多种技术方案实现，比如通过扩扑性，硬件遮挡查询。扩扑性比较麻烦，我这里推荐硬件遮挡查询技术，实现起来相对比较容易。
>     >
>     > (3). LOD技术：根据物体距离摄像头的距离，动态调节物体三角形的数量。
>     >
>     > (4). 图元类型的优化：使用GL_TRIANGLE_FAN或者GL_TRIANGLE_STRIP替代GL_TRIANGLES，因为这样可以重用顶点，减少三角形的数量。
>     >
>     > (5).使用顶点索引的方法做渲染：使用glDrawElements替代glDrawArrays，因为前者通过索引的方式可以减少三角形的数量。
>
> 15. 纹理的优化
>
>     > (1). 纹理的长宽最好是2的幂。
>     >
>     > (2). 纹理压缩：纹理压缩在opengl es 3.0和webgl 2.0上有比较好的支持，经压缩后的纹理可以减少图形数据，节省宽带。常见的压缩格式为ETC，Khronos公司提供有ETC格式压缩的免费压缩包，在opengl/webgl程序中使用glCompressedTexImage2D函数加载被压缩的纹理。
>     >
>     > (3). 纹理的上传：传统的纹理上传比较耗时，可以考虑使用两个PBO上传纹理，性能会有较大的提升。
>     >
>     > (4). 纹理的合成：如果有很多个小的纹理，每一个纹理单独加载，效率比较低下。可以考虑将多个小纹理合成到一个纹理上，仅仅加载一次，然后在程序中使用的时候，使用不同的纹理坐标范围来加载不同的纹理。
>
> 16. 减少系统内存向GPU内存传送数据的次数
>
>     > (1). 尽可能使用VBO/VAO
>     >
>     > (2). 在opengl es 3.0/webgl 2.0上可以使用Transform Feedback, 该方案可以使用GPU做通用运算，把计算的结果存入VBO中，在后期的渲染流程中使用该VBO作为输入。
>     >
>     > (3). 批次合并：比如一个最小包围体内有多个物体，可以将这些物体的三角形合并在一起，一次性的发送到GPU。
>     >
>     > (4). 使用instance: 如果要渲染多个重复的物体，可以使用instance特性。

![WebGL优化点](https://raw.githubusercontent.com/xcsf/blog-figure-bed/master/WebGL优化点.jpg)





