---
title: BillBoard in UE
publish: true
---
丨 前言：本文记录的是笔者对于UE4中使用billboard的总结和归纳

大纲
1. Billboard是什么
2. billboard种类
	1. Free Rotation Method
	2. Z-Axis Constrained Method
3. 各种种类的实现方式
	1. 原教旨主义
	2. 运用uv
4. 特殊使用案例
	1. instance
	2. 骨骼网格
5. 相关参考资料



Billboard的目标是，想办法让一个面片始终朝向摄像机的方向。






局部坐标系在发生变化，将局部坐标系变为世界空间下的相机空间三个基向量所构成坐标系

* 参考：[https://zhuanlan.zhihu.com/p/427484373](https://zhuanlan.zhihu.com/p/427484373)

# Tranform3x3Matrix
首先复习一下transform的含义
![[Pasted image 20250808204309.png]]

# WPO
### 模型在LocalSpace为XY平面，法线朝上
![[企业微信截图_17392750045593.png]]
![[企业微信截图_17392755237612.png]]

### mesh为UE默认Plane，不使用UV
![[Pasted image 20250508170252.png]]
# Normal
输入为LocalSpace法线，输出为TangentSpace法线
![[企业微信截图_17392741931423.png]]