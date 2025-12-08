---
title: BillBoard in UE
publish: true
---
局部坐标系在发生变化，将局部坐标系变为世界空间下的相机空间三个基向量所构成坐标系

* 参考：[https://zhuanlan.zhihu.com/p/427484373](https://zhuanlan.zhihu.com/p/427484373)

# Tranform3x3Matrix

![[Pasted image 20250808204309.png]]

# WPO
### 模型在LocalSpace为XY平面，法线朝上
![[企业微信截图_17392750045593.png]]
![[企业微信截图_17392755237612.png]]

### mesh为UE默认Plane，不使用UV
![[Pasted image 20250508170252.png]]
mesh为UE默认Plane，使用UV
# Normal
输入为LocalSpace法线，输出为TangentSpace法线
![[企业微信截图_17392741931423.png]]