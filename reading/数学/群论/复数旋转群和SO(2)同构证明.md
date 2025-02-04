---
title: 复数旋转群和SO(2)同构证明
description: 证明了复数旋转群 e^{iθ} 与 SO(2) 旋转群的同构关系。通过同构映射和运算保持性,验证了二者的等价性。
author:
  - SDGLBL
created: 2025-02-04
tags:
  - 群论
  - 李群
  - 同构
  - group-theory
  - Lie-group
  - isomorphism
  - note
  - 笔记
share: "true"
---


# $e^{i\theta}$ 群与 SO(2) 群的同构
## 定义

1. $e^{i\theta}$ 群：
   - 元素：$\{e^{i\theta} | \theta \in \mathbb{R}\}$
   - 运算：复数乘法

2. SO(2) 群：
   - 元素：$\left\{\begin{pmatrix}\cos\theta & -\sin\theta \\ \sin\theta & \cos\theta\end{pmatrix} | \theta \in \mathbb{R}\right\}$
   - 运算：矩阵乘法

## 同构映射

$\phi: e^{i\theta} \mapsto \begin{pmatrix}\cos\theta & -\sin\theta \\ \sin\theta & \cos\theta\end{pmatrix}$

## 同构证明

1. 双射性：显然，每个 $\theta$ 唯一确定一个 $e^{i\theta}$ 和一个 SO(2) 矩阵。

2. 保持运算：
   
   对于 $e^{i\theta_1}$ 和 $e^{i\theta_2}$，有：
   
   $\phi(e^{i\theta_1} \cdot e^{i\theta_2}) = \phi(e^{i(\theta_1+\theta_2)}) = \begin{pmatrix}\cos(\theta_1+\theta_2) & -\sin(\theta_1+\theta_2) \\ \sin(\theta_1+\theta_2) & \cos(\theta_1+\theta_2)\end{pmatrix}$
   
   $\phi(e^{i\theta_1}) \cdot \phi(e^{i\theta_2}) = \begin{pmatrix}\cos\theta_1 & -\sin\theta_1 \\ \sin\theta_1 & \cos\theta_1\end{pmatrix} \cdot \begin{pmatrix}\cos\theta_2 & -\sin\theta_2 \\ \sin\theta_2 & \cos\theta_2\end{pmatrix} = \begin{pmatrix}\cos(\theta_1+\theta_2) & -\sin(\theta_1+\theta_2) \\ \sin(\theta_1+\theta_2) & \cos(\theta_1+\theta_2)\end{pmatrix}$

   因此，$\phi(e^{i\theta_1} \cdot e^{i\theta_2}) = \phi(e^{i\theta_1}) \cdot \phi(e^{i\theta_2})$

这证明了 $e^{i\theta}$ 群在复数乘法下与 SO(2) 矩阵在矩阵乘法下是同构的。